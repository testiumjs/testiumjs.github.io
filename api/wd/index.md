---
title: Promise Chains
layout: sidebar
---

## [`testium-driver-wd`](https://www.npmjs.com/package/testium-driver-wd)

### General Notes

`browser` is just a `wd` session with some additional helpers.
Which means that all [methods in the `wd` docs](https://github.com/admc/wd/blob/master/doc/api.md) are expected to work.
This page omits many things that are already part of `wd`.

All methods return a Promise that can be chained.
So whenever it says "returns a string", read "returns the promise of a string".
The `wd` docs have [more examples for what that means](https://github.com/admc/wd/#q-promises--chaining).
This also implies that the returned promises have to be passed to the test runner (e.g. mocha), either by using async functions and `await` or by returning the promise.
Since all return values are promises,
the examples assume that `assert` refers to a library that is promise-aware like [`assertive-as-promised`](https://github.com/groupon/assertive-as-promised).

Another thing to keep in mind is to always use `browser.loadPage` (or `browser.navigateTo`) and never `browser.get` to load a page.
The reason is that `browser.loadPage` will properly capture status code and headers whereas `browser.get` will not.
This is different from using `wd` directly because capturing status codes is not a native `wd` feature.

#### `browser.capabilities`

An object describing the [WebDriver capabilities](https://code.google.com/p/selenium/wiki/JsonWireProtocol#Capabilities_JSON_Object) that the current browser supports.

This is **not** a promise but a plain JavaScript object.

##### Example: Capability Check

```js
before(function requiresAlerts() {
  // Skip the current test suite unless the browser can work with alerts
  if (!browser.capabilities.handlesAlerts) this.skip();
});
```

#### `browser.getConsoleLogs(logLevel='all')`

Returns all log events with `logLevel` (log/warn/error/debug) since the last time this method was called.
**Warning:** Each browser implements this differently against the WebDriver spec.

```js
var errorLogs = browser.getConsoleLogs('error');
```

#### `browser.getScreenshot()`

Returns screenshot as a base64 encoded PNG.

#### `browser.assertImgLoaded(selector)`

*Not implemented yet.*

Asserts that the image element at `selector` has both loaded and been decoded successfully.

```js
browser.assertImgLoaded('.logo');
```

### Navigation

#### `browser.getHeader(name)`

Returns the value of the response header with the provided name.
Header names should be provided in lowercase.

#### `browser.getHeaders()`

Returns all response headers for the current page as a plain object.
All keys in the object will be lowercase,
e.g. `content-type` instead of `Content-Type`.

#### `browser.getPath()`

Returns the current path of the page,
e.g. `/some/route`.

#### `browser.getUrl()`

Returns the current absolute url of the page,
e.g. `http://localhost:1234/some/route`.

```js
assert.match(/^https:/, browser.getUrl());
```

#### `browser.getStatusCode()`

Returns the response status code for the current page.

#### `browser.assertStatusCode(statusCode)`

Asserts that the most recent response status code is `statusCode`.

```js
browser
  .loadPage('/products');
  .assertStatusCode(200);
```

This is especially useful as a method to short circuit test failures.

```js
describe('products', function() {
  before(browser.beforeHook());
  before(() =>
    browser.loadPage('/products')
  );

  it('works 1', /* ... */);
  it('works 2', /* ... */);
  it('works 3', /* ... */);
});
```

#### `browser.loadPage(url, options)`

Navigates the browser to the specificed relative or absolute url.
The following options are supported:

* `query`: An object with additional query parameters.
* `headers`: An object with headers to inject.
* `expectedStatusCode`: Defaults to `200`, can be one of:
  * An integer status code that the response should equal
  * A RegExp that the status code (as a string) should match
  * A Function which takes the status code and returns `true` or `false` if it is acceptable

#### `browser.navigateTo(url, options)` (deprecated)

This does the same thing as `browser.loadPage()`, but:

1. Doesn't support the `expectedStatusCode` option
1. Doesn't default to asserting status code 200

##### Example: Relative URL with Headers

If relative, the root is assumed to be `http://127.0.0.1:#{app.port}`,
where `app.port` refers to [the config option](/config.html#app-port).

```js
// Navigates to `http://127.0.0.1:${app.port}/products`
// passing along the `X-Custom-Header: Some Value` header.
browser.loadPage('/products', {
  headers: {
    'X-Custom-Header': 'Some-Value'
  }
});
```

##### Example: Absolute URL

If the url is absolute,
any methods that depend on the proxy
(`getStatusCode` and `getHeaders`)
will not work.  (e.g. `browser.loadPage`)
This is a bug and will be fixed.

```js
browser
  // Navigates to https://www.google.com/?q=testium+api
  .navigateTo('http://www.google.com', { query: { q: 'testium api'} });
  // But this will fail because the URL was absolute:
  .getStatusCode();
```

#### `browser.refresh()`

Refresh the current page.

#### `browser.waitForPath(path, timeout=5000)`

Waits `timeout` ms for the browser to be at the specified `path`.
`path` can be a String, or a regular expression.

#### `browser.waitForUrl(url, timeout=5000)`

Waits `timeout` ms for the browser to be at the specified `url`.

#### `browser.waitForUrl(url, query, timeout=5000)`

Waits `timeout` ms for the browser to be at the specified `url` with query parameters per the `query` object.
`url` can be a String, or a regular expression.

Using `query` instead of passing the query params in the `url`
allows the test to accept any order of query parameters.

##### Example: Order Doesn't Matter

```js
browser
  .loadPage('/products?count=15&start=30')
  // This will return immediately even though the order of `count` and `start`
  // is reversed.
  .waitForUrl('/products', { start: 30, count: 15 });
```

### Elements

#### `browser.clickOn(cssSelector)`

Calls `clickOn()` on the Element found by the given `cssSelector`.

```js
browser.clickOn('.button');
```

#### `browser.getElement(cssSelector)`

Finds an element on the page using the `cssSelector` and returns an [Element](/api/wd/#element).  If it fails to find an element matching the selector, it will reject with an error.

```js
var button = browser.getElement('.button');
button.isDisplayed();
```

#### `browser.getElementOrNull(cssSelector)`

Finds an element on the page using the `cssSelector`. Returns `null` if the element wasn't found.

```js
browser.getElementOrNull('.button1,.button2')
  .then(button => assert.equal('OK', button && button.text()));
```

#### `browser.getElements(cssSelector)`

Finds all elements on the page using the `cssSelector` and returns an array of Elements.

#### `browser.waitForElementDisplayed(cssSelector, timeout=3000)`

Waits for the element at `cssSelector` to exist and be visible, then returns the [Element](/api/wd/#element). Times out after `timeout` ms.

#### `browser.waitForElementNotDisplayed(cssSelector, timeout=3000)`

Waits for the element at `cssSelector` to exist and not be visible, then returns the [Element](/api/wd/#element). Times out after `timeout` ms.

#### `browser.waitForElementExist(cssSelector, timeout=3000)`

Waits for the element at `cssSelector` to exist, then returns the [Element](/api/wd/#element). Times out after `timeout` ms. Visibility is not considered.

#### `browser.waitForElementNotExist(cssSelector, timeout=3000)`

Waits for the element at `cssSelector` to not exist, then returns `null`. Times out after `timeout` ms.

#### `browser.assertElementHasText(selector, textOrRegex)`

Asserts that the element at `selector` contains `textOrRegex`.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node does not contain the given `textOrRegex`.

```js
browser.assertElementHasText('.user-name', 'someone');
// is the same as:
assert.equal('someone', browser.getElement('.user-name').text());
```

#### `browser.assertElementLacksText(selector, textOrRegex)`

Asserts that the element at `selector` does not contain `textOrRegex`.
Returns the element.

Inverse of `assertElementHasText`.

#### `browser.assertElementHasValue(selector, textOrRegex)`

Asserts that the element at `selector` does not have the value `textOrRegex`.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node's value is not `textOrRegex`.

```js
browser.assertElementHasValue('.user-name', 'someone else');
// is the same as:
assert.equal('someone else', browser.getElement('.user-name').value());
```

#### `browser.assertElementLacksValue(selector, textOrRegex)`

Asserts that the element at `selector` does not have the value `textOrRegex`.
Returns the element.

Inverse of `assertElementHasValue`.

#### `browser.assertElementHasAttributes(selector, attributesObject)`

Asserts that the element at `selector` contains `attribute:value` pairs specified by attributesObject.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node does not contain the given `attribute:value` pairs.

```js
browser.assertElementHasAttributes('.user-name', {
  text: 'someone',
  name: 'username',
});
```

#### `browser.assertElementIsDisplayed(selector)`

Asserts that the element at `selector` exists and is visible.
Returns the element.

```js
browser.assertElementIsDisplayed('.user-name');
// is the same as
assert.expect(browser.getElement('.user-name').isDisplayed());
```

#### `browser.assertElementNotDisplayed(selector)`

Asserts that the element at `selector` is not visible.
Returns the element.

Inverse of `assertElementIsDisplayed`.


#### `browser.assertElementExists(selector)`

Asserts that the element at `selector` exists.
Returns the element.

```js
browser.assertElementExists('.user-name');
```

#### `browser.assertElementDoesntExist(selector)`

Asserts that the element at `selector` doesn't exist.
Returns the element.

Inverse of `assertElementExists`.

<a name="element"></a>

#### `element.click()`

Clicks on the element, e.g. clicking on a button or a dropdown.

#### `element.get(attribute)`

Returns the element's specified HTML `attribute`,
e.g. "value", "id", or "class".

#### `element.text()`

Returns the element's text contents.

Note that WebDriver (and therefore testium)
will not return text of hidden elements.

```js
var elementId = element.get('id');
```

#### `element.isDisplayed()`

Returns `true` if the element is visible.

### Forms

#### `browser.clear(cssSelector)`

Clears the input found by the given `cssSelector`.

#### `browser.fillFields(fields)`

Convenience method for setting the value for multiple form fields at once.
`fields` is an object that maps css selectors to values.

```js
browser.fillFields({
  '.first-name': 'Jamie',
  '.last-name': 'Smith',
});
// Equivalent to:
browser
  .setValue('.first-name', 'Jamie')
  .setValue('.last-name', 'Smith');
```

#### `browser.setValue(cssSelector, value)`

*Alias: browser.clearAndType*

Sets the value of the [Element](#element) with the given `cssSelector` to `value`.

```js
browser
  .setValue('.first-name', 'John')
  .setValue('.last-name', 'Smith')
  .click('.submit');
```

#### `browser.type(cssSelector, value)`

Sends `value` to the input found by the given `cssSelector`.

### `element.clear()`

Clears the input element.

### `element.type(strings...)`

Sends `strings...` to the input element.

### Page

#### `browser.getPageTitle()`

Returns the current page title.

#### `browser.getPageSource()`

Returns the current page's html source.
Using this method usually means
that you are trying to test something
that can be done without a browser.

```js
assert.match(/body/, browser.getPageSource());
```

Note that if the browser is presenting something like an XML or JSON response,
this method will return the presented HTML,
not the original XML or JSON response.
If you need to simply test such a response,
use a simpler test that doesn't involve a browser.
For example you could use a normal HTTP client library to load a URL relative to `browser.appUrl`.

#### `browser.getPageSize()`

Returns the current window's size
as an object with height and width properties
in pixels.

```js
assert.deepEqual({ width: 800, height: 600 }, browser.getPageSize());
```

This can be useful for responsive UI testing
when combined with `browser.setPageSize`.
Testium defaults the page size to
`height: 768` and `width: 1024`.

#### `browser.setPageSize({height, width})`

Sets the current window's size in pixels.

```js
browser.setPageSize({ height: 400, width: 200 });
// Window has resized to a much smaller screen
```

### Evaluate

Note that client-side functions
will not have access to server-side values.
Return values have to cleanly serialize as JSON,
otherwise the behavior is undefined.

#### `browser.evaluate(code)`

**Warning:** Consider using the `function` variant below.
Having code with proper syntax highlighting and linting can prevent frustrating debugging sessions.

Executes the given javascript `code`.
It must contain a return statement in order to get a value back.

```js
var code = 'return document.querySelector(\'.menu\').style;';
var style = browser.evaluate(code);
```

#### `browser.evaluate(args..., function(args...))`

Returns the result of the given function, invoked on the webdriver side (so you can not bind its `this` object or access context variables via lexical closure).
If you provided `args`, testium will marshals them as JSON and passes them to the function in the given order.

##### Example: Messing With Globals

```js
// Patching global state to influence the app's behavior.
browser.evaluate(function fakeConfigData() {
  window.myConfigBlob.magicalSetting = true;
});
```

##### Example: Passing Arguments

```js
// This will return the current url fragment
browser.evaluate('hash', function getLocationProp(prop) {
  return window.location[prop];
});
```

### Cookies

Cookie objects have the following general structure:

```js
{
  "name",
  "value",
  "path" = "/",
  "domain" = currentPageDomain,
  "expiry" = endOfCurrentSession,
  "secure" = false,
}
```

Note that the defaults shown above are testium-specific and **only** applied
if you use `setCookieValue()` or `setCookieValues()` (recommended)

#### `browser.setCookieValue(name, value)`

Sets a cookie on the current page's domain to the value given.
You can set cookies before loading your first page.
Testium tells the browser to load a blank page first
so that cookies can be set before loading a real page.

```js
browser.setCookieValue('userId', '3');
```

#### `browser.setCookieValues({ [name]: value })`

Given an object mapping cookie names to values, sets multiple cookies by
calling `setCookieValue()` the requisite number of times for you.

```js
browser.setCookieValues({
  userId: '3',
  dismissedPopup: 'true',
});
```

#### `browser.setCookie(Cookie)`

This is the built-in `wd` `setCookie()` function, and applies none of the
defaults above, and is only recommended for specific advanced uses.

#### `browser.setCookies([Cookie])` **[DEPRECATED]**

Sets all cookies in the array.

```js
cookies = [
  { name: 'userId', value: '3' },
  { name: 'dismissedPopup', value: 'true' },
];
browser.setCookies(cookies);
```

This function calls `setCookie()`, and applies none of the useful testium
defaults described above; it is now recommended that you instead use
`setCookieValues()`

#### `browser.getCookie(name)`

Returns the cookie visible to the current page with `name`.

```js
var userIdCookie = browser.getCookie('userId');
assert.equal('3', userIdCookie.value);
```

#### `browser.getCookies()`

Returns all cookies visible to the current page.

#### `browser.clearCookies()`

Deletes all cookies visible to the current page.

#### `browser.clearCookie(name)`

Delete a cookie by `name` that is visible to the current page.

### Alerts & Confirms

This API allows you to interact with
alert, confirm, and prompt dialogs.

Some browsers, notably phantomjs,
don't support this part of the WebDriver spec.
You can guard against this by checking
the [Capabilities](#capabilities) object.

```js
describe('Alert-based tests', function() {
  before(browser.beforeHook());

  before(function requiresAlerts() {
    // Skip the current test suite unless the browser can work with alerts
    if (!browser.capabilities.handlesAlerts) this.skip();
  });

  // <insert actual test cases here>
});
```

#### `browser.getAlertText()`

Gets the text of a visible
alert, prompt, or confirm dialog.

```js
var alertText = browser.getAlertText();
```

#### `browser.acceptAlert()`

Accepts a visible
alert, prompt, or confirm dialog.

```js
browser.acceptAlert();
```

#### `browser.dismissAlert()`

Dismisses a visible
alert, prompt, or confirm dialog.

```js
browser.dismissAlert();
```

#### `browser.typeAlert(value)`

Types into a visible prompt dialog.

```js
browser.typeAlert('');
```

### Windows & Frames

#### `browser.closeWindow()`

Close the currently focused window.

```js
browser.clickOn('#open-popup');
browser.switchToWindow('popup1');
browser.closeWindow();
browser.switchToDefaultWindow();
```

The name used to identify the popup can be set in the code used to create it.
It's the `strWindowName` in [`window.open(strUrl, strWindowName, [strWindowFeatures])`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open).

#### `browser.switchToDefaultFrame()`

Switch focus to the default frame (i.e., the actual page).

```js
browser.switchToFrame('some-frame');
browser.clickOn('#some-button-in-frame');
browser.switchToDefaultFrame();
```

#### `browser.switchToFrame(id)`

Switch focus to the frame with name or id `id`.

#### `browser.switchToDefaultWindow()`

Switch focus to the window that was most recently referenced by `loadPage`. Useful when interacting with popup windows.

```js
browser.loadPage('/path');
browser.clickOn('#open-popup');
browser.switchToWindow('popup1');
browser.clickOn('#some-button-in-popup');
browser.closeWindow();
browser.switchToDefaultWindow();
```

#### `browser.switchToWindow(name)`

Switch focus to the window with name `name`.
