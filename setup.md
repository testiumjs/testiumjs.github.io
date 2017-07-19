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

```sh
npm install --save-dev testium-mocha testium-driver-sync mocha
```

### Step 2: Configure

Create a `.testiumrc` file with your configuration.
This example uses [`ini` file syntax](https://en.wikipedia.org/wiki/INI_file)
but JSON is supported as well.

```ini
launch = true
driver = wd
```

`launch` tells testium to start your application before running the test. Testium will automatically stop your application once all tests finished.

### Step 3: Write Tests

```js
const assert = require('assert');

const browser = require('testium-mocha').browser;

describe('The homepage', () => {
  before(browser.beforeHook());

  before('Load homepage', () =>
    browser.loadPage('/')
  );

  it('shows the right title', () =>
    browser.getPageTitle().then((title) => {
      assert.equal(title, 'Hello');
    })
  );
});
```

### Step 4: Run Tests

You can just run the file above using `mocha`
and testium will handle everything for you:

* Launch the app and phantomjs.
* Wait for the app to listen.
* Clear cookies & reset the window to a well-known size.
* Close the webdriver session and tear down your app & phantomjs.

```sh
./node_modules/.bin/mocha my-test.js
# Or, after adding 'node_modules/.bin' to your $PATH:
mocha my-test.js
```

Thanks to [`rc`](https://www.npmjs.com/package/rc) you can use environment variables to override options.
This makes it super easy to run the same test in Chrome:

```sh
testium_browser=chrome mocha my-test.js
```

Testium will automatically download selenium and chromedriver the first time they are needed.
