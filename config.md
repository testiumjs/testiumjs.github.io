---
title: Configuration
layout: sidebar
category: intro
---

## Configuration

Testium uses [`rc`](https://www.npmjs.com/package/rc) for loading configuration.
Be aware that command line arguments won't work
since most test runners (e.g. `mocha`) have their own argv handling.

### Top-Level Options

#### `browser`

The name of the browser to use.
If no browser is specified, Testium will assume it's `'phantomjs'`.
Typical values are `phantomjs`, `chrome`, or `firefox`.
The latter two depend on which browsers are installed on your computer.

A fully featured selenium grid might support many more browsers.
The webdriver docs contain a [list of known values for `browserName`](https://code.google.com/p/selenium/wiki/DesiredCapabilities#Used_by_the_selenium_server_for_browser_selection).

#### `desiredCapabilities`

Additional desired [webdriver capabilities](https://code.google.com/p/selenium/wiki/DesiredCapabilities).
The `browserName` field is ignored. It's read from `browser`.
This is mostly useful when running against a dedicated selenium grid.
It allows you to describe what kind of browser and OS you need
and the grid will create a session for the best match it can find.

#### `driver`

Determines the interface for interacting with the browser.
Testium will prepend the value with `testium-driver-` and treat the resulting string as an npm module name.
The default value is "wd", so Testium will try to load `testium-driver-wd` (the ["promise-chain" interface](/api/wd/) is powered by [`wd`](http://admc.io/wd/))

A simple example:

```js
await browser.loadPage('/');

assert.strictEqual(await browser.getPageTitle(), 'Hello');
```

#### `launch`

Set this to `true` if you want Testium to handle starting the app during initialization and stopping it at the end.
This is not enabled by default.

#### `launchEnv`

The `NODE_ENV` to use when launching the app.
By default, Testium uses `NODE_ENV=test`.

#### `logDirectory`

Testium will capture the output of all processes it manages
and write it to separate files in this directory.
Directory paths are relative to the working directory.
The default is `./test/log`.

Testium will create the following files in this directory:

* `application.log`: Output of the application under test.
* `phantomjs.log`: Phantomjs logs.
  These include client-side JavaScript errors which otherwise might be hard to debug.
  This file won't be created when using any browser but PhantomJS.
* `proxy.log`: Output of the proxy that captures status codes and headers.
* `selenium.log`: This only exists if you use a browser other than PhantomJS. It fulfills a role similar to `phantomjs.log`.

#### `screenshotDirectory`

Automatically generated screenshots like the ones created on failing mocha tests will be placed in this directory.
Directory paths are relative to the working directory.
By default, Testium will use `test/log/failed_screenshots`.

### app

#### `app.command`

The command to use when launching the app.
This is ignored unless `launch` is enabled.
If no command is provided, Testium will parse `package.json` to simulate the behavior of `npm start`.

#### `app.port`

The port the app is listening on.
Setting the port to `0` tells Testium to select a random available port.
Testium will always pass the port as the environment variable `PORT` to your app.
Defaults to `0`.

The final port (after resolving a port of `0`) determines the base url when navigating to urls paths.
E.g. with a port of `8000`, `browser.loadPage('/foo')` will load `http://127.0.0.1:8000/foo`.

#### `app.timeout`

How long Testium should wait for the app to listen.
Expects a time in milliseconds.
By default, Testium will wait 30 seconds.

### mixins

Mixins allow you to extend `testium.browser` for all your tests.
They are commonly used for shorthands like `browser.setSessionCookies`.
This feature is best used in moderation.
Having too many mixed-in methods can make it hard to tell where a specific method is defined or what a certain test is actually doing.

Each setting is an array of module names that will be resolved relative to the working directory. E.g. `./test/mixins/session.js` will be resolved to `${cwd}/test/mixins/session.js` and `mixins/session` may be resolved to `${cwd}/node_modules/mixins/session.js`.

#### `wd`

Additional methods for `browser.*`.
See [section on custom methods](/api/wd/mixins.html) for how the files should be structured.

### mocha

As part of its before hook, `testium-mocha` will override some of mocha's options.
This allows you to define custom `slow` and `timeout` values just for integration tests without having to touch the global `--slow` and `--timeout` settings.

#### `mocha.slow`

Specify the "slow" test threshold.
Mocha uses this to highlight test-cases that are taking too long.
The default value is two seconds.
([Mocha Documentation](http://mochajs.org/#s-slow-lt-ms-gt))

#### `mocha.timeout`

Specifies the test-case timeout. The default value is 20 seconds.
([Mocha Documentation](http://mochajs.org/#t-timeout-lt-ms-gt))

### phantomjs

#### `phantomjs.command`

Command to start PhantomJS.
Change this if you don't have PhantomJS in your `$PATH`.
By default, this is just `phantomjs`.

#### `phantomjs.timeout`

How long to wait for phantomjs to listen in milliseconds.
By default, Testium will wait 6 seconds.

### proxy

Controls the behavior of Testium's HTTP proxy.
Testium will send all requests through a local proxy process.
This allows Testium to provide some additional features:

* Capture HTTP status codes and -headers.
* Inject additional headers to outgoing requests.
* Normalize response codes to 200.
  Some Webdriver implementations are sensible to server errors.
  Which means that server error pages won't show up properly.
  By always rewriting the status code to 200,
  Testium ensures that screenshots taken on error actually show the error pages.

#### `proxy.port`

The port to use for proxying.
If the port is set to 0, Testium will select a random available port.
The default is 4445.

#### `proxy.host`

If you wish to run your browser on a different host (see [`selenium.serverUrl`](config.html#seleniumserverurl)), this will default to the output of `hostname -f` in order to let the browser know how to reach this proxy running on localhost - if this is not correct, set this to a hostname that can be reached by the selenium browser.  If this host is not reachable directly by the selenium host (due to firewall, NAT, VPN, CI container, etc.), see `proxy.tunnel.host`.

#### `proxy.tunnel.host`
If you are using `driver = wd`, you may set this to open an SSH tunnel from a random port on a third party host thru to `proxy.port` on localhost by setting `proxy.tunnel.host` to a hostname which the Testium-running user has permission to ssh to with no password (via `$SSH_AUTH_SOCK` agent or unencrypted private keys in `~/.ssh`).  If this hostname is not the same as the hostname that selenium will need to contact (e.g. you have to ssh to `foo-internal.example.com`, but selenium will contact the same host as `foo-external.example.com`), set `proxy.host` to the latter - otherwise you can leave `proxy.host` unset and it will default to the value of `proxy.tunnel.host`.

You **must** set `GatewayPorts yes` in `/etc/ssh/sshd_config` on the tunnel host to allow it to accept connections to proxy.

#### `proxy.tunnel.username`
If the `$USER` environment variable is not an appropriate username for the ssh connection, you can override it here.

#### `proxy.tunnel.sshPort`
Defaults to 22

#### `proxy.tunnel.port`
If have already set up your own tunnel from a port on `proxy.tunnel.host` to `proxy.port` on this host, set `proxy.tunnel.port` to that port, and the SSH connection open will be skipped.

### repl

#### `repl.module`

Module to use for the Testium repl.
By default, this is node's built-in `repl` module.
If you want to use coffee-script in the repl, use `coffee-script/repl`.

### selenium

These settings will only be used when using a browser other than `phantomjs`.

#### `selenium.debug`

Set to false to disable verbose logging into `selenium.log`.
This is enabled by default.

#### `selenium.chromedriver`

Path to the `chromedriver` binary.
If not provided, Testium will download it automatically on demand.

#### `selenium.jar`

Path to the selenium standalone jar.
If not provided, Testium will download it automatically on demand.

#### `selenium.serverUrl`

Allows to provide an existing selenium server.
If the `serverUrl` is set, Testium will not try to launch its own instance of selenium.

#### `selenium.timeout`

How long to wait for selenium to listen for requests in milliseconds.
Defaults to 90 seconds.
