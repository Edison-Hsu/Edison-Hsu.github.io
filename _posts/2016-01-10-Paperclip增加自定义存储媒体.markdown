---
layout: post
title:  "Paperclip增加自定义存储媒体"
date:   2016-01-10 18:41:52
categories: ruby paperclip
---

# Paperclip增加自定义存储媒体
现在国内外的存储云越来越多。基于这些云提供商的服务也越来越多。所以如果你在使用[Paperclip](https://github.com/thoughtbot/paperclip)，并且需要将自己的服务与服务商对接上，那么可以参考本文完成自定义Paperclip的Storage功能

## 第一步，利用Monkey Patch增加自己的Storage模块


    module Paperclip
      module Storage
        module MyStorage
        #...具体实现
        end
      end
    end


这里是简单的利用猴子补丁，将具体的实现卸载对应的module下，这样paperclip也可以利用到我们写的模块。具体paperclip是怎么利用的，请看下面的内容。

## 第二步，实现extended钩子函数


  def self.extended base
    begin
      require 'upyun'
    rescue LoadError => e
      e.message << " (You may need to install the upyun gem)"
      raise e
    end unless defined?(::Upyun)

      base.instance_eval do
        @bucket = @options[:bucket]
        @operator = @options[:operator]
        @password = @options[:password]

        raise Paperclip::Upyun::Exceptions::OptionsError '(You should set upyun_host)' unless @host = @options[:upyun_host]

        @options[:interval] = "!"

        @upyun = ::Upyun::Rest.new(@bucket, @operator, @password)
        log("@options[:url]: #{@options[:url]}")
        @options[:path]  = @options[:path].gsub(/:url/, @options[:url])
        log("@options[:path]: #{@options[:path]}")
        @options[:url]   = ':upyun_public_url'

        Paperclip.interpolates(:version) do |attachment, style|
          attachment.version(style)
        end unless Paperclip::Interpolations.respond_to? :version

        Paperclip.interpolates(:upyun_public_url) do |attachment, style|
          attachment.public_url(style)
        end unless Paperclip::Interpolations.respond_to? :upyun_public_url

        Paperclip.interpolates :upyun_host  do |attachment, style|
          attachment.upyun_host(style)
        end
      end
  end


我拿了我实现paperclip-upyun的例子说一下这个函数中需要走得流程，可以看得出来这个函数主要的任务就是初始化。通常云存储服务都会提供他自己的SDK，那么我们的任务其实就是将SDK的功能和paperclip结合起来，所以可以看到，一开始我就在`require 'upyun‘`了。紧接着后面的`@options`成员变量就是我们在paperclip配置表中设置的选项。初始完SDK后。主要的工作就是利用`Paperclip.interpolates`注册接口。具体`interpolates`实现的机制，可以看他的[源码](https://github.com/thoughtbot/paperclip/blob/master/lib/paperclip/interpolations.rb)，我这里不在赘述。

## 第三步，完成自己注册的接口


    def public_url(style = default_style)
      log("url:path=>#{path(style)}")
      url = "#{@options[:upyun_host]}/#{path(style)}"
    end

    def version(style = default_style)
      url = ''
      url += @options[:interval] unless style == default_style
      url += style.to_s unless style == default_style
      url
    end

    def upyun_host(style = default_style)
      "#{@options[:upyun_host]}" || raise('upyun_host is nil')
    end

我同样用upyun的例子，这些是我实现的接口，主要的作用是在注册Model时，实现对应的路径或者参数，例如

    class User < ActiveRecord::Base
        has_attached_file :avatar,
                          :path => ":class/:attachment/:id/:fingerprint/:filename:version",
                          :default_url => ":upyun_host/:class/:attachment/default_avatar.png:version"
    end

## 第四步，完成paperclip的回调函数


    def exists?(style = default_style)
      puts "enter exists? #{style}"
      resp = @upyun.getinfo(path(style))
      begin
        Paperclip::Upyun::Response.parse(resp)
      rescue => err
        log("UPYUN<ERROR>: #{err}")
      end
    end

    def flush_writes
      for style, file in @queued_for_write do
        log("saving #{path(style)}")
        retried = false
        begin
          upload(file, path(style))
        ensure
          file.rewind
        end
      end
      after_flush_writes # allows attachment to clean up temp files
      @queued_for_write = {}
    end

    def flush_deletes
      for path in @queued_for_delete do
        log("deleting: #{path}")
        delete(path)
      end
      @queued_for_delete = []
    end

    
这个一目了然，一共需要实现三个方法`exists?`, `flush_writes`, `flush_deletes`，对应的，当Model调用save或者其他需要写入数据库的方法时，paperclip就会调用自动调用者几个方法来实现保存和删除，那么我们要做的就是在`flush_writes`的方法中实现上传，在`flush_deletes`中实现删除的功能，在`exists?`中实现查询的功能。

## 总结
基本上就只需要上面4个步骤，就能实现自己的paperclip存储插件。我上面的所有例子也是根据我自己写的paperclip-upyun的gem来解释说明的。这个gem的github地址在[https://github.com/jhw-dev/paperclip-upyun](https://github.com/jhw-dev/paperclip-upyun)，有需要的童鞋也可以拿来参考。

如果喜欢的话也不妨点个star吧:)
