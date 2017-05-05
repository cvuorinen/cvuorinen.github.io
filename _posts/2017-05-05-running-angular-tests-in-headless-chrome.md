---
title: Running Angular tests in headless Chrome
description: Executing Angular tests with Karma and Protractor from command-line without Chrome browser window.
categories:
  - Programming
tags:
  - Angular
  - Testing
  - JavaScript
image:
    src: 2017/running-angular-tests-in-headless-chrome.jpg
    alt: Running Angular tests in headless Chrome
    align: none
---

Angular has some great tooling for running tests, namely Karma and Protractor. By default (at least when using Angular CLI) they run using Chrome. So when you execute the tests from command-line, it will pop open a browser window where the tests execute. This works well enough, but sometimes you either don't want to see that browser window pop open or you are running the tests in an environment where there is no graphical environment (on a CI server or a Docker container for example).

There is nothing new in running Karma tests without a browser window, you have been able to do it with PhantomJS by installing the `karma-phantomjs-launcher`. PhantomJS has been good enough solution for this, but you might encounter some issues every now and then and need to add some additional polyfills etc. But Chrome now has the ability to run in [headless mode since version 59](https://www.chromestatus.com/feature/5678767817097216), so you can use it to run tests without needing to install any additional packages and with a more standard environment.

<!--more-->

## Conf your Karma

Whether you have a Karma config generated with Angular CLI or one that you have created manually, you can use a config option called **customLaunchers** to create a new launcher based on an existing one by defining additional flags for it. To use Chrome in headless mode, you need to add the following to your `karma.conf.js`

```
customLaunchers: {
  ChromeHeadless: {
    base: 'Chrome',
    flags: [
      '--headless',
      '--disable-gpu',
      // Without a remote debugging port, Google Chrome exits immediately.
      '--remote-debugging-port=9222',
    ],
  }
}
```

Note, depending on your Chrome version, the `--disable-gpu` flag might not be needed.

Then you can replace whatever you had in the `browsers` section (either 'Chrome' or 'PhantomJS' etc.) with `ChromeHeadless`. That's it, after that you can enjoy running your tests without any browser window popping up.

## E2E tests with Protractor

Running E2E tests in headless mode has been a bit more difficult, since it has not worked very nicely with PhantomJS. Basically your only option has been to run [Chrome in Xvfb]() (that's X virtual framebuffer in case you were wondering). But now it's as simple as [adding a few lines to your `protractor.conf.js`](https://github.com/angular/protractor/blob/master/docs/browser-setup.md#using-headless-chrome) to also run your E2E tests in headless mode. You need to add the following options under the `capabilities` key (where you should already have `browserName: 'chrome'`):

```
chromeOptions: {
  args: [ "--headless", "--disable-gpu", "--window-size=800x600" ]
}
```

## Go forth and test all the things

Now you can just sit back and enjoy running all your tests without any distracting browser windows, or more importantly you can run them in your Continuous Integration server like Travis CI or Jenkins etc.
