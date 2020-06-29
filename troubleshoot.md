---
title: Troubleshoot
layout: sidebar
category: intro
---

## Troubleshooting

### Logs

Sometimes your tests suite crashes in unexpected ways.
When that happens, look at the logs under `test/log`.
These files record the internals of Testium.

* `application.log`:
  The output your app generated while running.
* `chromedriver.log`:
  These are the logs from `chromedriver` and contain chromedriver actions and client side network and Javascript errors
  *Only available when using chromedriver.*
* `phantomjs.log`:
  This is where you can look at what happened from PhantomJS' perspective.
  It can be helpful to see client side errors and logs.
  *Only available when using phantomjs.*
* `proxy.log`:
  This is the log from the internal proxy server.
  The proxy server is used to capture data not provided by the WebDriver spec itself.
  This log shows incoming requests and outgoing responses.
  If your tests hang, check this log for something that came in, but didn't go back out.
* `selenium.log`:
  This is the log from the selenium server itself.
  It includes verbose output.
  You can see what's happening inside selenium, which will be mostly unhelpful.
  However, you can also see when requests come in and how they are handled, including responses.
  *Only available when not using phantomjs.*

### Debugging Mocha Tests

Sometimes stepping through tests can help better understand why and where things fail.
To prepare do the following:

1. Open `chrome://inspect` in Chrome.
2. Click "Open dedicated DevTools for Node".
3. Add a `debugger;` statement to a test file just before the line where you want to pause.

Then it's possible to use the following command to start a test run with the debugger enabled:

```console
node --inspect ./node_modules/.bin/_mocha test/integration
```

The above just runs the whole `test/integration` directory, 
but it's also possible to run individual test files or pass additional mocha options.
To run the tests in a specific browser, the command can be prefixed with `testium_browser=chrome`.

### Hanging Processes

The process handling of Testium is supposed to clean up after itself,
even after errors are thrown.
However, this may not be the case 100% of the time.

Check for any running node processes (pointing to Testium's proxy.js file)
or java processes (with the selenium jar).
If you see these but aren't currently running tests,
something left them abandoned.
You should kill those processes before trying again.

### Error: Error communicating with the remote browser. It may have died.

Too much client interaction within a single test will sometimes kill the browser. 
Try to split large `it` blocks with multiple interactions & assertions into smaller blocks.
