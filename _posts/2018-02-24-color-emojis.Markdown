---
layout: post
title: 图片渲染彩色 emojis 总结
date: 2018-02-20T22:50:00+08:00
categories: 总结
published: true
---

之前使用 splash 但是彩色的emoji怎么都渲染不出来，怀疑是底层的QtWebView不支持emoji。所以后来使用了puppeteer做了一份图片渲染的代码：

```
'use strict';

// Setup Koa
const Koa = require('koa');
const Router = require('koa-router');
const logger = require('koa-logger')
const bodyParser = require('koa-bodyparser');
const json = require('koa-json')

// Chrome Headless
const puppeteer = require('puppeteer');

const app = module.exports = new Koa();
const router = new Router();

app.use(bodyParser());

let browserWSEndpoint;

// Async route handlers are wrapped with this to catch rejected promise errors.
const catchAsyncErrors = fn => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

const timeout = ms => new Promise(res => setTimeout(res, ms))

async function getChrome() {
  let browser;

  if (browserWSEndpoint) {
    console.info('Reusing Chrome instance...');
    browser = await puppeteer.connect({browserWSEndpoint});
  } else {
    console.info('Starting new Chrome instance...');
    browser = await puppeteer.launch({
      args: [
        '--no-sandbox',
        '--disable-setuid-sandbox',
        '--disable-dev-shm-usage'
      ],
      executablePath: 'google-chrome-unstable'
    });
    const pages = await browser.pages();
    if (pages.length) {
      await Promise.all(pages.map(page => page.close()));
    }
    browserWSEndpoint = await browser.wsEndpoint();
    console.log(browserWSEndpoint);
  }

  return browser;
}

async function renderImage(url, options) {
  const browser = await getChrome();
  const page = await browser.newPage();
  page.setViewport({width: 2134, height: 3000});
  await page.goto(url, {timeout: 300000, waitUntil: 'networkidle0'}); // timeout 300s
  await page.waitFor(3000);
  const buf = await page.screenshot(options);
  await page.close();
  await browser.disconnect()
  return buf;
}

async function renderJpeg(ctx, next) {
  const {url, quality = 75} = ctx.request.body;
  if (url == null) {
    throw new Error('url parameter is empty');
  }
  console.log("url: ", url);
  console.log("quality: ", quality);

  ctx.body = await renderImage(url, {quality: parseInt(quality), type: 'jpeg'})
}

async function renderPng(ctx, next) {
  const {url} = ctx.request.body;
  if (url == null) {
    throw new Error('url parameter is empty');
  }
  console.log("url: ", url)

  ctx.body = await renderImage(url, {type: 'png', omitBackground: true})
}

// routes
router.get('/ping', (ctx, next) => {
  ctx.body = {name: 'puppeteer', version: '1.0.0'};
})

router.post('/render.jpeg', renderJpeg);
router.post('/render.png', renderPng);

app
  .use(logger())
  .use(router.routes())
  .use(router.allowedMethods());


(async () => {
  await getChrome();
  if (!module.parent) {
    console.log("start server and listen 3000")
    app.listen(3000);
  }
})();
```
