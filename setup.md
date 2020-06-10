---
title: Setup Guide
layout: sidebar
category: intro
---

## Setup Guide

**Note:** This guide assumes that your app can be started using `npm start`
and accepts a port passed in as the `PORT` environment variable.
Have a look at the [configuration guide](config.html#app) if that's not the case.

### Step 1: Install

Install Testium related packages & `Mocha`
```console
npm install --save-dev testium-mocha testium-driver-wd mocha
```

If your CI provides headless chrome and `chromedriver`, you should install the `chromedriver` package globally.
```console
npm install -g chromedriver
```

otherwise locally

```console
npm install --save-dev chromedriver
```

**NOTE:** 

All `chromedriver` versions have a specific min required Chrome version. 
If your local Chrome browser updates the version, you likely have to update 
your `chromedriver` install as well.

### Step 2: Configure

Create a `.testiumrc` file with your configuration.
This example uses a `JSON` config file,
but [INI is supported](https://en.wikipedia.org/wiki/INI_file) as well.

```json5
{
  "launch": true,
  "driver": "wd", // this is not necessary with testium-mocha 5.x+
  "browser": "chrome",
  "app": {
    "command": "npm start"
  },
  "chrome": {
    "headless": true
  },
}

```

`launch` tells Testium to start your application before running the test. Testium will automatically 
stop your application once all tests finished.

### Step 3: Write Tests

```js
const assert = require('assert'); // node assertion 

const { browser } = require('testium-mocha');

describe('The homepage', () => {
  // by default testium will use the `wd` driver
  before(browser.beforeHook());

  before('Load homepage', () =>
    browser.loadPage('/')
  );

  it('shows the right title', async () => {
    const title = await browser.getPageTitle()
    
    assert.strictEqual(title, 'Hello');
  });
});
```

### Step 4: Run Tests

You can just run the file above using `mocha`
and Testium will handle everything for you:

* Launch the app and headless chrome.
* Wait for the app to listen.
* Clear cookies & reset the window to a well-known size.
* Close the webdriver session and tear down your app & headless chrome.

```console
npx mocha my-test.js
# Or, after adding 'node_modules/.bin' to your $PATH:
mocha my-test.js
```

Thanks to [`rc`](https://www.npmjs.com/package/rc) you can use environment variables to override options.
This makes it super easy to run the same test in regular Chrome:

```console
testium_chrome__headless=false mocha my-test.js
```

**NOTE**: Testium will automatically download selenium the first time it is needed.
