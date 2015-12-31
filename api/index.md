---
title: Core API
layout: sidebar
---

## Core API

### Mocha Bindings: [`testium-mocha`](https://www.npmjs.com/package/testium-mocha)

There are two general ways of interacting with the mocha bindings.
The first (and older) way is to use the `browser` property on the test context:

```js
var injectBrowser = require('testium-mocha');

before(injectBrowser());

it('has a browser', function() {
  return this.browser.navigateTo('/');
});
```

The problem is that this works very poorly with ES6 arrow functions since it depends on the implicit test context.
The more ES6-friendly interface is:

```js
var browser = require('testium-mocha').browser;

before(browser.beforeHook());

it('is the browser', () => browser.navigateTo('/'));
```

**Warning:** Testium switches out the prototype of `browser` to turn the browser export "magically" into a functioning object after successful initialization.
Trying to call anything but `beforeHook` before the hook finished will not work as expected.

#### `injectBrowser(options = {})`

Returns a [mocha before hook](http://mochajs.org/#hooks).
The hook will initialize testium (if it didn't happen already)
and apply a number of changes to the current test suite:

* Intercept errors and automatically save screenshots.
* Override the default timeouts using the [`mocha.timeout`](/config.html#mocha-timeout) setting.
* Install an after hook to ensure everything is properly cleaned up.
* Inject a `browser` property onto the test context.
* Switch out the prototype of the [`browser` export](/api/#browser) to point to [`testium.browser`](/api/#testium-browser).

The `options` are forwarded to [`getTestium`](/api/#gettestium-options)

#### `browser`

After `browser.beforeHook` has been executed,
this object will be a proxy to [`testium.browser`](/api/#testium-browser).

#### `browser.beforeHook`

*This is an alias for [`injectBrowser`](/api/#injectbrowser-options).*

### Low-Level API: [`testium-core`](https://www.npmjs.com/package/testium-core)

Most of the time it won't be neccessary to interact with `testium-core` directly.
But when implementing a new kind of `browser` interface or integrating with a different test framework,
the information below might be helpful.

#### `getBrowser(options)`

Convenience method that calls [`getTestium`](/api/#gettestium-options) and then returns the [`browser` property](/api/#testium-browser).

#### `getConfig()`

Retrieves the config testium will use.
This can be useful for selectively disabling tests in certain browsers:

```js
var getConfig = require('testium-core').getConfig;

describe('Feature that only works in real browsers', function() {
  if (getConfig().get('browser') === 'phantomjs') {
    return xit('Not supported in PhantomJS');
  }

  // Actual test code.
});
```

The advantage over [`testium.config`](/api/#testium-config) is that `getConfig()` returns the value immediately without having to wait for the init to finish.

#### `getTestium(options)`

Returns a Promise of an object (`testium`) with the following properties:

* [`browser`](/api/#testium-browser): The actual browser to interact with.
* [`config`](/api/#testium-config): An Map-like object that can be used to read and write config values.
* [`getInitialUrl`](/api/#testium-getinitialurl):
  Function returning a URL to load before running any tests.
* [`getNewPageUrl`](/api/#testium-getnewpageurl-url-options):
  Function that generates a proxy-friendly URL.

The `options` allow for overriding a limited number of config options.
Supported are `driver`, `keepCookies`, and `reuseSession`.

#### `testium.browser`

An instance of the requested driver implementation.
The exact interface depends on the value of the [`driver`](/config.html#driver) setting:

* For "sync" see: [Synchronous Interface](/api/sync/)
* For "wd" see: [Promise Chains](api/wd/)

The browser always has an `appUrl` property which contains the base url of the application under test.

#### `testium.config`

Exposes `has`/`get`/`set` methods to interact with the config.
It will throw when someone attempts to read a config key that has no value.
Accepts [lodash-style](https://lodash.com/docs#get) property paths as keys.

#### `testium.getInitialUrl()`

Returns a URL to load before running any tests.
This ensures that `setCookie` calls work before loading the first real page.

#### `testium.getNewPageUrl(url, options = {})`

Generates a proxy-friendly URL.
Drivers use this when implementing `navigateTo` to get status code and header information among other things.
Available options:

* `query`: Additional query parameters to add to the url.
* `headers`: Headers to inject into the request.
