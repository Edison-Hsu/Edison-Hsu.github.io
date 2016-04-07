---
layout: post
title:  "Effective Ruby 21-25条 小结"
date:   2016-04-07
categories: 读书 总结 笔记
---

{% include image.html
            img="images/Effective Ruby 21-25条 小结.png"
            title="mind map"
            caption=""
            url="/images/Effective Ruby 21-25条 小结.png" %}
            
## 组合代替继承
当代软件发展到这种程度，很多时候我们研究Python和Ruby或其他近期发展的语言，都会发现一个特性，那就是不再支持多继承。取而代之的是使用Mixin的方式去实现多继承。也就是组合代替继承的思想。

## 异常
这几条开始写异常了，个人感觉异常开发中是一块非常容易被忽视的内容。通过这些内容的阅读。我自己的api-service项目，其中一些包括401参数和402认证错误，这些都应该用异常来完成，这样会带来更好的维护性和可读性

