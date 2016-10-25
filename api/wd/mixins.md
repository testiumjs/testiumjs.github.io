---
title: Promise Chain Mixins
layout: sidebar
---

## Promise Chain Mixins

Testium's promise chain mixins are a thin wrapper around [`wd.addPromiseChainMethod`](https://github.com/admc/wd/#adding-custom-methods).

### Example: `browser.login()`

#### The Mixin File

Create a file `test/mixins/login.js` with the following content:

```js
exports.login = function login() {
  return this.navigateTo('/login')
    .type('.username', 'test')
    .type('.password', 'passw0rd')
    .clickOn('#login-btn');
};
```

Every export of the file will be turned into a method on `browser`.
When called, `this` will refer to the `browser`.

#### Add Config

Edit (or create) `.testiumrc` in the project directory:

```ini
; [other settings...]

[mixins]
; This path will be resolved relative to the project directory
wd[] = ./test/mixins/login.js
```

If you prefer JSON for `.testiumrc`:

```json
{
  "mixins": {
    "wd": ["./test/mixins/login.js"]
  }
}
```

#### Using the Helper

In test files you can now start using `browser.login`:

```js
// ...
before(() => browser.login());

before(() => browser.navigateTo('/account'));

it("shows the test user's account", () => { /* ... */ });
```
