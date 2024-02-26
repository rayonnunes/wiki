
# ⚒️ Requirements

- Node/npm

# 👣 Resources:

- [Testing JS on Stackblitz](https://stackblitz.com/edit/stackblitz-starters-4etrrx)

# 🛫 Intro

Here’s a simple implementation of a test suite:

```jsx
async function test(title, callback) {
  try {
    await callback()
		// This will print a nice green success message
    console.log(`\x1b[92m✓ ${title}\x1b[0m`)
  } catch (error) {
		// This will print a showy red error message
    console.error(`\x1b[91m✕ ${title}\x1b[0m`)
    console.error(error)
  }
}

function expect(actual) {
  return {
    toBe(expected) {
      if (actual !== expected) {
        throw new Error(`${actual} is not equal to ${expected}`);
      }
    },

    notToBe(expected) {
      if (actual === expected) {
        throw new Error(`${actual} is equal to ${expected}`);
      }
    }
  }
}
```

```jsx
const sum = (a, b) => a + b
const subtract = (a, b) => a - b
```

```jsx
test('sum adds numbers', () => {
  const result = sum(3, 7)
  const expected = 10

  expect(result).toBe(expected)
})

test('subtract subtracts numbers', () => {
  const result = subtract(7, 3)
  const expected = 4

  expect(result).toBe(expected)
})
```

- ⌨️ On terminal:
    ```bash
    node --require ./setup-globals.js ./1-intro/math.test.js
    ```

# 🎭 Mocking

Mocking consists of simulating an existing module or function by overriding it to detach external dependencies and define controlled results.

```jsx
// utils.js
function rollDice(numSides) {
  return Math.floor(Math.random() * numSides) + 1;
}

function makeAPIRequest() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) {
        resolve('Hit saved!');
      } else {
        reject(new Error('Request failed'));
      }
    }, 500);
  });
}

module.exports = {
  rollDice,
  makeAPIRequest,
};
```

```jsx
// attack.js
const utils = require('./utils');

function attack(player, enemy) {
  const diceRollResult = utils.rollDice(6);

  if (diceRollResult > 3) {
    try {
      utils.makeAPIRequest();
      return `${player} hit ${enemy}`;
    } catch (e) {
      return 'Request failed';
    }
  }

  return 'Miss';
}

module.exports = attac
```

We can simply override those functions in our test file:

```tsx
// attack.mokey-patch-test.js
const assert = require('assert');
const utils = require('./utils');
const attack = require('./attack');

utils.rollDice = (numSides) => 4;
utils.makeAPIRequest = () => new Promise((resolve) => resolve('Hit saved!'));

const player = 'Warrior';
const enemy = 'Goblin';

const attackResult = attack(player, enemy);

assert.strictEqual(
  attackResult,
  `${player} hit ${enemy} 1`,
  `\x1b[91m✕ ${player} didn't hit ${enemy}\x1b[0m`
);

console.log(`\x1b[92m✓ ${player} hit ${enemy}\x1b[0m`);

```

- ⌨️ On terminal:
    ```bash
    node ./2-mocking/attack.mokey-patch-test.js
    ```


## 🚨 Caveats

### 🔍 Implementation changes will not be tracked

Suppose we add a new rule limiting the maximum number of sides of a dice to 6.

```jsx
function rollDice(numSides) {
	const maxNumSides = numSides > 6 ? 6 : numSides
  return Math.floor(Math.random() * maxNumSides) + 1;
}
```

This function should never return 11, but the mock keeps returning an inconsistent result.

### 🧹 Cleanup

It is a good practice to always revert your changes to not impact other tests in the same scope

```jsx
// attack.mokey-patch-test.js
const assert = require('assert');
const utils = require('./utils');
const attack = require('./attack');

const originalRollDice = utils.rollDice
const originalMakeAPIRequest = utils.makeAPIRequest
utils.rollDice = (numSides) => 11;
utils.makeAPIRequest = () => new Promise((resolve) => resolve('Hit saved!'));

const player = 'Warrior';
const enemy = 'Goblin';

const attackResult = attack(player, enemy);

assert.strictEqual(
  attackResult,
  `${player} hit ${enemy} 1`,
  `\x1b[91m✕ ${player} didn't hit ${enemy}\x1b[0m`
);

utils.rollDice = originalRollDice
utils.makeAPIRequest = originalMakeAPIRequest
```

### 📋 Destructuring assignment

the following syntax:

```jsx
const utils = require('./utils')
```

differs from the destructuring assignment:

```jsx
const { rollDice, makeAPIRequest } = require('./utils')
```

destructuring creates a new copy of your functions instead of pointing to module reference. So this simple refactor can break your tests.

## 🃏 Using Jest

Without having to monkey-patch all of tests. Jest provides a broad set of tools available to analyze and check your files:

```jsx
// attack.test.js
const utils = require('./utils');
const attack = require('./attack');

test('attack hits the enemy', () => {
  const originalRollDice = utils.rollDice
  const originalMakeAPIRequest = utils.makeAPIRequest

  utils.rollDice = (numSides) => 4;
  utils.makeAPIRequest = () => new Promise((resolve) => resolve('Hit saved!'))

  const player = 'Warrior';
  const enemy = 'Goblin';

  const attackResult = attack(player, enemy);
  expect(attackResult).toBe(`${player} hit ${enemy}`);

  expect(utils.rollDice).toHaveBeenCalledTimes(1);
  expect(utils.makeAPIRequest).toHaveBeenCalledTimes(1);

  expect(utils.rollDice).toHaveBeenCalledWith(6);
  expect(utils.rollDice.mock.calls).toEqual([[6]]);

  utils.rollDice = originalRollDice;
  utils.makeAPIRequest = originalMakeAPIRequest;
});

```

### 📦 Install

```bash
npm i jest -D
```

- ⌨️ On terminal:
    ```bash
    npx jest
    ```

## ⚙️ The `fn` function

One way to capture function calls can be by creating mock functions, in jest `fn` function allows you to create a simulated and observable function and adds a special property `.mock` which is where data about how the function has been called and what the function returned is kept. The `.mock` property also tracks the value of this for each call:

```jsx
// attack.test.js
test('attack hits the enemy', () => {
  const originalRollDice = utils.rollDice;
  const originalMakeAPIRequest = utils.makeAPIRequest;

  utils.rollDice = jest.fn((numSides) => 4);
  utils.makeAPIRequest = jest.fn(
    () => new Promise((resolve) => resolve('Hit saved!'))
  );

  console.log(utils.makeAPIRequest);

  const player = 'Warrior';
  const enemy = 'Goblin';

  const attackResult = attack(player, enemy);
  expect(attackResult).toBe(`${player} hit ${enemy}`);

  expect(utils.rollDice).toHaveBeenCalledTimes(1);
  expect(utils.makeAPIRequest).toHaveBeenCalledTimes(1);

  expect(utils.rollDice).toHaveBeenCalledWith(6);
  expect(utils.rollDice.mock.calls).toEqual([[6]]);

  utils.rollDice = originalRollDice;
  utils.makeAPIRequest = originalMakeAPIRequest;
});
```

### 🔬 A closer look at `fn` function

```jsx
// mock-fn.js
function fn(impl = () => {}) {
  const mockFn = (...args) => {
    mockFn.mock.calls.push(args);
    return impl(...args);
  };

  mockFn.mock = {
    calls: [],
  };

  mockFn.mockImplementation = (newImpl) => (impl = newImpl);

  return mockFn;
}
```

```jsx
// mock-fn.mokey-patch-test.js
const assert = require('assert');
const utils = require('./utils');
const attack = require('./attack');
const fn = require('./mock-fn');

utils.rollDice = fn((numSides) => 4);
utils.makeAPIRequest = fn(
  () => new Promise((resolve) => resolve('Hit saved!'))
);

const player = 'Warrior';
const enemy = 'Goblin';

const attackResult = attack(player, enemy);

assert.strictEqual(
  attackResult,
  `${player} hit ${enemy}`,
  `\x1b[91m✕ ${player} didn't hit ${enemy}\x1b[0m`
);
console.log(`\x1b[92m✓ ${player} hit ${enemy}\x1b[0m`);

assert.deepStrictEqual(utils.rollDice.mock.calls, [[6]]);

console.log(`\x1b[92m✓ dice rolled 6\x1b[0m`);
```

- ⌨️ On terminal:
    
    ```bash
    node 2-mocking/mock-fn.mokey-patch-test.js
    ```
    

## 👁️‍🗨️ The `spyOn` function

Creates a mock function similar to jest.fn but also can track calls of a property of the object. Can return a Jest mock function.

```jsx
// attack.test.js
test('attack hits the enemy using spyOn function', () => {
  jest.spyOn(utils, 'rollDice');
  jest.spyOn(utils, 'makeAPIRequest');

  utils.rollDice.mockImplementation((numSides) => 4);
  utils.makeAPIRequest.mockImplementation(
    () => new Promise((resolve) => resolve('Hit saved!'))
  );

  const player = 'Warrior';
  const enemy = 'Goblin';

  const attackResult = attack(player, enemy);
  expect(attackResult).toBe(`${player} hit ${enemy}`);

  expect(mockedRollDice).toHaveBeenCalledTimes(1);
  expect(mockedMakeAPIRequest).toHaveBeenCalledTimes(1);

  expect(utils.rollDice).toHaveBeenCalledWith(6);
  expect(utils.rollDice.mock.calls).toEqual([[6]]);

  utils.rollDice.mockRestore();
  utils.makeAPIRequest.mockRestore();
});
```

### 🔬 A closer look at `spyOn` function

```jsx
// mock-spyOn.js
function spyOn(obj, prop) {
  const originalValue = obj[prop];

  obj[prop] = fn();

  obj[prop].mockRestore = () => (obj[prop] = originalValue);
}
```

```jsx
// mock-spyOn.mokey-patch-test.js
const assert = require('assert');
const attack = require('./attack');
const utils = require('./utils');
const spyOn = require('./mock-spyOn');

spyOn(utils, 'rollDice');
spyOn(utils, 'makeAPIRequest');

utils.rollDice.mockImplementation((numSides) => 4);
utils.makeAPIRequest.mockImplementation(
  () => new Promise((resolve) => resolve('Hit saved!'))
);

const player = 'Warrior';
const enemy = 'Goblin';

const attackResult = attack(player, enemy);

assert.strictEqual(
  attackResult,
  `${player} hit ${enemy}`,
  `\x1b[91m✕ ${player} didn't hit ${enemy}\x1b[0m`
);
console.log(`\x1b[92m✓ ${player} hit ${enemy}\x1b[0m`);

assert.deepStrictEqual(utils.rollDice.mock.calls, [[6]]);

console.log(`\x1b[92m✓ dice rolled 6\x1b[0m`);

utils.rollDice.mockRestore();
utils.makeAPIRequest.mockRestore();
```

- ⌨️ On terminal:
    
    ```bash
    node 2-mocking/mock-spyOn.mokey-patch-test.js
    ```
    

## 🧩 Using `mock` function to mock modules

As you move from **CommonJS** to **ESModules** (especially after ES6) replacing `require()` and `module.exports` with `import` and `export` and adding ‘use strict’ directive, you will no longer be able to use monkey-patching to just override the global scope of your modules, instead, jest provides an API to handle entire modules:

```jsx
// attack-mock.test.js
const utils = require('./utils');
const attack = require('./attack');

jest.mock('./utils', () => ({
  rollDice: jest.fn((numSides) => 4),
  makeAPIRequest: jest.fn(
    () => new Promise((resolve) => resolve('Hit saved!'))
  ),
}));

test('attack hits the enemy using spyOn function', () => {
  const player = 'Warrior';
  const enemy = 'Goblin';

  const attackResult = attack(player, enemy);
  expect(attackResult).toBe(`${player} hit ${enemy}`);

  expect(utils.rollDice).toHaveBeenCalledTimes(1);
  expect(utils.makeAPIRequest).toHaveBeenCalledTimes(1);

  expect(utils.rollDice).toHaveBeenCalledWith(6);
  expect(utils.rollDice.mock.calls).toEqual([[6]]);

  utils.rollDice.mockReset();
  utils.makeAPIRequest.mockReset();
});
```

> [!note]
> Test files will be transformed before run to move  jest.mock() to the top-level of the fileto get control of modules system

### 📨 Sharing mocked modules

simply create a folder `__mocks__` on the level that you want to include your mocked modules and then create a file with the same name as the module you want to mock

## 💡 Further tips:

Therefore, the idea of mocking can be summed up as:

- jest.fn((…args) ⇒ impl)
- jest.spyOn(obj, prop)
    - remember to clean up with mockRestore()
- mockImplementation 🚨
- jest.mock(’./path-to-module’, implFactory)
    - remember to cleanup with mock.reset()

To avoid headaches by forgetting to clear mocks, remember this:

- to clear mocked functions (jest.fn or jest.spyOn)

```jsx
// replace afterEach hook based on your test needs
afterEach(() => {
	jest.restoreAllMocks()
})
```

```jsx
// replace afterEach hook based on your test needs
afterEach(() => {
	jest.clearAllMocks()
})
```

# 🔦Static Analysis Testing

## Eslint

Eslint is a static analysis tool that can find errors and enforce code format.

### 📦 Install

```bash
npm i eslint -D
```

Also, you need to have a configuration file (it could be named as `.eslintrc` followed by `.js` / `.yml` / `.json` extension, whatever you prefer), you can set the `parserOption` property for customization and `rules` to modify what exactly you want to check or not in your code (you can learn more about the the whole list of rules in the [rules reference page](https://eslint.org/docs/latest/rules), and how rules work in [configure rules page](https://eslint.org/docs/latest/use/configure/rules)), here’s a short example:

```json
// .eslintrc.json
{
	"parserOptions": {
		// JavaScript version that you want to support
		"ecmaVersion": 2020,
		// "script" for require/module.exports and "module" for import/export standard
		"sourceType": "module",
		// specific tweaks of ECMAscript standard
		"ecmaFeatures": {
			// allow JSX syntax, useful for frameworks like react.
			"jsx": true
		}
	},
	// rules can be set as "off", "warn", "error", some of them can contain special configurations inside it
	"rules": {
		// "use strict" statement should never be contained in files otherwise raises an error.
		"strict": [
			"error",
			"never",
		]
		// Enforce comparing typeof expressions against valid strings
		"valid-typeof": "error",
		// raises a warning when you use console.[log | warn | error]()
		"no-console": "warn",
	}
	// tells eslint which environment/global variables are available for your files
	"env": {
		// includes all browser global variables (like console)
		"browser": true,
	}
}
```

>[!tip]
>You can skip files or directories on eslint by adding them on .eslintignore file or passing `--ignore path` on cli.


A handy way to startup eslint (as recommended by the [docs](https://eslint.org/)) with some pre-defined presets is to run:

```bash
npm init @eslint/config
```

> [!info]
> You will be asked about some questions regarding of your project to create the config file

### ESLint extension for VSCode

Instead of running

```bash
npx eslint . --fix
```

every time to fix your files you can install [ESLint extension for VSCode](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) to have real-time aid about the errors and warnings in your code. In VSCode you can press `Ctrl Shift P` and run **ESLint: Fix all auto-fixable Problems** or select the error piece of your code, press `Ctrl .` and select `Fix this <rule_name> problem`. 

## Prettier

[Prettier](https://prettier.io/) is an opinionated code formatter that ensures that all the code conforms to a consistent style

### 📦 Install

```bash
npm install -D --save-exact prettier
```

to format your files:

```bash
npx prettier path_to_file.js --write
```

>[!info]
>If you miss `--write` flag, prettier will only output the formatted content, without overriding the original file

>[!tip]
>Furthermore, you can also pass the flag `--list-different` to validate your files


to format only specific patterns to be formatted for example:

```bash
npx prettier --write "**/*.+(js|json)"
```
>[!tip]
>Just like ESLint is possible to ignore files in a similar way, either by creating a .prettierignore file or passing `--ignore path` on cli


### ⚙️ Configuration

It is possible to set up your code-format preferences by creating a `.prettierrc` file and adding the options you want there.

```json
{
  "arrowParens": "avoid",
  "bracketSameLine": false,
  "bracketSpacing": false,
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": false,
  "quoteProps": "consistent",
  "trailingComma": "all",
  "singleAttributePerLine": true,
  "htmlWhitespaceSensitivity": "css",
  "vueIndentScriptAndStyle": false,
  "proseWrap": "always",
  "insertPragma": false,
  "printWidth": 80,
  "requirePragma": false,
  "tabWidth": 2,
  "useTabs": false,
  "embeddedLanguageFormatting": "auto"
}
```

Also, prettier has a [playground](https://prettier.io/playground/) where it’s possible to visualize and play with the option that it gives and export a JSON containing the options you’ve chosen.

### Prettier extension for VSCode

There is an extension of [Prettier for VSCode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) that allows you to format your code directly from your editor, after installing it you need to add a few settings in settings.json on your editor to take full advantage of the tool, you can do this from two ways:

- Press `Ctrl Shif P` and search for **Preferences: Open Settings (UI):**
    - Then look for **Editor: Default Formatter** and select **“Prettier - Code formatter”** option.
    - Next look for **Editor: Format On Save** option and enable it;
- Press `Ctrl Shif P` and search for **Preferences: Open Default Settings (JSON):**
    - Add or modify the option `"editor.defaultFormatter": "esbenp.prettier-vscode"`
    - Add or modify the option `"editor.formatOnSave": true`

### Conflicts between prettier and eslint

Some rules in prettier and eslint can overlap, for example, suppose you have in your `.eslinrc` arule for:

```json
"semi": ["error", "always"]
```

In your .prettierrc file, you have:

```json
"semi": false
```

To avoid ambiguity between rules it is possible to configure your eslint to disable any rules applied by prettier by installing [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier) plugin as a dev dependency, with:

```bash
npm i eslint-config-prettier -D
```

then you can go in your `.eslintrc` file and add it on your `extends` config

```json
"extends": ["eslint-config-prettier", ...your-additional-configs]
```

## TypeScript

Typescript is an extension of JavaScript making it a strong-typed programming language giving you a better [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense) for your editor.

### 📦 Install

```bash
npm i typescript -D
```

Changing your files from .js to .ts already makes VSCode do type-checking in your file.

The typescript package provides a CLI compiler called `tsc` to verify your files (and maybe transpile them to .js) but in order to do that you must configure a `tsconfig.json` file, the easiest way is to run:

```bash
npx tsc --init
```

Some options can be set inside `compilerOptions` object, for example:

```json
// tsconfig.json
{
	"compilerOptions": {
		// does not create output files, only type-check them
		// useful when you have another transpiler like babel
		"noEmit": true,
    // source directory of your project
		"baseUrl": "./src"
	}
}
```
>[!warning]
>Some transpilers like babel doesn’t support typescript files straightforward, to add support to .ts files, install [@babel/preset-typescript](https://babeljs.io/docs/babel-preset-typescript) as a dev dependency

### Linting your typescript files

TypeScript already covers many of the error tracking of your file, but to integrate your eslint to support typescript files you can install the packages `@typescript-eslint/eslint-plugin` and `@typescript-eslint/parser` as dev dependencies

```bash
npm i @typescript-eslint/eslint-plugin @typescript-eslint/parser -D
```

and then, in your `.eslintrc` file, you should add:

```json
{
  ...
  "overrides": [
    {
      "files": "**/*.+(ts|tsx)",
			"parser": "@typescript-eslint/parser",
			"parserOptions": {
        "project": "./tsconfig.json"
      },
			"plugins": ["@typescript-eslint/eslint-plugin"],
      "extends": [
        "plugin:@typescript-eslint/eslint-recommended",
        "plugin:@typescript-eslint/recommended",
				"eslint-config-prettier/@typescript-eslint"
      ]
    }
  ]
}
```
This helps remove duplicated error messages in your code (one from ts, other from eslint)

## Husky git hooks

Husky is a tool that allows you to add actions before committing or pushing your code in git. It does so by manipulating your .git/hooks directory

### 📦 Install

```bash
npm i husky -D
```

### ⚙️ Configuration

Basically like other tools, you will need to create a `.huskyrc` file to define your hooks associated with a script in `package.json`, for example:

```json
{
  "hooks": {
    "pre-commit": "npm run my_custom_script"
  }
}
```

## lint-staged

Sometimes your code can be manipulated by people that doesn’t have all the VSCode extensions or even from other editors and generate errors when it tries to commit your code, to avoid that you can use `lint-staged` package to run your lint/format scripts before doing your commit

### 📦 Install

```bash
npm i lint-staged -D
```

### ⚙️ Configuration

As you may guess, you will need to have a configuration file `.lintstagedrc`, with your preferences:

```json
{
  "*.+(js|ts|tsx)": [
    "npm run my_custom_script",
		"git add"
  ]
}
```

there you can run any script you want on files ending with `.js`, `.ts`, and `.tsx` and to execute it you can run:

```bash
npx lint-staged
```

Or you can integrate with Husky to validate/format your code for example on commit:

```json
// .huskyrc
{
  "hooks": {
    "pre-commit": "lint-staged"
  }
}
```
>[!info]
>Added “git add” command on lint-staged config to add lint-staged modified files to the same commit

## 🏃 npm-run-all

npm-run all is a tool that increases your performance by running multiple npm scripts in parallel

### 📦 Install

```json
npm i npm-run-all -D
```

### ⚙️ Configuration

In your `package.json` file you can create a script such as:

```json
{
	...
	"scripts": {
		...
		"my_script": "npm-run-all --parallel command_1 test_script build_script"
	}
```
This will avoid adding `npm run` before each command

# Testing library using DOM (framework-agnostic)

Testing library is a family of utilities that provides DOM manipulation for any kind of JS framework, essentially it provides some utilities such as:

```jsx
/* 
 * getQueriesForElement provides several methods to retrieve your elements
 * such as getByText, queryByText, getByTestId and so on
 */
import {getQueriesForElement} from '@testing-library/dom'
/* 
 * userEvent is capable to provide user interactions with your DOM
 * such as click, input, blur, etc.
 */
import userEvent from '@testing-library/user-event'
```

this allows you to essentially manipulate your dom independent of the framework, leaving to them only the job of transpile your component to static HTML/CSS/JS

## React

```jsx
import '@testing-library/jest-dom/extend-expect'
import * as React from 'react'
import ReactDOM from 'react-dom'
import {getQueriesForElement} from '@testing-library/dom'
import userEvent from '@testing-library/user-event'

function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  return (
    <div>
      <button onClick={increment}>{count}</button>
    </div>
  )
}

function render(ui) {
  const container = document.createElement('div')
  ReactDOM.render(ui, container)
  document.body.appendChild(container)
  return {
    ...getQueriesForElement(container),
    container,
    cleanup() {
      ReactDOM.unmountComponentAtNode(container)
      document.body.removeChild(container)
    },
  }
}

test('renders a counter', () => {
  const {getByText, cleanup} = render(<Counter />)
  const counter = getByText('0')
  userEvent.click(counter)
  expect(counter).toHaveTextContent('1')

  userEvent.click(counter)
  expect(counter).toHaveTextContent('2')
  cleanup()
})
```

## Vue

```jsx
import Vue from 'vue/dist/vue'
import '@testing-library/jest-dom/extend-expect'
import {userEventAsync} from './user-event-async'
import {getQueriesForElement} from '@testing-library/dom'

Vue.config.productionTip = false
Vue.config.devtools = false

const Counter = {
  template: `
    <div>
      <button @click='increment'>
        {{count}}
      </button>
    </div>
  `,
  data: () => ({count: 0}),
  methods: {
    increment() {
      this.count++
    },
  },
}

function render(Component) {
  const vm = new Vue(Component).$mount()
  return {
    container: vm.$el,
    ...getQueriesForElement(vm.$el),
  }
}

test('counter increments', async () => {
  const {getByText} = render(Counter)
  const counter = getByText('0')
  await userEventAsync.click(counter)
  expect(counter).toHaveTextContent('1')

  await userEventAsync.click(counter)
  expect(counter).toHaveTextContent('2')
})
```

## Svelte

```jsx
import '@testing-library/jest-dom/extend-expect'
import {userEventAsync} from './user-event-async'
import {getQueriesForElement} from '@testing-library/dom'
import Counter from './counter.svelte'

function render(Component) {
  const container = document.createElement('div')

  new Component({target: container})

  return {
    ...getQueriesForElement(container),
    container,
  }
}

test('counter increments', async () => {
  const {getByText} = render(Counter)
  const counter = getByText('0')
  await userEventAsync.click(counter)
  expect(counter).toHaveTextContent('1')

  await userEventAsync.click(counter)
  expect(counter).toHaveTextContent('2')
})
```

## VanillaJS

```jsx
import '@testing-library/jest-dom/extend-expect'
import {getQueriesForElement} from '@testing-library/dom'
import userEvent from '@testing-library/user-event'

function countify(el) {
  el.innerHTML = `
    <div>
      <button>0</button>
    </div>
  `
  const button = el.querySelector('button')
  button._count = 0
  button.addEventListener('click', () => {
    button._count++
    button.textContent = button._count
  })
}

// tests:
test('counter increments', () => {
  const div = document.createElement('div')
  countify(div)
  const {getByText} = getQueriesForElement(div)
  const counter = getByText('0')
  userEvent.click(counter)
  expect(counter).toHaveTextContent('1')

  userEvent.click(counter)
  expect(counter).toHaveTextContent('2')
})
```

# Configure Jest for testing JS Applications

## ⛰️ Environments

Jest can simulate many environments for your applications, by default it relies on node environment, but you can also have a jsdom that allows you to test frontend applications by providing global variables like `window`, `document`, etc.

As of Jest 28 "jest-environment-jsdom" is no longer shipped by default, make sure to install it separately with:

```bash
npm i jest-environment-jsdom -D
```

## 🏗️ Mocking CommonJS Modules

If you are struggling with files that were not JS, such as CSS, you can add to your `jest.config.js` an option `moduleNameMapper` to redirect your modules when it matches some pattern. for example, to mock CSS files you can create at your root folder a `./test/style-mock.js` file just exporting an empty object:

```jsx
module.exports = {};
```

and on your `jest.config.js`

```json
module.exports = {
    ...
    moduleNameMapper: {
        '\\.css$': require.resolve("./test/style-mock.js"),
    }
}
```

Alternatively, you can use [identity-obj-proxy](https://www.npmjs.com/package/identity-obj-proxy) to transform your CSS paths into strings, avoiding losing some information in your tests. For example a React component like below:

```jsx
function AutoScalingText({children}) {
  const nodeRef = React.useRef()
  const scale = getScale(nodeRef.current)
  return (
    <div
      className={styles.autoScalingText}
      style={{transform: `scale(${scale},${scale})`}}
      ref={nodeRef}
      data-testid="total"
    >
      {children}
    </div>
  )
}
```

using the previous solution of resolving css modules with a mocked object  will make the debug method from react testing library interpret the rendering object with className missing:

```jsx
test('renders', () => {
    const { debug } = render(<AutoScalingText />)
    debug()
})

// Output:
// <body>
//      <div>
//          <div
//              data-testid="total"
//              style="transform: scale(1,1);"
//          />
//      </div>
// </body>
```

but after installing `identity-obj-proxy` as dev dependency and then changing your `jest.config.js` to:

```jsx
module.exports = {
    ...
    moduleNameMapper: {
        '\\.css$': 'identity-obj-proxy',
    }
}
```

your debug method from react testing library will return:

```html
<body>
    <div>
        <div
        class="autoScalingText"
        data-testid="total"
        style="transform: scale(1,1);"
        />
    </div>
</body>
```

## 📸 Snapshot testing

Snapshot tests are a very useful tool whenever you want to make sure your UI does not change unexpectedly.

A typical snapshot test case renders a UI component, takes a snapshot, and then compares it to a reference snapshot file stored alongside the test.

The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

Every time you create a new snapshot testing, jest serializes into a new file within a `__snapshots__` folder, the file will contain the same name as your test file followed by `.snap` extension. Consider this example for [t](https://github.com/jestjs/jest/blob/main/examples/snapshot/Link.js)he AutoScaling component:

```jsx
import { render } from '@testing-library/react'

import AutoScalingText from '../auto-scaling-text'

it('renders correctly', () => {
  const { baseElement } = render(<AutoScalingText />)
  expect(baseElement).toMatchSnapshot();
});
```

the `__snapshot__/auto-scaling-text.js.snap` will be serialized as:

```jsx
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`renders 1`] = `
<body>
  <div>
    <div
      class="autoScalingText"
      data-testid="total"
      style="transform: scale(1,1);"
    />
  </div>
</body>
`;
```
>[!info]
>You can update your snapshots by passing `-u` flag to your jest command

Another approach to handle snapshots is using `toMatchInlineSnapshot` instead of `toMatchSnapshot` this way jest will not create a new file but it will write the serialized output directly as the argument of your `toMatchInlineSnapshot` method:

```jsx
it('renders correctly', () => {
  const { baseElement } = render(<AutoScalingText />)
  expect(baseElement).toMatchInlineSnapshot();
});
```

After the next test run, the same file will be overwritten as:

```jsx
test('renders', () => {
  const {baseElement} = render(<AutoScalingText />)
  expect(baseElement).toMatchInlineSnapshot(`
    <body>
      <div>
        <div
          class="autoScalingText"
          data-testid="total"
          style="transform: scale(1,1);"
        />
      </div>
    </body>
  `)
})
```
>[!tip]
>Pro tip: install and configure prettier before using `toMatchInlineSnapshot` to avoid creating a messy test file


## 🎨 Testing style from UI

It’s a common standard to use a css-in-js library, such as [styled-components](https://styled-components.com/). testing them can be a little bit tricky because class names are harder to read since they are labeled with a random string.

Suppose the following styled component:

```jsx
const Container = styled.div`
  position: relative;
  color: white;
  background: #1c191c;
  line-height: 130px;
  font-size: 6em;
  flex: 1;
`;
```

When jest processes the file, it will output as a result, an encoded value for the class name representing the applied style, so as long as your CSS remains the same, the snapshot will keep matching:

```jsx
test('CalculatorDisplay renders', () => {
  const {baseElement} = render(<CalculatorDisplay value="0" />)
  expect(baseElement).toMatchInlineSnapshot(`
    <body>
      <div>
        <div
          class="sc-bdfCDU cJFxBv"
        >
          <div
            class="autoScalingText"
            data-testid="total"
            style="transform: scale(1,1);"
          >
            0
          </div>
        </div>
      </div>
    </body>
  `)
})
```

But, for example, if you change your `font-size` to `8rem`, jest will generate another snapshot that will break your tests, which is fine but the only debug jest will present will be another encoded class name

![1-jest-output-class-name.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/1-jest-output-class-name.png)
>Log output from jest showing the difference between the snapshot (**class="sc-bdfCDU cJFxBv”)** and received (**class="sc-bdfCDU kYXyTR”)**

As you can see this can make it hard to keep track of what was changed, a better way to create those snapshots is using [jest-styled-components](https://github.com/styled-components/jest-styled-components) that will register your CSS at the top of your component and serialize it:

```jsx
import 'jest-styled-components'

test('CalculatorDisplay renders', () => {
  const {baseElement} = render(<CalculatorDisplay value="0" />)
  expect(baseElement).toMatchInlineSnapshot(`
    .c0 {
      position: relative;
      color: white;
      background: #1c191c;
      line-height: 130px;
      font-size: 6em;
      flex: 1;
    }

    <body>
      <div>
        <div
          class="c0"
        >
          <div
            class="autoScalingText"
            data-testid="total"
            style="transform: scale(1,1);"
          >
            0
          </div>
        </div>
      </div>
    </body>
  `)
})
```

To install `jest-styled-components`:

```jsx
npm i jest-styled-components -D
```

Now, when a change occurs, jest will be able to give much better information about what was changed

![2-jest-styled-components-output.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/2-jest-styled-components-output.png)
>Log output from jest using jest-styled-components, now is possible to have the exact information about what was changed

To make the serialization easier, instead of `import 'jest-styled-components'` in every file, you can add to your `jest.config.js`
```js
module.exports = {
	snapshotSerializers: ['jest-styled-components'],
	...
}
```

## ⚒️ Supporting a test utility file with moduleDirectories

Usually in React, we rely on providers to grant access to some variables within our component tree (styled-components, react-router, redux…). But when rendering a component in a test file we do not want to wrap them with providers every time. So, taking as an example the main App component wrapping the Calculator with [ThemeProvider](https://styled-components.com/docs/advanced#theming).

```jsx
import { ThemeProvider } from 'styled-components'

function App() {
  return (
    <div>
      <ThemeProvider theme={{
        displayTextColor: '#fff',
        displayBackgroundColor: '#000',
      }}>
        <Calculator />
      </ThemeProvider>
    </div>
  )
}
```

In our [[previous calculator display using styled-components#Testing style from UI]] inside `<Calculator />` we can refactor the `color` and `background` to be obtained directly from the theme provider

```jsx
const Container = styled.div`
  position: relative;
  line-height: 130px;
  font-size: 6em;
  flex: 1;
  color: ${({theme}) => theme.displayTextColor};
  background: ${({theme}) => theme.displayBackgroundColor};
`;
```

You will notice that the snapshot of `__tests__/calculator-display.js` will not include color and background. This occurs because styled-components automatically remove undefined properties from the template string and they are undefined because we are not passing our provider in the test file.

```jsx
`.c0 {
    position: relative;
    line-height: 130px;
    font-size: 6em;
    flex: 1;
}`
```

### ♻️ Reusing render with providers
So to Incorporate our desired providers we can create a helper file to encapsulate the `render` method from `@testing-library/react`, let's create a new file called `render-utils.js` inside our `test` folder at the root of the project.

```jsx
import React from 'react'
import PropTypes from 'prop-types'
import {render as rtlRender} from '@testing-library/react'
import {ThemeProvider} from 'styled-components'

import themes from '../src/themes'

function render(ui, options) {
  return rtlRender(ui, {wrapper: Wrapper, ...options})
}

function Wrapper({ children }) {    
  return <ThemeProvider theme={themes.dark}>{children}</ThemeProvider>
}

Wrapper.propTypes = {
  children: PropTypes.node,
}

export * from '@testing-library/react'

export { render }
```

Now when you run the same test but with `render` from our `render-utils.js` you will notice that the snapshot test will fail because of the properties `color` and `background` are now present

![3-jest-output-custom-render.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/3-jest-output-custom-render.png)
>properties defined by theme are now present on test render

To expose some customization to your wrapper, you can inject additional properties in the options
```jsx
import React from 'react'
import PropTypes from 'prop-types'
import {render as rtlRender} from '@testing-library/react'
import {ThemeProvider} from 'styled-components'

import themes from '../src/themes'

// It is possible to pass additional options in render to customize your providers during tests
function render(ui, {...options}) {
  
  function Wrapper({ children }) {
	// here we have theme that is not a default option from testing-library
	const {theme = 'dark'} = options

    return <ThemeProvider theme={themes[theme]}>{children}</ThemeProvider>
  }

  Wrapper.propTypes = {
    children: PropTypes.node,
  }

  return rtlRender(ui, {wrapper: Wrapper, ...options})
}

export * from '@testing-library/react'

export { render }
```

Now to customize your provider in your test, you can do:
```jsx
	render(<CalculatorDisplay value="0" />, { theme: 'light' })
```


To create an even more professional solution, is it possible to avoid the import hell `../../../test/render-utils` adding it to `moduleDirectories` inside your jest configuration file (`jest.config.js`)
```js
module.exports = {
...
	moduleDirectories: [
		...
		path.join(__dirname, 'test'),
	],
}
```

Now it is possible to simply use:
```js
import { render } from 'render-utils'
```

Unfortunately, doing that creates an issue with eslint because it is doesn't know how to resolve `render-utils` and to fix that you need to install `eslint-import-resolver-jest` as dev dependency and add to your `.eslintrc`
```js
module.exports = {
...
	overrides: [
	...
		{
			files: ['**/__tests__/**'],
			settings: {'import/resolver': {
				jest: {
					jestConfigFile: path.join(__dirname, 'jest.config.js')
				}
			}}
		}
	]
}
```

And finally you will need also add in your `jsconfig.json` or `tsconfig.json` (if you are using typescript)
```json
{
	"compilerOptions": {
		"paths": {
			"*": [
				"test/*"
				...
			]
			...
		}
		...
	},
	"include": [
		"test/*"
	...
	]
	...
}
```
to make your test folder included to your JS/TS compiler, this allow you to for example ctrl + click on the `render` method and be taken to the declaration file of render (in this case `test/render-utils.js`).
>[!note]
>I will complement this lesson that uses, webpack and babel with my own implementation with vitest on present-day react applications

## 📡 Jest watch mode
Passing `--watch` flag along your jest script, you runs it in a watch mode
```bash
npx jest --watch
```
that reruns your tests when a file is changed, in watch mode you can press some keys to have additional features:

 › Press a to run all tests.
 › Press f to run only failed tests.
 › Press p to filter by a filename regex pattern.
 › Press t to filter by a test name regex pattern.
 › Press u to update failing snapshots.
 › Press i to update failing snapshots interactively.
 › Press q to quit watch mode.
 › Press Enter to trigger a test run.

## 🪲 Debugging 
### 🌎 Chrome Dev Tools
To have a better output than `console.log` in your terminal, you can make use of chrome to inspect your tests in real time, the solution though, is not straightforward.

Fist of all, add a `debugger` statement wherever you want to place a breakpoint in your code.

Then proceed to add the following script to your `package.json`
```json
{
...
	"scripts": {
		"test:debug": "node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand --watch",
	...
	}
}
```
>[!info]
>`--inspect-brk` flag will monitor breakpoints in your code and will pause the running on debuggers
>
>`./node_modules/jest/bin/jest.js` is the process that will be executed by node (in this case the test runner)
>
> `--runInBand` disables multi threading on node that avoids creating multiple processes, this is not desirable for this case.

To debug in Google Chrome (or any Chromium-based browser), open your browser and go to `chrome://inspect` and click on "Open Dedicated DevTools for Node", which will give you a list of available node instances you can connect to. Click on the address displayed in the terminal (usually something like `localhost:9229`) after running the above command, and you will be able to debug Jest using Chrome's DevTools.

### 💻 VSCode

Create a file `launch.json` inside your `.vscode` folder at the top level of your repo, with the following content:

```json
{
	"version": "0.2.0",	
	"configurations": [
		{
			"name": "Debug Jest Tests",	
			"type": "node",
			"request": "launch",
			"runtimeArgs": [
				"--inspect-brk",
				"${workspaceRoot}/node_modules/.bin/jest",
				"--runInBand"
			],
			"console": "integratedTerminal",
			"internalConsoleOptions": "neverOpen"
		}
	]
}
```
>Check out [[https://jestjs.io/docs/troubleshooting#debugging-in-vs-code|Troubleshooting - Jest]] to see more examples of `launch.json` configuration for windows or create-react-app

## 📑 Coverage
To check how much of your files are covered by tests simply add `--coverage` to your jest run.

This will both output a table in your terminal and generate a `coverage` folder at the top of your project that will show information about the files/folders, statements, branches, functions, and lines that were or not being covered by tests.

- Statements: means the executable lines of code (variable declarations, function calls, return statements)
- Branches: means different paths that your execution flows through, such as if/switch statements and if tests reaches it.
- Functions: Check how many declared functions were actually being called during your tests.
- Lines: Exhibit how many lines of your code are being executed or not, both executable and non-executable lines (e.g., blank lines or comments).


Here's an example of how an coverage report looks like:
![Code coverage on Terminal](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/4-cli-coverage-report.png)
>Note that even though all test are passing, are some indications about what is being tested or not.

![Coverage Report on UI](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/5-ui-coverage-report.png)
>Here's a UI view inside browser in `coverage/lcov-report/index.html`. You can navigate through the links to check individual files from each folder

Note that checking coverage report for some folders is not always necessary, as example of `test` because it contains tools that will be used several times, showing 100% coverage for it, misleading up the percentage of your overall coverage. Also there are some files that is not being included, such as `app.js`, `index.js`. so to give more clarification, lets add the following line to `jest.config.js`

```js
module.exports = {
...
	collectCoverageFrom: ['**/src/**/*.js'],
}
```
This will check only files inside src (ignoring `test`) and ending with .js. Jest will ignore `__test__` folder and `*.(test|spec).js` files by default

![CoverageFrom update report](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/6-collectCoverageFrom-update-report-.png)
>Note that `test` folder is not being checked anymore, and some files inside `src` are now included.

Remember also to include `coverage` folder into your `.gitignore` file to avoid committing it.

If you are curious how jest analyses where your code is being tested or not, it does so using: [[https://www.npmjs.com/package/babel-plugin-istanbul|babel-plugin-istanbul]] check the [[https://gist.github.com/kentcdodds/3f57448c47c404e12200815048b1cc01|example gist from Kent C Dodds]] for details.

## 📊 Coverage Threshold
To setup threshold for your tests to keep everything testes specially when your codebase grows, you can add the following in your `jest.config.js`:
```js
module.exports = {
...
	coverageThreshold: {
	    global: {
		    statements: 100,
		    branches: 100,
		    functions: 100,
			lines: 100,
	    },
	},
}
```
>[!tip]
>Remember to consider realistic thresholds to your code base, the tip is to set nearly below the current percentage indicated on coverage report

You can also set different threshold for individual files or folders depending on the scenario you have, inside `jest.config.js`:
```js
module.exports = {
...
	coverageThreshold: {
	...
		// you can specify any glob pattern to files or folders
		'.src/shared/utils.js': {
			statements: 100,
		    branches: 80,
		    functions: 100,
			lines: 100,
		}
	}
}
```
>Be careful on adding different files to your `coverageThreshold` because they will be pulled out from `global` average, impacting on your coverage values that can go below the values on global

## 🧫 Testing in CI environment

It is possible can make use of [is-ci-cli](https://www.npmjs.com/package/is-ci-cli) to create a script that will work different if you are in local or CI environment, for example if you change your test script inside `package.json`:
```json
{
...
	"scripts": {
		"test": "is-ci \"test:coverage\" \"test:watch\"",
		...
	}
}
```

## ♻️ Reuse common imports for all test files with setupFilesAfterEnv

`setupFilesAfterEnv` modules are meant for code which is repeating in each test file. Having the test framework installed makes Jest [globals](https://jestjs.io/pt-BR/docs/api), [`jest` object](https://jestjs.io/pt-BR/docs/jest-object) and [`expect`](https://jestjs.io/pt-BR/docs/expect) accessible in the modules. For example, you can add extra matchers from [`jest-extended`](https://github.com/jest-community/jest-extended) library or call [setup and teardown](https://jestjs.io/pt-BR/docs/setup-teardown) hooks:

__setup-jest.js__
```js
import * as jestDOM from '@testing-library/jest-dom/extend-expect'
import matchers from 'jest-extended'
expect.extend(matchers);

afterEach(() => {
  jest.useRealTimers();
});
```

Applying `setupFilesAfterEnv` in your `jest.config.js` makes you avoid to import and extend on every file:
```js
module.exports = {
...
	setupFilesAfterEnv: ['<rootDir>/setup-jest.js'],
}
```
## ⚗️ Running tests with different configurations

Sometimes you may have situations where configuration needs to be different for certain tests, for example if you have a monorepo that contains code from client and server, it will be useful to specify different test configs, for example, in server side the `testEnvironment` should be `jest-environment-node` and for client, should be `jest-environment-jsdom`. Suppose your monorepo have a folder structure like this:

```
jest.config.js
package.json
client
|    jest.config.js
|    package.json
|----src
|--------__tests__
|            app.js
|        app.js
server
|	jest.config.js
|   package.json
|----src
|--------__tests__
|            api.js
|        api.js
```

Root file should contain common configurations that are shared across environments, while you can have individual parameters for each `client` and `server` environments:

`jest.config.js`
```js
const path = require('path')

module.exports = {
  moduleDirectories: [
    'node_modules',
    path.join(__dirname, 'src'),
    'shared',
    path.join(__dirname, 'test'),
  ],
  moduleNameMapper: {
    '\\.module\\.css$': 'identity-obj-proxy',
    '\\.css$': require.resolve('./test/style-mock.js'),
  },
  collectCoverageFrom: ['**/src/**/*.js'],
}
```

`client/jest.config.js`
```js
const path = require('path')

module.exports = {
  ...require('../jest.config'),
  displayName: {
	name: 'client',
    color: 'magenta',
  },
  rootDir: path.join(__dirname, '..'),
  testEnvironment: 'jest-environment-jsdom',
  setupFilesAfterEnv: ['jest-extended'],
  coverageThreshold: {
    global: {
      statements: 15,
      branches: 10,
      functions: 15,
      lines: 15,
    },
  },
}
```
>[!tip]
>`rootDir` param can be very handy if for example your config files (like `.babelrc`) file is in another directory

`server/jest.config.js`
```js
const path = require('path')

module.exports = {
	...require('../jest.config'),
	displayName: {
		name: 'server',
	    color: 'blue',
	},
	coverageDirectory: path.join(__dirname, '../coverage/server'),
	testEnvironment: 'jest-environment-node',
}
```

Then you can run your tests using custom config files:
```bash
npx jest --config client/jest.config.js
```
or update your scripts in your root `package.json`
```json
{
...
	"scripts": {
		"test:client": "jest --config client/jest.config.js",
		"test:client:dev": "jest --config client/jest.config.js --watch",
		"test:server": "jest --config server/jest.config.js",
		"test:server:dev": "jest --config server/jest.config.js --watch",
		...
	}
}
```

If you have a `.eslintrc.js` file , remember to add your jest files to it
```js
module.exports = {
  overrides: [
    {
      files: ['**/__tests__/**'],
      settings: {
        'import/resolver': {
          jest: {
            jestConfigFile: path.join(__dirname, 'client_or_server/jest.config.js'),
          },
        },
      },
    },
    ...
  ],
  ...
}

```

## 🧰 Support multiple project configurations

Instead of having multiple scripts `test:client`, `test:client:dev`, `test:server`, and `test:server:dev` for example, you can setup `projects` in your root jest config, Jest will run tests in all of the specified projects at the same time. This is great for monorepos or when working on multiple projects at the same time.

`jest.config.js`
```js
module.exports = {
	projects: ['client', 'server'],
	...
}
```

## 🔩 Running ESLint with jest
One way to integrate your jest with eslint, running only your test script instead of both test and lint is to add [jest-runner-eslint](https://github.com/jest-community/jest-runner-eslint) as a dev dependency
```bash
npm i -D jest-runner-eslint
```

And now you can create a config file for it. Let's create it inside our `client`folder named as `jest.config.eslint.js`
```
const path = require('path')

module.exports = {
  displayName: {
    name: 'lint',
    color: 'gray',
  },
  rootDir: path.join(__dirname, '..'),
  runner: 'jest-runner-eslint',
  testMatch: ['<rootDir>/**/*.js'],
}
```

Add it to your `projects` on root `jest.config.js`:
```js
module.exports = {
	projects: [
	    'src/jest.config.eslint.js',
	    'src/jest.config.js',
	    'server/jest.config.js',
	],
}
```

And finally, to avoid linting every kind of file, you should ignore them as using `--ignore-path .gitignore` flag on eslint, to achieve that, you have to configure your dependency on `package.json`
```json
{
	"jest-runner-eslint": {
	    "clipOptions": {
		    "ignorePath": "./.gitignore"
	    }
	},
}
...
```

Now you have all the benefits from eslint on your test runner

## 🧿 Test Specific Projects in Jest Watch Mode with jest-watch-select-projects

As you project grows, test with multiple projects configured will run all of them at once and it will be nice if we could re-run only the projects that we are working on, to achieve this, we have [jest-watch-select-projects](https://www.npmjs.com/package/jest-watch-select-projects).

Install it with: 
```bash
npm i -D jest-watch-select-properties
```

And add it to your root `jest.config.js`:
```js
module.exports = {
	watchPlugins: ['jest-watch-select-projects'],
	...
}
```

with it is possible to run tests in watch mode and have a new option 

`› Press P to select projects (all selected).`

Pressing "capital P" you will be prompted with:
![jest-watch-select-projects-prompt.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/7-jest-watch-select-projects-prompt.png)
and you can re run only tests relevant to your current work

## 🪬 Filter which Tests are Run with Typeahead Support in Jest Watch Mode
Another interesting plugin for watch mode is [jest-watch-typeahead](https://www.npmjs.com/package/jest-watch-typeahead). It allows you to get suggestions of which files will match with your regex input when you press "p" to filterin watch mode.

Install it with:
```bash
npm i -D jest-watch-typeahead
```

And add the following to your root `jest.config.js`:
```js
module.exports = {
	watchPlugins: [
		'jest-watch-typeahead/filename',
		'jest-watch-typeahead/testname',
	  ...
	],
	...
}
```

Now running jest in watch mode and pressing "t" or "p", you will have:
![8-jest-watch-typeahead-test-empty.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/8-jest-watch-typeahead-test-empty.png)
>Note the *Start typing to filter by a test name regex pattern.* that didn't exist before

As you type something, it will be displayed the matches, and you can navigate through them with arrow keys:
![9-jest-watch-typeahead-test-match.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/9-jest-watch-typeahead-test-match.png)

## 🛸 Run Only Relevant Jest Tests on Git Commit to Avoid Breakages

To have your tests running before commiting files to git, you can use `husky` to create a pre-commit hook, however, if you run all tests every time you commit, you can have a great lagging running all the tests every single time, to avoid that, we can use also `lint-staged`.

To install them:
```bash
npm i -D husky lint-staged
```

And create the following configuration in your root `package.json`
```json
{
	"husky": {
	    "hooks": {
	      "pre-commit": "lint-staged && npm run build"
	    }
	},
	"lint-staged": {
	    "**/*.+(js|json|css|html|md)": [
	      "prettier",
	      "jest --findRelatedTests",
	      "git add"
	    ]
	},
	...
}
```

This will run only the tests that is related with your changes, that are in staging.

# ⚛️ Test React Applications with Jest + React Testing Library (RTL)

## ✨ Disclaimer: Course updates

- Destructuring `render` result was replaced from `import { screen } from '@testing-librtary/react'`
- Use `userEvent` rather than `fireEvent` (e.g.: `userEvent.type` replaces `fireEvent.change`)
- Replace `wait` with `waitfor` from `'@testing-librtary/react'`
- Switch `createMemoryHistory` from `history` and `Router` from `react-router-dom` to `window.history.pushState({}, 'Test Page', '/')` and `BrowserRouter` respectively.
- Added MSW to mock HTTP requests

## 🩻 Render a React component for testing

Here's a basic example of react component that we want to test
```jsx
import * as React from 'react'

function FavoriteNumber({min = 1, max = 9}) {
  const [number, setNumber] = React.useState(0)
  const [numberEntered, setNumberEntered] = React.useState(false)
  function handleChange(event) {
    setNumber(Number(event.target.value))
    setNumberEntered(true)
  }
  const isValid = !numberEntered || (number >= min && number <= max)
  return (
    <div>
      <label htmlFor="favorite-number">Favorite Number</label>
      <input
        id="favorite-number"
        type="number"
        value={number}
        onChange={handleChange}
      />
      {isValid ? null : <div role="alert">The number is invalid</div>}
    </div>
  )
}

export {FavoriteNumber}
```

This component contains state, ternary statements  and callbacks for handling user input. To test if it is rendering correctly only using react and react-dom, we can do the following:
```jsx
import * as React from 'react'
import ReactDOM from 'react-dom'
import {FavoriteNumber} from '../favorite-number'

test('renders a number input with a label "Favorite Number"', () => {
  const div = document.createElement('div')
  ReactDOM.render(<FavoriteNumber />, div)
  expect(div.querySelector('input').type).toBe('number')
  expect(div.querySelector('label').textContent).toBe('Favorite Number')
})
```

Here we have created an empty div and bounded it with our react component using `ReactDOM.render`, and checked if the input was number-type and the label content with basic Jest assertions.

If you want to have a better error output when you commit some typo, you can use `toHaveAttribute` and `toHaveTextContent` property from `jest-dom`

```js
expect(div.querySelector('input')).toHaveAttribute('type', 'number')
expect(div.querySelector('label')).toHaveTextContent('Favorite Number')
```

If you mistype, for example:
```js
expect(div.querySelector('input').type).toBe('number')
```

The output will be:
![10-jest-output-null-querySelector.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/10-jest-output-null-querySelector.png)
Note that will only show a generic error saying: `TypeError: Cannot read properties of null (reading 'type')`

But the same typo error with `toHaveAttribute` assertion
```js
expect(div.querySelector('nput')).toHaveAttribute('type', 'number')
```

Will output the following error message:
![11-jest-output-toHaveAttribute.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/11-jest-output-toHaveAttribute.png)
This output `received value must be an HTMLElement or an SVGElement. Received has value: null` gives you a slightly better clue about what is going on.

A common situation that may happen when handling inputs is break the connection between an input `id` and its label bounded by `htmlFor` or `aria-labelledby` this is usually not covered by tests. So an abstraction to test this kind of accessibility is using `queries` from `@testing-library/dom`

```jsx
import {queries} from '@testing-library/dom'
...
const div = document.createElement('div')
ReactDOM.render(<FavoriteNumber />, div)

const input = queries.getByLabelText(div, /favorite Number/i)
```
>regex `/favorite Number/i`  is user instead of 'Favorite number' to allow case insensitivity.

So if a typo occurs in `htmlFor` or `id` the output of the test will show the following error:
```bash
TestingLibraryElementError: Found a label with the text of: /favorite number/i, however no form control was found associated to that label. Make sure you're using the "for" attribute or "aria-labelledby" attribute correctly.
```

To shorten the `getByLabelText` you can use `getQueriesForElement` once at the div that will return the function `getByLabelText`, this way:
```jsx
import { queries, getQueriesForElement } from '@testing-library/dom'
...
const div = document.createElement('div')
ReactDOM.render(<FavoriteNumber />, div)

const { getByLabelText } = getQueriesForElement(div)
const input = getByLabelText(/favorite number/i)
```

To make it reusable, you can call it inside a function called `render`:
```js
function render(ui) {
	const container = document.createElement(div)
	
	ReactDOM.render(ui, container)
	const queries = getQueriesForElement(container)
	return queries
}
```

And this is the main functionality of `render` that is already provided by `@testing-library/react`. You can safely replace your `render` method by it.
```js
import { render } from '@testing-library/react'
```

## 🐞 Debbugging components in RTL
React Testing Library provides a `debug` method on return from `render`, you can use it to log your component (or part of it) at any point of testing

```jsx
test('renders a number input with a label "Favorite Number"', () => {
	const { getByLabelText, debug } = render(<FavoriteNumber />)
	debug() // here it will print the whole FavoriteNumber
	const input = getByLabelText(/favorite number/i)
	debug(input) // here it will print only the input inside FavoriteNumber
});
```
>[!tip]
>Usually is not a good practice to keep `debug` in your commits

## 🖱️ Handling user events
In [[Testing JavaScript#🩻 Render a React component for testing]] `<FavoriteNumber />` component contains a validation if passed number on input is between a certain range or not, to test that behavior, is it possible to interact with component using user events as follows

```jsx
import user from '@testing-library/user-event'

test('entering an invalid value shows an error message', async () => {
  const screen = render(<FavoriteNumber />)
  
  const input = screen.getByLabelText(/favorite number/i)
  
  user.type(input, '10')
  const alert = screen.getByRole('alert')
  expect(alert).toHaveTextContent(/the number is invalid/i)
})
```

## 🔄 Testing prop updates with rerender

During your tests, if you want to update your component with different props you can use `rerender(ui)` to fire an update in your component.
```jsx
import user from '@testing-library/user-event'

test('entering an invalid value shows an error message', async () => {
  const screen = render(<FavoriteNumber />)
  
  const input = screen.getByLabelText(/favorite number/i)
  
  user.type(input, '10')
  const alert = screen.getByRole('alert')
  expect(alert).toHaveTextContent(/the number is invalid/i)
  
  screen.debug()
  screen.rerender(<FavoriteNumber max={10} />)
  screen.debug()
})
```

You will notice that we have placed two `debug` calls, before and after the `rerender` call. Note that the first call will output the alert message, but not for the second call.
```html
<body>
	<div>
		<div>
			<label for="favorite-number">		
				Favorite Number
			</label>
			<input
				id="favorite-number"
				type="number"
				value="0"
			/>
			<div role="alert">
				The number is invalid
			</div>
		</div>
	</div>
</body>
```
>First call

```html
<body>
	<div>
		<div>
			<label for="favorite-number">		
				Favorite Number
			</label>
			<input
				id="favorite-number"
				type="number"
				value="0"
			/>
		</div>
	</div>
</body>
```
>Second call

## 🫥 Assert that element is not rendered in DOM

Instead of using queries with `get` prefix like `getByRole`, `getByTestId`, `getByLabelText`, `getAllBy...` is it possible to make queries with `query` prefix. The main difference between them is that `get` throws an exception with a better log output when no matching element is found and `query` only returns null. Therefore, query is pretty much handy to assert that an element is not present in dom

```jsx
import user from '@testing-library/user-event'

test('entering an invalid value shows an error message', async () => {
  const screen = render(<FavoriteNumber />)
  
  const input = screen.getByLabelText(/favorite number/i)
  
  user.type(input, '10')
  
  let alert = screen.getByRole('alert')
  
  expect(alert).toHaveTextContent(/the number is invalid/i)
  
  screen.rerender(<FavoriteNumber max={10} />)

  alert = screen.queryByRole('alert')
  expect(alert).not.toBeInTheDocument()
})
```

## ♿ Testing basic accessibility with jest-axe

Disclaimer from [jest-axe page](https://github.com/nickcolley/jest-axe#readme): Tools like axe are similar to [code linters](https://en.wikipedia.org/wiki/Lint_%28software%29) such as [eslint](https://eslint.org/) or [stylelint](https://stylelint.io/): they can find common issues but cannot guarantee that what you build works for users.

You'll also need to:

- test your interface with the [assistive technologies that real users use](https://www.gov.uk/service-manual/technology/testing-with-assistive-technologies#when-to-test) (see also [WebAIM's survey results](https://webaim.org/projects/screenreadersurvey8/#primary)).
- include disabled people in user research.

### 📦 Install
```shell
npm install --save-dev jest-axe
```

### ⚙️ Configuration
You can use as the following example:

```jsx
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage', async () => {
  const screen = render(<img src="#" />)

  const results = await axe(screen.container)

  expect(results).toHaveNoViolations()
})
```

the output will be something like this:
```shell
Expected the HTML found at $('img') to have no violations:

    <img src="#">

    Received:

    "Images must have alternate text (image-alt)"

    Fix any of the following:
      Element does not have an alt attribute
      aria-label attribute does not exist or is empty
      aria-labelledby attribute does not exist, references elements that do not exist or references elements that are empty
      Element has no title attribute
      Element default semantics were not overridden with role="none" or role="presentation"

    You can find more information on this issue here: 
    https://dequeuniversity.com/rules/axe/4.8/image-alt?application=axeAPI
```

You can replace
```js
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)
```

with
```js
`import 'jest-axe/extend-expect'`
```

Or, even better as mentioned in [[Testing JavaScript#♻️ Reuse common imports for all test files with setupFilesAfterEnv]] you can automate import/extend of `toHaveNoViolations` creating a `setup-env.js` or adding to an existing one the following part:
```js
...
import 'jest-axe/extend-expect' 
```

and couple this file in your `jest.config.js`
```js
module.exports = {
...
	setupFilesAfterEnv: ['<rootDir>/setup-jest.js'],
}
```


## 👺Mocking HTTP Requests

### 🃏Using jest.mock
It is possible to use jest.mock any kind of module to mock modules, so supposing an async HTTP call called `loadGreeting`:
```jsx
import { loadGreeting } from './api'

function GreetingLoader() {
  const [greeting, setGreeting] = React.useState('')
  async function loadGreetingForInput(e) {
    e.preventDefault()
    const {data} = await loadGreeting(e.target.elements.name.value)
    setGreeting(data.greeting)
  }
  return (
    <form onSubmit={loadGreetingForInput}>
      <label htmlFor="name">Name</label>
      <input id="name" />
      <button type="submit">Load Greeting</button>
      <div aria-label="greeting">{greeting}</div>
    </form>
  )
}
```

It is possible to create a test file such as:
```jsx
import user from '@testing-library/user-event'
import { loadGreeting } from '../api'

jest.mock('../api')

test('loads greetings on click', () => {
  mockLoadGreeting.mockResolvedValueOnce({data: {greeting: 'TEST_GREETING'}})
  const {getByLabelText, getByText} = render(<GreetingLoader />)
  const nameInput = getByLabelText(/name/i)
  const loadButton = getByText(/load/i)
  nameInput.value = 'Mary'
  user.click(loadButton)
})
```

`mockedResolvedValueOnce` lets you control when your function behaves different for multiple calls
```js
test('async test', async () => {
  const asyncMock = jest
    .fn()
    .mockResolvedValue('default')
    .mockResolvedValueOnce('first call')
    .mockResolvedValueOnce('second call');

  await asyncMock(); // 'first call'
  await asyncMock(); // 'second call'
  await asyncMock(); // 'default'
  await asyncMock(); // 'default'
});
```

Although the test passes, you will be prompted with a warning that says:
```shell
console.error
    Warning: An update to GreetingLoader inside a test was not wrapped in act(...).
    
    When testing, code that causes React state updates should be wrapped into act(...):
    
    act(() => {
      /* fire events that update state */
    });
    /* assert on the output */
    
    This ensures that you\'re testing the behavior the user would see in the browser. Learn more at https://reactjs.org/link/wrap-tests-with-act
        at GreetingLoader (/home/revolutionary-linux/Documentos/estudos/JavaScript/testing-javascript/react-testing-library-course/src/greeting-loader-01-mocking.js:5:41)

       7 |     e.preventDefault()
       8 |     const {data} = await loadGreeting(e.target.elements.name.value)
    >  9 |     setGreeting(data.greeting)
         |     ^
      10 |   }
      11 |   return (
      12 |     <form onSubmit={loadGreetingForInput}>

      at printWarning (node_modules/react-dom/cjs/react-dom.development.js:67:30)
      at error (node_modules/react-dom/cjs/react-dom.development.js:43:5)
      at warnIfNotCurrentlyActingUpdatesInDEV (node_modules/react-dom/cjs/react-dom.development.js:24064:9)
      at setGreeting (node_modules/react-dom/cjs/react-dom.development.js:16135:9)
      at loadGreetingForInput (src/greeting-loader-01-mocking.js:9:5)
```
What it's saying here is that an update to `GreetingLoader` inside the test was not wrapped in `act`. When testing, code that causes state update should be wrapped in act and act. It gives us some more information. What's happening here is after our test completes, our component actually isn't done.

On the next tick of the event loop, our promise resolves, and then we call this `setGreeting`. That is what React is complaining about. It's saying, "Hey, you called `setGreeting`, but I have no way to know when I should flush these updates to the DOM, and you're probably missing something in your test," which is exactly the case.

It's telling us that we need to wrap our code in an `act` function. We don't actually need to do that, because React Testing Library does that for us automatically. We're going to import `waitFor` from React Testing Library, which is async utility. We need to wait for this assertion to pass.

So even if you try to wrap your `user.click(loadButton)` inside an `act(() => user.click(loadButton))` the warning will persist, actually what need to be done is import `waitFor` from react testing library instead.

```jsx
import user from '@testing-library/user-event'
import { loadGreeting } from '../api'

jest.mock('../api')

test('loads greetings on click', () => {
  mockLoadGreeting.mockResolvedValueOnce({data: {greeting: 'TEST_GREETING'}})
  const {getByLabelText, getByText} = render(<GreetingLoader />)
  const nameInput = getByLabelText(/name/i)
  const loadButton = getByText(/load/i)
  nameInput.value = 'Mary'
  user.click(loadButton)
  // Added waitFor
  await waitFor(() =>
    expect(screen.getByLabelText(/greeting/i)).toHaveTextContent(testGreeting),
  )
})
```
This way it is ensuring that the setGreeting was been called and rendered the message on page.

### 🏗️Using MSW

Alternatively, we can use [MSW](https://mswjs.io/) to intercept external requests, first you will need to config a `setupServer` specifying the endpoint that you want to mock:
```js
import {rest} from 'msw'
import {setupServer} from 'msw/node'

const server = setupServer(
  rest.post('/greeting', (req, res, ctx) => {
  return res(ctx.json({data: {greeting: `Hello ${req.body.subject}`}}))
}),
```

And in your test files you can start to intercept your requests with `server.listen()`, usually is recommended to do that in a `beforeAll` hook.
```js
beforeAll(() => server.listen({onUnhandledRequest: 'error'}))
```
>[!info]
>`{onUnhandledRequest: 'error'}` outputs an error indicating that some request is not mocked

Some additional utilities are recommended to use to ensure the consistency of your HTTP mocks, such as
```js
afterAll(() => server.close())
afterEach(() => server.resetHandlers())
```
`close` will ensure that your interceptor is closed, and `server.resetHandlers` will reset all your requests to the original state.

## 🪤Test componentDidCatch Handler Error Boundaries
There is a pattern in React to catch unexpected errors that may happen within your components using the following `ErrorBoundary` component:
```jsx
import * as React from 'react'
import {reportError} from './api'

class ErrorBoundary extends React.Component {
  state = {hasError: false}
  componentDidCatch(error, info) {
    this.setState({hasError: true})
    reportError(error, info)
  }
  tryAgain = () => this.setState({hasError: false})
  render() {
    return this.state.hasError ? (
      <div>
        <div role="alert">There was a problem.</div>{' '}
        <button onClick={this.tryAgain}>Try again?</button>
      </div>
    ) : (
      this.props.children
    )
  }
}

export {ErrorBoundary}
```

The following test file can cover the funcionality:
```jsx
import * as React from 'react'
import {render} from '@testing-library/react'
import {reportError as mockReportError} from '../api'
import {ErrorBoundary} from '../error-boundary'

jest.mock('../api')

afterEach(() => {
  jest.clearAllMocks()
})

function Bomb({shouldThrow}) {
  if (shouldThrow) {
    throw new Error('💣')
  } else {
    return null
  }
}

test('calls reportError and renders that there was a problem', () => {
  mockReportError.mockResolvedValueOnce({success: true})
  const {rerender} = render(
    <Bomb />
    {wrapper: ErrorBoundary}
  )

  rerender(
    <Bomb shouldThrow={true} />
  )

  const error = expect.any(Error)
  const info = {componentStack: expect.stringContaining('Bomb')}
  expect(mockReportError).toHaveBeenCalledWith(error, info)
  expect(mockReportError).toHaveBeenCalledTimes(1)
})
```

this will render two console.error in your test outputs (one from jest-dom, another from react-dom), to avoid this kind of message specifically for this case, you can do 
```jsx
beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {})
})

afterAll(() => {
  console.error.mockRestore()
})

test('calls reportError and renders that there was a problem', () => {
  ...
  
  expect(console.error).toHaveBeenCalledTimes(2)
})
```
>[!info]
>`expect(console.error).toHaveBeenCalledTimes(2)` asserts that only the undesired `console.errors` are being suppressed, if for some reason there is more, the test will fail

## 🩹 Ensure Error Boundaries Can Successfully Recover from Errors

To rerender your component without the error, you can add the following at the end of your test:
```jsx
  expect(screen.getByRole('alert').textContent).toMatchInlineSnapshot(
    `"There was a problem."`,
  )

  console.error.mockClear()
  mockReportError.mockClear()

  rerender(
    <Bomb />
  )

  userEvent.click(screen.getByText(/try again/i))

  expect(mockReportError).not.toHaveBeenCalled()
  expect(console.error).not.toHaveBeenCalled()
  expect(screen.queryByRole('alert')).not.toBeInTheDocument()
  expect(screen.queryByText(/try again/i)).not.toBeInTheDocument()
```

## 🎡Test Driven Development

### 📑Testing a form
The Idea behind TDD(Test Driven Development) is to first create your tests assertion and then writing your implementation of if, a process called _red-green refactor_ (because your console starts with red errors and finishes with green success messages) this is a cycle process of writing tests, write your actual code implementation, refactor/improve your tests, and refactor/improve your code until it is done.

For example, we want to create a form with an input __title__, a textarea __content__, and another input __tags__ so we can start by creating a simple test ensuring that the elements are present on the component:
```jsx
test('renders a form with title, content, tags, and a submit button', () => {
  const { getByLabelText, getByText } = render(<Editor />)
  getByLabelText(/title/i)
  getByLabelText(/content/i)
  getByLabelText(/tags/i)
  getByText(/submit/i)
})
```

Supposing you have created an empty `Editor` component such as:
```jsx
function Editor() {
  return <></>
}
```

Then run the test in the watch mode to check the error while the `Editor` is being updated, until you have reached something like:
```jsx
export function Editor() {
  return (
    <form>
      <label htmlFor="title-input">Title</label>
      <input id="title-input" />

      <label htmlFor="content-input">Content</label>
      <textarea id="content-input" />

      <label htmlFor="tags-input">Tags</label>
      <input id="tags-input" />

      <button type="submit">Submit</button>
    </form>
  )
}
```

### 📤 Submitting the form
Let's say that when the submit button is clicked, it becomes disabled, so we can refactor our tests adding the following assertions:
```jsx
test('renders a form with title, content, tags, and a submit button', () => {
  ...
  const submitButton = screen.getByText(/submit/i)
  userEvent.click(submitButton)
  expect(submitButton).toBeDisabled()
})
```

The test will now fail, and the `Editor` needs to be updated with:
```jsx
function Editor() {
  const [isSaving, setIsSaving] = React.useState(false)
  function handleSubmit(e) {
    e.preventDefault()
    setIsSaving(true)
  }

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="title-input">Title</label>
      <input id="title-input" />

      <label htmlFor="content-input">Content</label>
      <textarea id="content-input" />

      <label htmlFor="tags-input">Tags</label>
      <input id="tags-input" />

      <button type="submit" disabled={isSaving}>
        Submit
      </button>
    </form>
  )
}
```

### ⏫ TDD on API Call of a react form
Now the implementation should call a POST request to an API on form submission, to test drive it we need to first mock our service with `jest.mock` and then fill the fields using `userEvent.type`, and check if our mocked function was been called with the correct data. And to keep the values consistent they can be centralized in a 
`fakePost` and `fakeUser` objects. the final code will look like below:
```js
import { savePost as mockSavePost } from  './api'

jest.mock('../api')

afterEach(() => {
  jest.clearAllMocks()
})

test('renders a form with title, content, tags, and a submit button', () => {
  mockSavePost.mockResolvedValueOnce()
  const { getByLabelText, getByText } = render(<Editor />)
  
  const fakeUser = {id: 'user-1'}
  const fakePost = {
    title: 'Test Title',
    content: 'Test content',
    tags: ['tag1', 'tag2'],
  }
  
  const title = getByLabelText(/title/i)
  const content = getByLabelText(/content/i)
  const tags = getByLabelText(/tags/i)
  
  userEvent.type(title, fakePost.title)
  userEvent.type(content, fakePost.content)
  userEvent.type(tags, fakePost.tags.join(', '))
  
  const submitButton = screen.getByText(/submit/i)
  userEvent.click(submitButton)

  expect(submitButton).toBeDisabled()
  expect(mockSavePost).toHaveBeenCalledWith({
    ...fakePost,
    authorId: fakeUser.id,
  })
  expect(mockSavePost).toHaveBeenCalledTimes(1)
})
```

Now we want to refactor our component. we need to add `name` property in each of the fields to acces them inside `handleSubmit` method through `e.target.elements` 
```jsx
import * as React from 'react'
import {savePost} from './api'

function Editor({user}) {
  const [isSaving, setIsSaving] = React.useState(false)
  function handleSubmit(e) {
    e.preventDefault()
    const {title, content, tags} = e.target.elements
    const newPost = {
      title: title.value,
      content: content.value,
      tags: tags.value.split(',').map((t) => t.trim()),
      authorId: user.id,
    }
    setIsSaving(true)
    savePost(newPost)
  }
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="title-input">Title</label>
      <input id="title-input" name="title" />

      <label htmlFor="content-input">Content</label>
      <textarea id="content-input" name="content" />

      <label htmlFor="tags-input">Tags</label>
      <input id="tags-input" name="tags" />

      <button type="submit" disabled={isSaving}>
        Submit
      </button>
    </form>
  )
}

export {Editor}

```

### 🔀 Mocking react-router-dom Redirect on form submission
Now we want to add a feature that redirects users after sending our API request, to test drive that, we need to mock the `react-router-dom` package with `jest.mock` and replace `Redirect` with `jest.fn()`. Then is possible to check over how it has been called using `toHaveBeenCalledWith` and `toHaveBeenCalledTimes` as the example below:

```js
import {Redirect as MockRedirect} from 'react-router'

jest.mock('react-router', () => {
  return {
    Redirect: jest.fn(() => null),
  }
})

test('renders a form with title, content, tags, and a submit button', async () => {
  ...

  expect(MockRedirect).toHaveBeenCalledWith({to: '/'})
  expect(MockRedirect).toHaveBeenCalledTimes(1)
})
```

Now for the component, it is possible to add
```jsx
function Editor({user}) {
...
 const [redirect, setRedirect] = React.useState(false)
  function handleSubmit(e) {
	e.preventDefault()
	const {title, content, tags} = e.target.elements
	const newPost = {
	  title: title.value,
	  content: content.value,
	  tags: tags.value.split(',').map((t) => t.trim()),
	  authorId: user.id,
	}
	setIsSaving(true)
	savePost(newPost).then(() => setRedirect(true))
  }
  if (redirect) {
	return <Redirect to="/" />
  }
  ...
}
```

You will notice that the test will fail with the following error message:
```shell
    expect(jest.fn()).toHaveBeenCalledWith(...expected)
    Expected: {"to": "/"}, {}
    Number of calls: 0
    ...
    console.error
    Warning: An update to Editor inside a test was not wrapped in act(...).
    
    When testing, code that causes React state updates should be wrapped into act(...):
    
    act(() => {
      /* fire events that update state */
    });
    /* assert on the output */
```
What is happening here is that `savePost` is a promise and `setRedirect` is being called asynchronously, and the assert is not handling that, so to fix this, is it possible to wrap the assertion in a `waitFor` function. A tip for `waitFor` is to use it as slim as possible passing only the relevant assertion inside it's callback to avoid taking a longer time to run your tests. Another recommendation is to remove `toHaveBeenCalledTimes` because since `Redirect` is a react component, its implementation can require that it will be loaded more times and this doesn't mean necessarily that the user have been redirected multiple times.

```js
import {Redirect as MockRedirect} from 'react-router'

jest.mock('react-router', () => {
  return {
    Redirect: jest.fn(() => null),
  }
})

test('renders a form with title, content, tags, and a submit button', async () => {
  ...

  waitFor(() => expect(MockRedirect).toHaveBeenCalledWith({to: '/'}))
})
```

### ⏱️ Handling Timestamps
Let's take the previous `savePost` call for the API and now should be necessary to inform the current timestamp, to test drive it, a starting point can be add a new property with a `Date` instance to it: 
```js
test('renders a form with title, content, tags, and a submit button', async () => {
  ...
  expect(mockSavePost).toHaveBeenCalledWith({
    ...fakePost,
    date: new Date().toISOString(),
    authorId: fakeUser.id,
  })
  ...
})
```

And in the component, simply add the same property
```jsx
function Editor({user}) {
  ...
  function handleSubmit(e) {
	...
	const newPost = {
	  title: title.value,
	  content: content.value,
	  tags: tags.value.split(',').map((t) => t.trim()),
	  date: new Date().toISOString(),
	  authorId: user.id,
	}
	...
  }
  ...
}
```

However, you will face some annoying error that indicates a slightly difference between both dates
```diff
- Expected
+ Received

    @@ -1,9 +1,9 @@
      Object {
        "authorId": "user-1",
        "content": "Test content",
-       "date": "2024-01-31T03:24:45.449Z",
+       "date": "2024-01-31T03:24:45.443Z",
        "tags": Array [
          "tag1",
          "tag2",
        ],
        "title": "Test Title",
```

To contour that, instead of comparing with a exact date, it should be suitable to compare within a range, before and after the submission of the form and retrieve the actual date from our `mockSavePost` as follows:
```js
test('renders a form with title, content, tags, and a submit button', async () => {
  ...
  const preDate = new Date().getTime()
  
  userEvent.click(submitButton)
  
  const postDate = new Date().getTime()
  
  const date = new Date(mockSavePost.mock.calls[0][0].date).getTime()
  expect(date).toBeGreaterThanOrEqual(preDate)
  expect(date).toBeLessThanOrEqual(postDate)
  ...
})
```

Alternatively, you can add `jest.useFakeTimers('modern')` to replace your dates with a standard value
```js
jest.useFakeTimers('modern')

test('renders a form with title, content, tags, and a submit button', async () => {
  ...
  expect(mockSavePost).toHaveBeenCalledWith({
    ...fakePost,
    date: new Date().toISOString(),
    authorId: fakeUser.id,
  })
  ...
})

```

Or even better, is it possible to add it to your `jest.config.js` as a global config:
```js
const config = {
  ...
  fakeTimers: {
    timers: 'modern',
    enableGlobally: true,
  },
};

module.exports = config;
```
>[!warning]
>`timers: 'modern'` is the default after version 27, in that case you will need to specify `timers: 'legacy'` if desired. [see more on New fake timers section of the docs](https://jestjs.io/blog/2020/05/05/jest-26#new-fake-timers)

A recommendation is to not affect other tests by wraping `jest.useFakeTimers()` within a `beforeEach` hook and cleanup it using `jest.useRealTimers()` at the `afterEach` hook
```js
beforeEach(() => {
  jest.useFakeTimers()
})

afterEach(() => {
  jest.useRealTimers()
})
```
### 🪄 Generating data with `test-data-bot`
[test-data-bot](https://github.com/jackfranklin/test-data-bot) is a library that can create fake values to be used throughout your test, the values are considered gibberish, and indicates that the value itself is not really important

```js
import {build, fake, sequence} from 'test-data-bot'
...

const postBuilder = build('Post').fields({
  title: fake((f) => f.lorem.words()),
  content: fake((f) => f.lorem.paragraphs().replace(/\r/g, '')),
  tags: fake((f) => [f.lorem.word(), f.lorem.word(), f.lorem.word()]),
})

const userBuilder = build('User').fields({
  id: sequence((s) => `user-${s}`),
})

test('renders a form with title, content, tags, and a submit button', async () => {
  ...
  const fakeUser = userBuilder()
  render(<Editor user={fakeUser} />)
  const fakePost = postBuilder()
  ...
})
```
this will replace your static values with mocks from test-data-bot

>[!tip]
>If you need do specify some value statically, you can pass arguments to your builder call, e.g.:
>
> `const fakePost = postBuilder({ title: 'Static Title' })`

### 🙅 Testing error states
To check for possible errors that may appear in your code, you can make use of `mockRejectedValueOnce` to throw an error at your promises. Let's create a new test that rejects the `savePost` call and try to look for a error message in the code, semantically defined with `alert` role:
```js
test('renders an error message from the server', async () => {
  const testError = 'test error'
  mockSavePost.mockRejectedValueOnce({data: {error: testError}})
  const fakeUser = userBuilder()
  render(<Editor user={fakeUser} />)
  const submitButton = screen.getByText(/submit/i)

  userEvent.click(submitButton)

  const postError = await screen.findByRole('alert')
  expect(postError).toHaveTextContent(testError)
  expect(submitButton).toBeEnabled()
})
```

Now in your component we can add the desired behavior with
```jsx
function Editor({user}) {
  ...
  const [error, setError] = React.useState(null)
  function handleSubmit(e) {
    ...
    savePost(newPost).then(
      () => setRedirect(true),
      (response) => {
        setIsSaving(false)
        setError(response.data.error)
      },
    )
  }
  ...
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="title-input">Title</label>
      <input id="title-input" name="title" />

      <label htmlFor="content-input">Content</label>
      <textarea id="content-input" name="content" />

      <label htmlFor="tags-input">Tags</label>
      <input id="tags-input" name="tags" />

      <button type="submit" disabled={isSaving}>
        Submit
      </button>
      {error ? <div role="alert">{error}</div> : null}
    </form>
  )
}
```

##  🛣️ Testing react-router-dom Routes
To test Elements from react-router-dom such as `<Link>`, `<Switch>`, and `<Route>` like in the following example:
```jsx
import * as React from 'react'
import {Switch, Route, Link} from 'react-router-dom'

const About = () => (
  <div>
    <h1>About</h1>
    <p>You are on the about page</p>
  </div>
)
const Home = () => (
  <div>
    <h1>Home</h1>
    <p>You are home</p>
  </div>
)
const NoMatch = () => (
  <div>
    <h1>404</h1>
    <p>No match</p>
  </div>
)

function Main() {
  return (
    <div>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route component={NoMatch} />
      </Switch>
    </div>
  )
}

export {Main}
```

To test it, the Main component routes, react-router-dom provides a `MemoryRouter` component, which replaces the `BrowserRouter` mocking `window.history`. 

`MemoryRouter` accepts an argument `initialEntries` that is an array that define your history stack, the first element will be the starting page of your routes
```jsx
<MemoryRouter
  initialEntries={["/one", "/two", { pathname: "/three" }]}
>
  <App />
</MemoryRouter>
```

Then the test file will look look this:

```jsx
test('main renders about and home and I can navigate to those pages', () => {
  render(
    <MemoryRouter>
      <Main />
    </MemoryRouter>,
  )
  expect(screen.getByRole('heading')).toHaveTextContent(/home/i)
  userEvent.click(screen.getByText(/about/i))
  expect(screen.getByRole('heading')).toHaveTextContent(/about/i)
})
```

### 🔙 Testing going back in a route

 To test going back in history, is it possible to make use of `useHistory` that will provide a `goBack` method, so just adding a button will do the job: 
```jsx
function Main() {
  const history = useHistory()

  return (
    <div>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <button onClick={() => history.goBack()}>Back</button>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route component={NoMatch} />
      </Switch>
    </div>
  )
}
```

The test implementation can be tricky though. It is important to know that `initialEntries` property works as a stack, the leftmost route is the oldest and the rightmost is the newest. So supposing that the initial state is `/about` and it should assert that the page goes back to `/` 
```jsx
test('navigates back', () => {
  render(
    <MemoryRouter initialEntries={['/', '/about']} initialIndex={1}>
      <Main />
    </MemoryRouter>,
  )

  expect(screen.getByRole('heading')).toHaveTextContent(/about/i)

  const backButton = screen.getByRole('button', {name: /back/i})
  userEvent.click(backButton)

  expect(screen.getByRole('heading')).toHaveTextContent(/home/i)
})
```

## ⚛️ Testing Redux

Suppose a simple redux implementation
`store.js`
```js
import {createStore} from 'redux'
import {reducer} from './redux-reducer'

const store = createStore(reducer)

export {store}
```

`reducer.js`
```js
const initialState = {count: 0}
function reducer(state = initialState, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {
        count: state.count + 1,
      }
    case 'DECREMENT':
      return {
        count: state.count - 1,
      }
    default:
      return state
  }
}

export {reducer}
```

`Counter.jsx`
```jsx
import * as React from 'react'
import {useSelector, useDispatch} from 'react-redux'

function Counter() {
  const count = useSelector((state) => state.count)
  const dispatch = useDispatch()
  const increment = () => dispatch({type: 'INCREMENT'})
  const decrement = () => dispatch({type: 'DECREMENT'})
  return (
    <div>
      <h2>Counter</h2>
      <div>
        <button onClick={decrement}>-</button>
        <span aria-label="count">{count}</span>
        <button onClick={increment}>+</button>
      </div>
    </div>
  )
}

export {Counter}
```

The test is pretty much straightforward, the only difference from a common test is that is needed to wrap the component with a `<Provider>` from `react-redux` as it is needed in the normal application around the `App` component.
```jsx
import * as React from 'react'
import {Provider} from 'react-redux'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {Counter} from '../redux-counter'
import {store} from '../redux-store'

test('can render with redux with defaults', () => {
  render(
    <Provider store={store}>
      <Counter />
    </Provider>,
  )
  userEvent.click(screen.getByText('+'))
  expect(screen.getByLabelText(/count/i)).toHaveTextContent('1')
})
```

To shorten your render code, it is possible to include the `Provider` in the custom render method created in [[Testing JavaScript#♻️ Reusing render with providers]] section
```jsx
function render(
  ui,
  {
    initialState,
    store = createStore(reducer, initialState),
    ...renderOptions
  } = {},
) {
  function Wrapper({children}) {
    return <Provider store={store}>{children}</Provider>
  }
  return {
    ...rtlRender(ui, {
      wrapper: Wrapper,
      ...renderOptions,
    }),
    // adding `store` to the returned utilities to allow us
    // to reference it in our tests (just try to avoid using
    // this to test implementation details).
    store,
  }
}
```
### 🔧 Using custom store to control the initial state
If for some reason it's necessary to create a store with some initial values, it is possible to use `createStore` in the local scope instead of using the default and pass the initialState as second argument:

```jsx
test('can render with redux with custom initial state', () => {
  const customStore = createStore(reducer, {count: 3})
  
  render(
    <Provider store={customStore}>
      <Counter />
    </Provider>,
  )
  userEvent.click(screen.getByText('-'))
  expect(screen.getByLabelText(/count/i)).toHaveTextContent('2')
})
```


## 🪝Testing custom hooks
To test hooks separated from components one possibility to achieve that is by using `act` from `@testing-library/react'` or `renderHook`. Given the following hook:
```jsx
function useCounter({initialCount = 0, step = 1} = {}) {
  const [count, setCount] = React.useState(initialCount)
  const increment = () => setCount((c) => c + step)
  const decrement = () => setCount((c) => c - step)
  return {count, increment, decrement}
}
```

One approach for test file can be written as:
```jsx
test('exposes the count and increment/decrement functions', () => {
  let result
  function TestComponent() {
    result = useCounter()
    return null
  }
  render(<TestComponent />)
  expect(result.count).toBe(0)
  act(() => result.increment())
  expect(result.count).toBe(1)
  act(() => result.decrement())
  expect(result.count).toBe(0)
})
```

Given that hooks need to be called at the top level of a function component, that workaround of create a `let result` and assign it inside  `TestComponent` were necessary, however, to emulate the process of DOM rendering and updates closer to how React works in the browser it is necessary to wrap the outside result within `act`. [see more about act](https://legacy.reactjs.org/docs/test-utils.html#act)

A modern approach can be make use of `renderHook` util from `@testing-library/react'` this simplifies the test as illustrated below:

```jsx
test('exposes the count and increment/decrement functions', async () => {
  const { result } = renderHook(useCounter);

  expect(result.current.count).toBe(0);
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
  act(() => result.current.decrement());
  expect(result.current.count).toBe(0);
});

test('rerenders hook with different props ', async () => {
  const { result, rerender } = renderHook(useCounter, { initialProps: { initialCount: 1, step: 2 }});

  expect(result.current.count).toBe(1);
  act(() => result.current.increment());
  expect(result.current.count).toBe(3);

  rerender({ step: 3 })
  act(() => result.current.decrement());
  expect(result.current.count).toBe(0);
});
```


## 🎞️ Testing scopes of the component
To narrow your test scope, instead of the default `document.body` react testing library provider `within` and `queries` methods that allows to specify from where in the dom the component will be render. Take as an example a Modal component:

```jsx
let modalRoot = document.getElementById('modal-root')
if (!modalRoot) {
  modalRoot = document.createElement('div')
  modalRoot.setAttribute('id', 'modal-root')
  document.body.appendChild(modalRoot)
}

// don't use this for your modals.
// you need to think about accessibility and styling.
// Look into: https://ui.reach.tech/dialog
function Modal({children}) {
  const el = React.useRef(document.createElement('div'))
  React.useLayoutEffect(() => {
    const currentEl = el.current
    modalRoot.appendChild(currentEl)
    return () => modalRoot.removeChild(currentEl)
  }, [])
  return ReactDOM.createPortal(children, el.current)
}

export {Modal}
```

The test can be done in the same way a common component does:
```jsx
import * as React from 'react'
import {render, within, queries} from '@testing-library/react'
import {Modal} from '../modal'

test('modal shows the children', () => {
  const { getByTestId, debug } = render(
    <Modal>
      <div data-testid="test" />
    </Modal>,
  )

  debug()
  
  expect(getByTestId('test')).toBeInTheDocument()
})
```

The debug will output will be:
```html
<body>
  <div id="modal-root">
	<div>
	  <div
		data-testid="test"
	  />
	</div>
  </div>
  <div />
</body>
```

One another approach can be using `within` from `@testing-library/react` that will return the same queries and utils from render, like `getByTestId`
```jsx
test('modal shows the children', () => {
  render(
    <Modal>
      <div data-testid="test" />
    </Modal>,
  )
  const { getByTestId } = within(document.getElementById('modal-root'))
  
  expect(getByTestId('test')).toBeInTheDocument()
})
```
> [!warning]
`within` method does not provides a `debug`  method

the test will still pass and if there's something outside of the scope of the `Modal`, it will not be retrieved:
```jsx
test('does not show anything outside the modal', () => {
  render(
    <>
	  <div data-testid="outside-modal-div" />
      <Modal>
        <div data-testid="test" />
      </Modal>
    </>
  )
  const { queryByTestId } = within(document.getElementById('modal-root'))
  
  expect(queryByTestId('outside-modal-div')).not.toBeInTheDocument()
})
```
> [!tip]
`get` prefix for queries like `getByTestId` indicates that an exception will be thrown if the element is not found, `queryByTestId` is more suitable for this kind of test.

A way to get the `<div data-testid="outside-modal-div" />` even so is to use `queries` util from `@testing-library/react` it allows you to use queries passing the desired element as the first argument like:

```jsx
import {..., queries} from '@testing-library/react'

test('does not show anything outside the modal', () => {
  ...
  expect(queries.getByTestId(document.body, 'outside-modal-div'))
})
```

## 🧹 Testing useEffect cleanup functions on Unmount
 Test unmounting effects is pretty much straightforward using `unmount` method that is returned by `render`. suppose a component with `setInterval` that requires a cleanup on it to avoid memory leaks, such as:
```jsx
function Countdown() {
  const [remainingTime, setRemainingTime] = React.useState(10000)
  const end = React.useRef(new Date().getTime() + remainingTime)
  React.useEffect(() => {
    const interval = setInterval(() => {
      const newRemainingTime = end.current - new Date().getTime()
      if (newRemainingTime <= 0) {
        clearInterval(interval)
        setRemainingTime(0)
      } else {
        setRemainingTime(newRemainingTime)
      }
    })
    return () => clearInterval(interval)
  }, [])
  return remainingTime
}
```

A strategy to cover the unmount call would be, knowing that react outputs a `console.error` message when the `clearInterval` was not set warning, check that `console.error` message below was never called.
>[!error]
>"Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in %s.%s", "a useEffect cleanup function"

A test file can be created as follows:
```jsx
beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {})
})

test('does not attempt to set state when unmounted (to prevent memory leaks)', () => {
  const {unmount} = render(<Countdown />)
  unmount()
  expect(console.error).not.toHaveBeenCalled()
})
```

At first glance it doesn't seem anything wrong, except that this test returns a false-positive assertion where `console.error` won't calls even if the cleanup function is not implemented (try comment the line `return () => clearInterval(interval)`). The issue occurs because the `setInterval` has a very long timer of 10 seconds (10000 ms) which exceeds the timeout of the test and during the time, `console.error` was not called indeed. To fix this, it is possible to use `jest.fakeTimers` as mentioned in [[Testing JavaScript#⏱️ Handling Timestamps]] and also [runOnlyPendingTimers](https://jestjs.io/docs/timer-mocks#run-pending-timers) will advance the timers to be executed immediately, making the test file look like this:
```jsx
beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation(() => {})
})

afterAll(() => {
  console.error.mockRestore()
})

afterEach(() => {
  jest.clearAllMocks()
  jest.useRealTimers()
})

test('does not attempt to set state when unmounted (to prevent memory leaks)', async () => {
  jest.useFakeTimers()
  const {unmount} = render(<Countdown />)
  unmount()
  await waitFor(() => jest.runOnlyPendingTimers())
  expect(console.error).not.toHaveBeenCalled()
})
```
Now if the cleanup function is not called, then the error message on console will be logged.

## 🌉 Integration Tests with react-testing-library

React testing library can be a tool that creates integration test, navigating through pages and testing all the expected behaviors. Take a whole app with a multi page form and a success page at the end such as:
```jsx
import * as React from 'react'
import {BrowserRouter as Router, Route, Link, Switch} from 'react-router-dom'
import {submitForm} from './api'

const MultiPageForm = React.createContext()

function MultiPageFormProvider({initialValues = {}, ...props}) {
  const [initState] = React.useState(initialValues)
  const [form, setFormValues] = React.useReducer(
    (s, a) => ({...s, ...a}),
    initState,
  )
  const resetForm = () => setFormValues(initialValues)
  return (
    <MultiPageForm.Provider
      value={{form, setFormValues, resetForm}}
      {...props}
    />
  )
}

function useMultiPageForm() {
  const context = React.useContext(MultiPageForm)
  if (!context) {
    throw new Error(
      'useMultiPageForm must be used within a MultiPageFormProvider',
    )
  }
  return context
}

function Main() {
  return (
    <>
      <h1>Welcome home</h1>
      <Link to="/page-1">Fill out the form</Link>
    </>
  )
}

function Page1({history}) {
  const {form, setFormValues} = useMultiPageForm()
  return (
    <>
      <h2>Page 1</h2>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          history.push('/page-2')
        }}
      >
        <label htmlFor="food">Favorite Food</label>
        <input
          id="food"
          value={form.food}
          onChange={(e) => setFormValues({food: e.target.value})}
        />
      </form>
      <Link to="/">Go Home</Link> | <Link to="/page-2">Next</Link>
    </>
  )
}

function Page2({history}) {
  const {form, setFormValues} = useMultiPageForm()
  return (
    <>
      <h2>Page 2</h2>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          history.push('/confirm')
        }}
      >
        <label htmlFor="drink">Favorite Drink</label>
        <input
          id="drink"
          value={form.drink}
          onChange={(e) => setFormValues({drink: e.target.value})}
        />
      </form>
      <Link to="/page-1">Go Back</Link> | <Link to="/confirm">Review</Link>
    </>
  )
}

function Confirm({history}) {
  const {form, resetForm} = useMultiPageForm()
  function handleConfirmClick() {
    submitForm(form).then(
      () => {
        resetForm()
        history.push('/success')
      },
      (error) => {
        history.push('/error', {error})
      },
    )
  }
  return (
    <>
      <h2>Confirm</h2>
      <div>
        <strong>Please confirm your choices</strong>
      </div>
      <div>
        <strong id="food-label">Favorite Food</strong>:{' '}
        <span aria-labelledby="food-label">{form.food}</span>
      </div>
      <div>
        <strong id="drink-label">Favorite Drink</strong>:{' '}
        <span aria-labelledby="drink-label">{form.drink}</span>
      </div>
      <Link to="/page-2">Go Back</Link> |{' '}
      <button onClick={handleConfirmClick}>Confirm</button>
    </>
  )
}

function Success() {
  return (
    <>
      <h2>Congrats. You did it.</h2>
      <div>
        <Link to="/">Go home</Link>
      </div>
    </>
  )
}

function Error({
  location: {
    state: {error},
  },
}) {
  return (
    <>
      <div>Oh no. There was an error.</div>
      <pre>{error.message}</pre>
      <Link to="/">Go Home</Link>
      <Link to="/confirm">Try again</Link>
    </>
  )
}

function App() {
  return (
    <MultiPageFormProvider initialValues={{food: '', drink: ''}}>
      <Router>
        <Switch>
          <Route path="/page-1" component={Page1} />
          <Route path="/page-2" component={Page2} />
          <Route path="/confirm" component={Confirm} />
          <Route path="/success" component={Success} />
          <Route path="/error" component={Error} />
          <Route component={Main} />
        </Switch>
      </Router>
    </MultiPageFormProvider>
  )
}

export default App
```

The test file is simpler than you can imagine:
```jsx
import * as React from 'react'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {submitForm as mockSubmitForm} from '../api'
import App from '../app'

jest.mock('../api')

test('Can fill out a form across multiple pages', async () => {
  mockSubmitForm.mockResolvedValueOnce({success: true})
  const testData = {food: 'test food', drink: 'test drink'}
  render(<App />)

  userEvent.click(screen.getByText(/fill.*form/i))

  userEvent.type(
    screen.getByLabelText(/food/i),
    testData.food,
  )
  
  userEvent.click(screen.getByText(/next/i))

  userEvent.type(
    screen.getByLabelText(/drink/i),
    testData.drink
  )
  
  userEvent.click(screen.getByText(/review/i))

  expect(screen.getByLabelText(/food/i)).toHaveTextContent(testData.food)
  expect(screen.getByLabelText(/drink/i)).toHaveTextContent(testData.drink)

  userEvent.click(screen.getByText(/confirm/i, {selector: 'button'}))

  expect(mockSubmitForm).toHaveBeenCalledWith(testData)
  expect(mockSubmitForm).toHaveBeenCalledTimes(1)

  userEvent.click(await screen.findByText(/home/i))

  expect(screen.getByText(/welcome home/i)).toBeInTheDocument()
})
```
>[!tip]
>To make your test more resilient, for example if some element has an animation or needs to load something, 
>replace all your queries with `getBy` prefix (e.g.: `getByLabelText`) to `await findBy` (e.g.: `await findByLabelText`) this will make your query wait until an element is found within a certain timeout

# 🤖 E2E tests with Cypress
Cypress is a frontend automation testing tool for web applications. It allows creating test files to access pages and manipulate them.

## 📦 Install
Install Cypress is simpler than it looks, just:
```shell
npm i --save-dev cypress
```

## 🎛️ Usage
To open Cypress UI dev tools access the binary at `node_modules/.bin`:
```shell
./node_modules/.bin/cypress open
```

> [!info]
> Alternatively, you can run cypress in headless mode (only on cli), by using
> ```shell
> ./node_modules/.bin/cypress run
> ```

The first page will be prompted with two options: [E2E Testing](https://docs.cypress.io/guides/end-to-end-testing/writing-your-first-end-to-end-test) and [Component Testing](https://docs.cypress.io/guides/component-testing/overview) we will look for now at E2E tests only
![12-cypress_welcome_page.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/12-cypress_welcome_page.png)

Clicking on **E2E Testing** will automatically pop up a wizard to setup your cypress environment for the first time, creating a config, additional support, and  custom commands files, as well as an example file
![13-cypress_config_page.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/13-cypress_config_page.png)
After clicking "Continue", a `cypress` folder will be created at the top level of the repository, there it is possible to find multiple folders:
- e2e: Here is where the test files will be placed. Cypress automatically detects the test files with the extension `cy.(js|ts|jsx|tsx|coffee|cjsx)` inside it 
- fixtures: pieces of static data (e.g.: json) that can be used by your tests. typically used with `cy.fixture()` when stubbing network requests
- asset: contains assets during the test run, it's recommended to be added to your `.gitignore` file
- downloads: any kind of file downloaded during the test run, it's recommended to be added to your `.gitignore` file
- screenshot: screenshots taken via `cy.screenshot`, it's recommended to be added to your `.gitignore` file
- videos: recorded videos of the run, it's recommended to be added to your `.gitignore` file
- support: To include code before your test files, you can have a file `cypress/support/e2e.(js|ts|jsx|tsx)` for e2e tests or `cypress/support/component.(js|ts|jsx|tsx)`

>[!tip]
>For TypeScript, you will need to create a `cypress/tsconfig.json` file
>```json
>{
>  "compilerOptions": {
>    "target": "es5",
>    "lib": ["es5", "dom"],
>    "types": ["cypress", "node"]
>  },
>  "include": ["**/*.ts"]
>}
  

After this step you will be required to choose a browser to navigate through your tests. the list will correspond the browsers you have installed in your machine, except for specific cases that you want to check the behavior in a specific browser, it's recommended to use Electron for a minimal browser environment.
![14-cypress_browser_selector.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/14-cypress_browser_selector.png))
Then it's possible to play around with multiple examples auto generated by cypress or create a new empty test file. Let's start creating a new spec named `cypress/e2e/my_first_test.cy.js`.

![15-cypress_start_page.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/15-cypress_start_page.png)
## Writing a test

Cypress will then create a template for your test:
```js
describe('template spec', () => {
    it('passes', () => {
        cy.visit('https://example.cypress.io')
    })
})
```

And you can test if everything is working well by clicking in "Okay, run the spec"
![16-cypress_spec_template.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/16-cypress_spec_template.png)
After some seconds, cypress will run the test files and will display the report
![17-cypress_report.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/17-cypress_report.png)
You will notice that cypress contains a whole browser inside it and a very powerful and helpful tool is that "aim" icon at the left of the url. This tool can retrieve queries from the selected element, for example, clicking on it and selecting the documentation for **[get](https://example.cypress.io/commands/querying)** will show the exact command to query it:
```js
cy.get(':nth-child(4) > .row > .col-xs-12 > .home-list > :nth-child(1) > ul > :nth-child(1) > a')
```

`ci` object is chainable, it means that you can run sequential commands, for example, you can assert that the previously selected element should contains the text **get**
```js
cy.get(":nth-child(4) > .row > .col-xs-12 > .home-list > :nth-child(1) > ul > :nth-child(1) > a")
  .should("have.text", "get")
```

So, a simple test case for this page can be written as:
```js
describe("template spec", () => {
  it("passes", () => {
    cy.visit("https://example.cypress.io");
    
    cy.get(":nth-child(4) > .row > .col-xs-12 > .home-list > :nth-child(1) > ul > :nth-child(1) > a")
      .should("have.text", "get")
      .click();
      
    cy.url().should("include", "/commands/querying");
    
    cy.get('#get > a').should("have.text", "cy.get()");
  });
});
```

This test will assert that we got the **get** link properly, navigates to the corresponding page and contains the corresponding section
>[!tip]
>[.should()](https://on.cypress.io/should) assertion is considered an implicit assertion, it means that it is dependent of the parent element to make an assertion, so it should be always used chained after some query.
>
>[.and()](https://on.cypress.io/and) is also another implicit assertion that allows you to assert multiple statements
>
>[expect]() is an explicit assertion, so it means that it is executed outside the query, like: `expect(true).to.be.true`

## Configuring cypress.config.js
`cypress.config.js` file located at the root of your project allows you to customize your tests, for example, it is possible to add a `baseUrl` to define your main page:
```js
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "https://example.cypress.io",
  },
});
```

Then it is possible to replace 
```js
cy.visit("https://example.cypress.io");
```
to
```js
cy.visit("/");
```

there is a whole set of configurations that can be checked in the docs at [Configuration](https://docs.cypress.io/guides/references/configuration) section, for example setting custom timeouts for your pageLoad, requests, responses, and etc.
## Cypress Testing Library
Cypress testing library adds some aditional queries to your cypress, for example it allows you to replace
```js
cy.get(":nth-child(4) > .row > .col-xs-12 > .home-list > :nth-child(1) > ul > :nth-child(1) > a")
```
to
```js
cy.findByText(/get/i)
```

### Install
```shell
npm i --save-dev @testing-library/cypress
```

And you will need to extend the default cypress commands by adding at your `cypress/support/commands.js`
```js
import '@testing-library/cypress/add-commands'
```

>[!tip]
>For TypeScript, you will need to add `"@testing-library/cypress"` at your `compilerOptions.types` array
>```js
>{
>  "compilerOptions": {
>  ...
>  "types": [
>    ...
>    "@testing-library/cypress"
>  ],
}
>```

>[!warning]
> use findBy for queries. getBy queries are not supported because for reasonable Cypress tests you need retryability and findBy queries already support that. queryBy queries are no longer necessary since v5 and will be removed in v6. findBy fully support built-in Cypress assertions (removes the only use-case for queryBy).

## Cypress in local development
Suppose you have a vite-react application that is running in your localhost at port 8080 environment and you want to test it locally instead of need to deploy it first. To do it you can use [start-server-and-test](https://www.npmjs.com/package/start-server-and-test) and create the following script at your `package.json`

```json
{
  "dev": "vite",
  "cy:open": "cypress open",
  "test:e2e:dev": "start-server-and-test dev http://localhost:8080 cy:open",
  ...
}
```

and then simply run `npm run test:e2e:dev`

## Debug
Given the chainable nature of cypress commands, and its async behavior, you can make use of the regular `then` from Promises, such as:
```js
cy.findByText("get")
  .then((subject) => {
	debugger;
	return subject;
  })
  .should("have.text", "get")
  .click();
```
There is a debugger statement that will pause your test run and log some useful information like `subject` that is a JQuery object containing information about the node in the context.

![18-cypress_debug.png](Testing%20JavaScript%20a47d046ff7444190bcb90e0a7d81b5f8/18-cypress_debug.png)

Alternatively you can achieve the same result using `.debug()` to pause and log details of the test, or just `.pause()` that is self explanatory.

## Testing forms with fake data
# 📚 References

1. [Testing JavaScript with Kent C. Dodds](https://testingjavascript.com/)
2. [Mock Functions](https://jestjs.io/docs/mock-functions)
3. [Jest Object (jest.spyOn)](https://jestjs.io/docs/jest-object#jestspyonobject-methodname)
4. [ESLint](https://eslint.org/)
5. [Prettier](https://prettier.io/)
6. [TypeScript](https://www.typescriptlang.org/)
7. [Husky](https://typicode.github.io/husky/)