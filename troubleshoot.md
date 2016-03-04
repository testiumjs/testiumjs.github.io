---
title: Troubleshoot
layout: sidebar
category: intro
---

## Troubleshoot

Sometimes your tests suite crashes in unexpected ways.
When that happens, look at the logs under `test/log`.
These files record the internals of testium.

### application.log

The output your app generated while running.

### phantomjs.log

*Only when using phantomjs.*

This is where you can look at what happened from PhantomJS' perspective.
It can be helpful to see client side errors and logs.

### proxy.log

This is the log from the internal proxy server.
The proxy server is used to capture data not provided by the WebDriver spec itself.
This log shows incoming requests and outgoing responses.

If your tests hang, check this log for something that came in, but didn't go back out.

### selenium.log

*Only when not using phantomjs.*

This is the log from the selenium server itself.
It includes verbose ouput.
You can see what's happening inside selenium, which will be mostly unhelpful.
However, you can also see when requests come in and how they are handled, including responses.

### webdriver.log

*Only when not using phantomjs.*

This is the log of which webdriver actions were performed.
It's not usually very helpful,
but it will let you know if testium saw your command before the process failed.

### Hanging Processes

The process handling of testium is supposed to clean up after itself,
even after errors are thrown.
However, this may not be the case 100% of the time.

Check for any running node processes (pointing to testium's proxy.js file)
or java processes (with the selenium jar).
If you see these but aren't currently running tests,
something left them abandoned.
You should kill those processes before trying again.

### Error: Error communicating with the remote browser. It may have died.

Too much client interaction within a single test will sometimes kill the browser.  Try to split large `it` blocks with multiple interactions & assertions into smaller blocks.
