---
title: Promise Chains
layout: sidebar
---

# [`Testium-driver-wd`](https://www.npmjs.com/package/Testium-driver-wd)

## General Notes

`browser` is just a `wd` session with some additional helpers.
Which means that all [methods in the `wd` docs](https://github.com/admc/wd/blob/master/doc/api.md) are expected to work.
This page omits many things that are already part of `wd`.

All methods return a Promise that can be chained.
So whenever it says "returns a string", read "returns the promise of a string".
The `wd` docs have [more examples for what that means](https://github.com/admc/wd/#q-promises--chaining).
This also implies that the returned promises have to be passed to the test runner (e.g. mocha), 
either by using async functions and `await` or by returning the promise.

Another thing to keep in mind is to always use `browser.loadPage` and never `browser.get` to load a page.
The reason is that `browser.loadPage` will properly capture status code and headers whereas `browser.get` will not.
This is different from using `wd` directly because capturing status codes is not a native `wd` feature.

### browser.capabilities`

An object describing the [WebDriver capabilities](https://code.google.com/p/selenium/wiki/JsonWireProtocol#Capabilities_JSON_Object) that the current browser supports.

This is **not** a promise, but a plain JavaScript object.

#### Example: Capability Check

```js
before(function requiresAlerts() {
  // Skip the current test suite unless the browser can work with alerts
  if (!browser.capabilities.handlesAlerts) this.skip();

  // skip when not Chrome browser
  if (browser.capabilities.browserName !== 'chrome') this.skip();
});
```

### browser.sessionCapabilities(): Promise\<browser.capabilities\>

Returns `browser.capabilities`. 

```js
const capabilities = await browser.sessionCapabilities()
```

### browser.getConsoleLogs([logLevel: string]): Promise\<ConsoleLogObj[]\>

Returns all log events for `logLevel` `all`(default) `log`,`warn`, `error`, `debug` since the last time this method was called.

```js
const errorLogs = await browser.getConsoleLogs('error');
```
**Warnings:** 
- Each browser implements this differently against the WebDriver spec.
- Logs are not persistent, and can't be retrieved multiple times.

```js

const errorLogs = await browser.getConsoleLogs('error'); // some errors exists
// get error logs once more. It will not contain the previous errors logs
const emptyErrors = await browser.getConsoleLogs('error'); 
``` 

### browser.getScreenshot(): Promise\<string\>

Returns screenshot as a base64 encoded PNG.

## Navigation

### browser.getHeader(name: string): Promise\<string\>

Returns the value of the response header with the provided name.
Header names should be provided in lowercase.

```js
const country = await browser.loadPage('/').getHeader('x-country')
```

### browser.getHeaders(): Promise\<Record\<string, string\>\>

Returns all response headers for the current page as a plain object.
All keys in the object will be lowercase,
e.g. `content-type` instead of `Content-Type`.

```js
const headers = await browser.loadPage('/').getHeaders()
```

### browser.getPath(): Promise\<string\>

Returns the current path and query of the page,
e.g. `/some/route?foo=bar`.

<span class="new-in">Added in: Testium-driver-wd v3.1.0</span>
`browser.getPath()` will return the path including hash, 
e.g. `/some/route?foo=bar#hash` 


### browser.getUrl(): Promise\<string\>

Returns the current absolute url of the page,
e.g. `http://localhost:1234/some/route`.

```js
const url = await browser.getUrl();

assert.ok(/^https:/.test(url));
```

### browser.getUrlObject(): Promise\<URLObject\>
<span class="new-in">Added in: Testium-driver-wd v3.1.0</span>

Returns a [WHATWG URL instance](https://nodejs.org/dist/latest-v13.x/docs/api/url.html#url_the_whatwg_url_api) of the current url.

```js
const urlObj = browser
  .loadPage('/#abc')
  .clickOn('#something') // do something that changes the url
  .getUrlObject()
```

### browser.getStatusCode(): Promise\<number\>

Returns the response status code for the current page.

```js
const statusCode = await browser.loadPage().getStatusCode();
```

### browser.loadPage(url: string, options?: loadPageOpts): Promise\<void\>

Navigates the browser to the specified relative or absolute url.

```js
await browser
  .loadPage('/products'); // implies assertStatusCode(200)
```

The following `loadPageOpts` options are supported:

* `query`: An object with additional query parameters.
* `headers`: An object with headers to inject.
* `expectedStatusCode`: Defaults to `200`, can be one of:
  * An integer status code that the response should equal
  * A RegExp that the status code (as a string) should match
  * A Function which takes the status code and returns `true` or `false` if it is acceptable

#### Example: Expecting a 302 status code

```js
await browser
  .loadPage('/page-that-redirects', { expectedStatusCode: 302, query: { foo: 'bar' } });
```  
  
#### Example: Running `loadPage` in Mocha `before` hook

To speed up your tests and short circuit test failures, run `loadPage` in the Mocha `before` hook. 

**_Note: Neither screenshot nor HTML page will be created in case there is an error in Mocha hooks._**

```js
describe('products', () => {
  before(browser.beforeHook());

  before(() =>
    browser.loadPage('/products')
  );

  it('works 1', /* ... */);
  it('works 2', /* ... */);
  it('works 3', /* ... */);
});
```

#### Example: Relative URL with Headers

If relative, the root is assumed to be `http://127.0.0.1:#{app.port}`,
where `app.port` refers to [the config option](/config.html#app-port).

```js
// Navigates to `http://127.0.0.1:${app.port}/products`
// passing along the `X-Custom-Header: Some Value` header.
await browser
  .loadPage('/products', { headers: { 'x-custom-header': 'Some-Value' } });
```

#### Example: Absolute URL

If the url is absolute, any methods that depend on the proxy (`getStatusCode` and `getHeaders`)
will not work.
This is a bug and will be fixed.

```js
await browser
  // Navigates to https://www.google.com/?q=Testium+api
  .loadPage('http://www.google.com', { query: { q: 'Testium api'} })
  // But this will fail because the URL was absolute:
  .getStatusCode();
```

### browser.refresh(): Promise\<void\>

Refresh the current page.

```js
await browser
  .loadPage('/')
  .clickOn('#something') // does something that changes the page
  .refresh();            // reload the current page
```

### browser.waitForPath(path: string|RegExp, timeout=5000: number): Promise\<void\>

Waits `timeout` ms for the browser to be at the specified `path`.
```js
await browser.waitForPath('/foo');
await browser.waitForPath('/foo', 10000); // with increased timeout
```
### browser.waitForUrl(url: string|RegExp, query?: Record\<string, string|number\>, timeout=5000: number): Promise\<void\>

Waits `timeout` ms for the browser to be at the specified `url`.

Using `query` instead of passing the query params in the `url`
allows the test to accept any order of query parameters.

#### Example: Order Doesn't Matter

```js
browser
  .loadPage('/products?count=15&start=30')
  // This will return immediately even though the order of `count` and `start`
  // is reversed.
  .waitForUrl('/products', { start: 30, count: 15 });
```

## Elements

### browser.clickOn(cssSelector: string): Promise\<void\>

Calls native `click()` on the Element found by the given `cssSelector`.

```js
await browser.clickOn('.button');
```

<span class="new-in">Changed in: Testium-driver-wd v3.0.0</span>
The selector passed into `clickOn()` must match **only one unique** element in the page. 
Otherwise `onClick()` will throw. Use `clickOnAll()` for multiple elements.


### browser.clickOnAll(cssSelector: string): Promise\<void\>
<span class="new-in">Added in: Testium-driver-wd v3.0.0</span>

Calls native `click()` on each Element found by the given `cssSelector`.

```js
await browser.clickOnAll('.foo, .bar, .elm');
```

### browser.moveTo(cssSelector: string, xOffset?: number, yOffset?: number): Promise\<void\>

Move the mouse by an offset of the specified element found by the given `cssSelector`.
`xOffset` and `yOffset` are optional.

```js
await browser.moveTo('.button');
```

### browser.getElement(cssSelector: string): Promise\<Element\>

Finds an element on the page using the `cssSelector` and returns an [Element](/api/wd/#element). 
If it fails to find an element matching the selector, it will reject with an error.

```js
const button = await browser.getElement('.button');
await button.isDisplayed();

// or as a promise chain
await browser.getElement('.button').isDisplayed();
```

### browser.getElementOrNull(cssSelector: string): Promise\<Element|null\>

Finds an element on the page using the `cssSelector`. Returns `null` if the element wasn't found.

```js
const button = await browser.getElementOrNull('.button1,.button2')
if (button) {
  assert.strictEqual(await button.text(), 'OK');
}
```

### browser.getElements(cssSelector: string): Promise\<Element[]\>

Finds all elements on the page using the `cssSelector` and returns an array of Elements.

### browser.waitForElementDisplayed(cssSelector: string, timeout=3000: number): Promise\<void\>

Waits for the element at `cssSelector` to exist and be visible, then returns the [Element](/api/wd/#element). 
Times out after `timeout` ms.

```js
await browser.waitForElementDisplayed('.foo');
await browser.waitForElementDisplayed('.bar', 5000);
```

### browser.waitForElementNotDisplayed(cssSelector: string, timeout=3000: number): Promise\<void\>

Waits for the element at `cssSelector` to exist and not be visible, then returns the [Element](/api/wd/#element). 
Times out after `timeout` ms.

```js
await browser.waitForElementNotDisplayed('.foo');
await browser.waitForElementNotDisplayed('.bar', 5000);
```

### browser.waitForElementExist(cssSelector: string, timeout=3000: number): Promise\<void\>

Waits for the element at `cssSelector` to exist, then returns the [Element](/api/wd/#element). 
Times out after `timeout` ms. Visibility is not considered.

```js
await browser.waitForElementExist('.foo');
await browser.waitForElementExist('.bar', 5000);
```

### browser.waitForElementNotExist(cssSelector: string, timeout=3000: number): Promise\<void\>

Waits for the element at `cssSelector` to not exist, then returns `null`. Times out after `timeout` ms.

```js
await browser.waitForElementNotExist('.foo');
```

### browser.assertElementHasText(selector: string, textOrRegex: string|RegExp): Promise\<Element\>

Asserts that the element at `selector` contains `textOrRegex`.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node does not contain the given `textOrRegex`.

```js
await browser.assertElementHasText('.user-name', 'someone');
// is the same as:
assert.strictEqual(await browser.getElement('.user-name').text(), 'someone');
```

### browser.assertElementLacksText(selector: string, textOrRegex: string|RegExp): Promise\<Element\>

Asserts that the element at `selector` does not contain `textOrRegex`.
Returns the element.

Inverse of `assertElementHasText`.
```js
await browser.assertElementLacksText('.user-name', 'someone');
```

### browser.assertElementHasValue(selector: string, textOrRegex: string|RegExp): Promise\<Element\>

Asserts that the element at `selector` does not have the value `textOrRegex`.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node's value is not `textOrRegex`.

```js
await browser.assertElementHasValue('.user-name', 'someone else');
// is the same as:
assert.strictEqual(browser.getElement('.user-name').value(), 'someone else');
```

### browser.assertElementLacksValue(selector: string, textOrRegex: string|RegExp): Promise\<Element\>

Asserts that the element at `selector` does not have the value `textOrRegex`.
Returns the element.

Inverse of `assertElementHasValue`.

```js
await browser.assertElementLacksValue('.user-name', 'someone else');
```

### browser.assertElementHasAttributes(selector: string, attributes: Record\<string, string\>): Promise\<Element\>

Asserts that the element at `selector` contains `attribute:value` pairs specified by `attributes` object.
Returns the element.

Throws exceptions if `selector` doesn't match a single node,
or that node does not contain the given `attribute:value` pairs.

```js
await browser.assertElementHasAttributes('.user-name', {
  text: 'someone',
  name: 'username',
});
```

### browser.assertElementIsDisplayed(selector: string): Promise\<Element\>

Asserts that the element at `selector` exists and is visible.
Returns the element.

```js
await browser.assertElementIsDisplayed('.user-name');
// is the same as
assert.ok(await browser.getElement('.user-name').isDisplayed());
```

### browser.assertElementNotDisplayed(selector: string): Promise\<Element\>

Asserts that the element at `selector` is not visible.
Returns the element.

Inverse of `assertElementIsDisplayed`.

### browser.assertElementExists(selector: string): Promise\<Element\>

Asserts that the element at `selector` exists.

```js
await browser.assertElementExists('.user-name');
```

### browser.assertElementDoesntExist(selector: string): Promise\<Element\>
Asserts that the element for `selector` doesn't exist.

Inverse of `assertElementExists`.

```js
await browser.assertElementDoesntExist('.user-name');
```


### browser.assertElementsNumber(selector: string, number: number): Promise\<Element\>

Asserts that `selector` matches exactly `number` elements.
```js
await browser.assertElementsNumber('.foo', 3) // must have exactly 3 elements
```
### browser.assertElementsNumber(selector:string, {min?: number, max?: number, equal?: number}): Promise\<Element[]\>
<span class="new-in">Added in: Testium-driver-wd v3.0.0</span>

Asserts that `selector` matches element numbers:

- `min`:  at least `min` amount of elements
- `max`: at most `max` amount of elements
- `equal`:  exactly `equal` amount of elements

```js
await browser.assertElementsNumber('.foo', { min: 1 });   // must have at least 5 elements
await browser.assertElementsNumber('.bar', { max: 5 });   // can have up to 5 elements
await browser.assertElementsNumber('.elm', { equal: 3 }); // must have exactly 3 elements
```


<a name="element"></a>

### element.click(): Node`

Clicks on the element, e.g. clicking on a button or a dropdown.

```js
const element = await browser.getElement('#foo');
element.click();
```

### element.get(attribute: string): Promise\<string|null\>

Returns the element's specified HTML `attribute`,
e.g. "value", "id", or "class".

```js
const classAttribute = await element.get('class')
```

### element.text(): Promise\<string\>

Returns the element's text contents.

Note that WebDriver (and therefore Testium) will not return text of hidden elements.

```js
const text = await element.text();
```

### element.isDisplayed(): Promise\<boolean\>

Returns `true` if the element is visible.

```js

const isDiplayed = await browser.getElement('#foo').isDiplayed();
if (isDiplayed) {
  // do something
} else {
  // do something else
}
```

## Forms

### browser.clear(cssSelector: string): Promise\<Element\>

Clears the input found by the given `cssSelector`.

```js
const value = await browser
  .clear('#input1')
  .getValue();
```

### browser.fillFields(fields: Record<string, string>): Promise\<Element\>

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

### browser.setValue(cssSelector: string, value: string|number): Promise\<Element\>

*Alias: browser.clearAndType*

Sets the value of the [Element](#element) with the given `cssSelector` to `value`.

```js
browser
  .setValue('.first-name', 'John')
  .setValue('.last-name', 'Smith')
  .clickOn('.submit');
```

### browser.type(cssSelector: string, value: string|number): Promise\<Element\>

Sets a value to the form element found for selector `cssSelector`.

```js
const value = await browser
  .type('#input1', 'foo')
  .getValue();
```

### element.clear(): Promise\<Element\>

Clears the input element.

```js
const value = await browser
  .getElement('#input1')
  .clear();
```

### element.type(value: string): Promise\<Element\>

Type text to the input element.

```js
const value = await browser
  .getElement('#input1')
  .type('foo');
```

## Page

### browser.getPageTitle(): Promise\<string\>

Returns the current page title.

```js
const title = await browser.getPageTitle();

assert.strictEqual(title, 'foo')
```

### browser.getPageSource(): Promise\<Element\>

Returns the current page's html source.
Using this method usually means
that you are trying to test something
that can be done without a browser.

```js
const pageSource = await browser.getPageSource();

assert.ok(/body/.test(pageSource));
```

Note that if the browser is presenting something like an XML or JSON response,
this method will return the presented HTML,
not the original XML or JSON response.
If you need to simply test such a response,
use a simpler test that doesn't involve a browser.
For example, you could use a normal HTTP client library to load a URL relative to `browser.appUrl`.

### browser.getPageSize(): Promise\<{width: number, height: number}\>

Returns the current window's size as an object with height and width properties in pixels.

```js
const pageSize = await browser.getPageSize();

assert.deepStrictEqual(pageSize, { width: 800, height: 600 });
```

This can be useful for responsive UI testing when combined with `browser.setPageSize`.
Testium defaults the page size to `height: 768` and `width: 1024`.

### browser.setPageSize({height: number, width: number}): Promise\<void\>

Sets the current window's size in pixels.

```js
// Resize window to a much smaller screen
await browser
  .setPageSize({ height: 400, width: 200 })
  .loadPage('/');
```

## Evaluate

Note that client-side functions will not have access to server-side values.
Return values have to cleanly serialize as JSON, otherwise the behavior is undefined.

### browser.evaluate(code: string): Promise\<any\>

**Warning:** Consider using the `function` variant below.
Having code with proper syntax highlighting and linting can prevent frustrating debugging sessions.

Executes the given javascript `code`.
It must contain a return statement in order to get a value back.

```js
const code = 'return document.querySelector(\'.menu\').style;';
const style = await browser.evaluate(code);
```

### browser.evaluate(args..., function(args...)): Promise\<any\>

Returns the result of the given function, invoked on the webdriver side 
(so you cannot bind its `this` object or access context variables via lexical closure).
If you provided `args`, Testium will marshals them as JSON and passes them to the function in the given order.

#### Example: Messing With Globals

```js
// Patching global state to influence the app's behavior.
browser.evaluate(function fakeConfigData() {
  window.myConfigBlob.magicalSetting = true;
});
```

#### Example: Passing Arguments

```js
// This will return the current url fragment
const hash = await browser.evaluate('hash', function getLocationProp(prop) {
  return window.location[prop];
});

// Passing multiple arguments
browser.evaluate('http://foo.bar', '/', '#hash', (domain, path, hash) => {
  window.location.href = `${domain}${path}${hash}`;
});
```


### browser.evaluateAsync(args..., function(args...)): Promise\<void\>

The executed script is required to return a Promise.

Returns a Promise of the given function, invoked on the webdriver side (so you
cannot bind its `this` object or access context variables via lexical closure).
If you provided `args`, Testium will marshal them as JSON and pass them to
the function in the given order.

**Note: NOT supported on PhantomJS**

#### Example: Fetch the data

```js
// Fetching data from the client
browser.evaluateAsync(async () => {
  const response = await fetch('http://example.com/movies.json');
  return response.json();
});
```

#### Example: Passing Arguments

```js
// Return the result after 1 sec
browser.evaluateAsync(3, 6, (a, b) => {
  // return a Promise
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(a * b);
    }, 1000);
  })
});
```

## Cookies

Cookie objects have the following interface:

```ts
interface Cookie {
  name: string
  value: string
  domain: string
  path: string
  expiry: number
  httpOnly: boolean
  secure: boolean
}
```

Note that the interface shown above is Testium-specific and **only** applies
when using `setCookieValue()` or `setCookieValues()`

### browser.setCookieValue(name :string, value: string): Promise\<void\>

Sets a cookie on the current page's domain to the value given.

```js
browser
  .setCookieValue('userId', '3') // set before loading the page
  .loadPage('/');
```

### browser.setCookieValues(Record<string, string>): Promise\<void\>

Given an object mapping cookie names to values, sets multiple cookies by
calling `setCookieValue()` the requisite number of times for you.

```js
browser
  .setCookieValues({
    userId: '3',
    dismissedPopup: 'true',
  })
  .loadPage('/');
```

### browser.getCookie(name: string): Promise\<string|undefined\>

Returns a cookie object: 
```ts
interface Cookie {
  name: string
  value: string
  domain: string
  path: string
  expiry: number
  httpOnly: boolean
  secure: boolean
}
```

```js
const cookie = await browser.getCookie('foo');

assert.strictEqual(cookie.value, 'bar');
```

### browser.getCookies(): Promise\<Record\<string, string\>[]\>

Returns an array of cookie objects with the structure:
```json5
{
  domain: '127.0.0.1',
  expiry: 1591781844,
  httpOnly: false,
  name: 'foo',
  path: '/',
  secure: false,
  value: 'bar'
}
```

```js
const cookies = await browser.getCookies();

assert.ok(cookies.every(cookie => cookie.secure));
```

### browser.clearCookies(): Promise\<void\>

Deletes all cookies.

```js
browser.clearCookies();
```

### browser.clearCookie(name: string): Promise\<void\>

Deletes a cookie by `name`.

```js
browser.clearCookie('_ga');
```

## Alerts & Confirms

This API allows you to interact with
alert, confirm, and prompt dialogs.

Some browsers, notably phantomjs,
don't support this part of the WebDriver spec.
You can guard against this by checking
the [Capabilities](#browsercapabilities) object.

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

### browser.getAlertText(): Promise\<string\>

Get the text of a visible alert, prompt, or confirm a dialog.

```js
const alertText = await browser.getAlertText();
```

### browser.acceptAlert(): Promise\<void\>

Accepts a visible alert, prompt, or confirm a dialog.

```js
browser.acceptAlert();
```

### browser.dismissAlert(): Promise\<void\>

Dismisses a visible alert, prompt, or confirm dialog.

```js
browser.dismissAlert();
```

### browser.typeAlert(value: string): Promise\<void\>

Types into a visible prompt dialog.

```js
browser.typeAlert('');
```

## Windows & Frames

### browser.closeWindow(): Promise\<void\>

Close the currently focused window.

```js
browser
  .clickOn('#open-popup')
  .switchToWindow('popup1')
  .closeWindow()
  .switchToDefaultWindow();
```

The name used to identify the popup can be set in the code used to create it.
It's the `strWindowName` in [`window.open(strUrl, strWindowName, [strWindowFeatures])`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open).

### browser.switchToDefaultFrame(): Promise\<void\>

Switch focus to the default frame (i.e. the actual page).

```js
browser
  .switchToFrame('some-frame')
  .clickOn('#some-button-in-frame')
  .switchToDefaultFrame();
```

### browser.switchToFrame(id: string): Promise\<void\>

Change focus to another frame on the page.
```js
browser.switchToFrame('some-frame')
```

### browser.switchToDefaultWindow(): Promise\<void\>

Switch focus to the window that was most recently referenced by `loadPage`. Useful when interacting with popup windows.

```js
browser
  .loadPage('/path')
  .clickOn('#open-popup')
  .switchToWindow('popup1')
  .clickOn('#some-button-in-popup')
  .closeWindow()
  .switchToDefaultWindow();
```

### browser.switchToWindow(name: string): Promise\<void\>

Switch focus to the window with name `name`.

```js
await browser.switchToWindow('popup1');
```

## Lighthouse & Puppeteer

Lighthouse and Puppeteer require Chromedriver and headless Chrome to be present in the environment.

### browser.emulate(deviceDescriptor: string): Promise\<void\>

Emulates a device given its name. 
A list of device descriptor names can be found at [puppeteers deviceDescriptors](https://github.com/puppeteer/puppeteer/blob/master/src/DeviceDescriptors.ts)

```js
browser.emulate('iPhone X').loadPage();
```


### browser.withPuppeteerPage(function(page: PuppeteerPage): any): Promise\<void\>
Connects Puppeteer to the Chromedriver and exposes the [Puppeteer `Page` api](https://github.com/puppeteer/puppeteer/blob/master/docs/api.md#class-page)

```js
const Page = await browser.withPuppeteerPage();
```

### browser.getLighthouseData(flags?: Record\<string, any\>, config? Record\<string, any\>): Promise\<LHResultObj\>
Returns Lighthouse and returns Lighthouse raw result data. Information regarding `flags` and `config` 
can be found in [the official Lighthouse docs](https://github.com/GoogleChrome/lighthouse/tree/master/docs#using-programmatically)

It is advised to increase the Mocha test timeout when running more than one Lighthouse audit at once.
```js
const flags = {
  chromeFlags: [
    '--disable-gpu',
    '--headless',
    '--disable-storage-reset',
    '--enable-logging',
    '--disable-device-emulation',
    '--no-sandbox',
  ],
};
const config = {
  extends: 'lighthouse:default',
  settings: {
    onlyCategories: ['performance', 'seo', 'accessibility'],
    throttlingMethod: 'simulate',
  },
};

it('my Lighthouse test', async function() {
  this.timeout(20000); // set Mocha timeout to 20 seconds

  const rawResults = await browser
    .loadPage('/')
    .getLighthouseData(flags, config);
  
  // ...
});
```


### browser.runLighthouseAudit(flags?: Record\<string, any\>, config? Record\<string, any\>): Promise\<parsedLHResultObj\>

Runs the lighthouse tests on the current page loaded by `loadPage` and resolves to a result object
containing score and list of error list.

```js
const rawResults = await browser
  .loadPage('/path')
  .runLighthouseAudit();
```
By default, the audit is running in "Mobile" mode. To run in "Desktop" mode,
we need to pass the `--disable-device-emulation` flag as shown in the example below.

```js
let lhResults;

before(async () => {
  lhResults = await browser
    .loadPage('/path')
    .runLighthouseAudit({
      chromeFlags: ['--disable-device-emulation'], // To run audit in Desktop mode
    });
});

it(`checking for score over 85`, () =>
  assert.ok(lhResults.isSuccess(85), `score is only ${lhResults.score}`)
);

it(`There are no errors`, () =>
  assert.strictEqual(Object.keys(lhResults.errors()).length, 0)
);
```

#### parsed LH result object
```ts
interface parsedLHResultObj {
  audits: Record<string, Object>                // Raw Lighthouse audit results
  score: number                                 // Success score (success / total)
  errors(): Record<string,any>[]                // Returns an array of all failed audit results
  errors(audit :string): Record<string,any>     // Returns a specific failed audit
  errorString(): string
  isSuccess(expectedScore: number): void        // Asserts that the results meet the expected score
  success(): Record<string, any>[]              // Returns an array of all successful audit results
  success(audit: string): Record<string, any>   // Returns a specific audit result
}
```

### browser.a11yAudit({ignore?: function|string[], flags?: Record<string, any>, config? Record\<string, any\>}): Promise\<void\>
Runs the accessibility issues on the current page loaded by `loadPage`. 
`flags`, `config` and `ignore` are optional parameters.

`ignore` is the ids of issues to ignore, a string array

```js
const issues = await browser
  .loadPage('/path')
  .a11yAudit();

assert.strictEqual(issues.length, 0)
```

```js
const issues = await browser
  .loadPage('/path')
  .a11yAudit({ 
    ignore: ['color-contrast', 'meta-viewport'], // ignore 'color-contrast' and 'meta-viewport'
  });

assert.strictEqual(issues.length, 0);
```

### assertAccessibilityScore(minScore: number, skipAudits: string[]): Promise\<parsedLHResultObj\>
Runs Lighthouse `accessibility` audit.

```js
browser.loadPage('/').assertAccessibilityScore(40);
```

To disable specific audits, pass the optional array containing the category's audit IDs.
[Accessibility audit IDs](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js#L451-L502)

```js
browser.loadPage('/').assertAccessibilityScore(40, ['listitem']);
```

### assertBestPracticesScore(minScore: number, skipAudits: string[]]): Promise\<parsedLHResultObj\>
Runs Lighthouse `Best Practices` audit.

```js
browser.loadPage('/').assertBestPracticesScore(40);
```

To disable specific audits, pass the optional array containing the category's audit IDs.
[Best Practices audit IDs](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js#L508-L524)

```js
browser.loadPage('/').assertBestPracticesScore(40, ['js-libraries']);
```

### assertPerformanceScore(minScore: number, skipAudits: string[]): Promise\<parsedLHResultObj\>
Runs Lighthouse `Performance` audit. 
The following performance audits are skipped by default: `final-screenshot`, `is-on-https`, `screenshot-thumbnails`,

```js
browser.loadPage('/').assertPerformanceScore(40);
```

To disable specific audits, pass the optional array containing the category's audit IDs.
[Performance audit IDs](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js#L392-L439)

```js
browser.loadPage('/').assertPerformanceScore(40, ['font-display']);
```


### assertPwaScore(minScore: number, skipAudits: string[]): Promise\<parsedLHResultObj\>
Runs Lighthouse `PWA` audit.

```js
browser.loadPage('/').assertPwaScore(40);
```

To disable specific audits, pass the optional array containing the category's audit IDs.
[PWA audit IDs](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js#L555-L574)

```js
browser.loadPage('/').assertPwaScore(40, ['apple-touch-icon']);
```

### assertSeoScore(minScore: number, skipAudits: string[]): Promise\<parsedLHResultObj\>
Runs Lighthouse `SEO` audit.

```js
browser.loadPage('/').assertSeoScore(40);
```

To disable specific audits, pass the optional array containing the category's audit IDs.
[SEO audit IDs](https://github.com/GoogleChrome/lighthouse/blob/master/lighthouse-core/config/default-config.js#L532-L546)

```js
browser.loadPage('/').assertPwaScore(40, ['structured-data']);
```
