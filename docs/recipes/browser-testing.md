# Setting up AVA for browser testing

Translations: [Español](https://github.com/avajs/ava-docs/blob/master/es_ES/docs/recipes/browser-testing.md), [Français](https://github.com/avajs/ava-docs/blob/master/fr_FR/docs/recipes/browser-testing.md), [Italiano](https://github.com/avajs/ava-docs/blob/master/it_IT/docs/recipes/browser-testing.md), [Русский](https://github.com/avajs/ava-docs/blob/master/ru_RU/docs/recipes/browser-testing.md), [简体中文](https://github.com/avajs/ava-docs/blob/master/zh_CN/docs/recipes/browser-testing.md)

AVA does not support running tests in browsers [yet](https://github.com/avajs/ava/issues/24). Some libraries require browser specific globals (`window`, `document`, `navigator`, etc).
An example of this is React, at least if you want to use ReactDOM.render and simulate events with ReactTestUtils.

This recipe works for any library that needs a mocked browser environment.

## Install browser-env

Install [browser-env](https://github.com/lukechilds/browser-env).

> Simulates a global browser environment using jsdom.

```
$ npm install --save-dev browser-env
```

## Setup browser-env

Create a helper file and place it in the `test/helpers` folder. This ensures AVA does not treat it as a test.

`test/helpers/setup-browser-env.js`:

```js
import browserEnv from 'browser-env';
browserEnv();
```

By default, `browser-env` will add all global browser variables to the Node.js global scope, creating a full browser environment. This should have good compatibility with most front-end libraries, however, it's generally not a good idea to create lots of global variables if you don't need to. If you know exactly which browser globals you need, you can pass an array of them.

```js
import browserEnv from 'browser-env';
browserEnv(['window', 'document', 'navigator']);
```

## Configure tests to use browser-env

Configure AVA to `require` the helper before every test file.

`package.json`:

```json
{
  "ava": {
    "require": [
      "./test/helpers/setup-browser-env.js"
    ]
  }
}
```

## Enjoy!

Write your tests and enjoy a mocked browser environment.

`test/my.dom.test.js`:

```js
import test from 'ava';

test('Insert to DOM', t => {
	const div = document.createElement('div');
	document.body.appendChild(div);

	t.is(document.querySelector('div'), div);
});
```

`test/my.react.test.js`:

```js
import test from 'ava';
import React from 'react';
import {render} from 'react-dom';
import {Simulate} from 'react-addons-test-utils';
import sinon from 'sinon';
import CustomInput from './components/custom-input.jsx';

test('Input calls onBlur', t => {
	const onUserBlur = sinon.spy();
	const input = render(
		React.createElement(CustomInput, onUserBlur),
		div
	);

	Simulate.blur(input);

	t.true(onUserBlur.calledOnce);
});
```
