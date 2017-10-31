# await-timeout

[![Build Status](https://travis-ci.org/vitalets/await-timeout.svg?branch=master)](https://travis-ci.org/vitalets/await-timeout)
[![npm](https://img.shields.io/npm/v/await-timeout.svg)](https://www.npmjs.com/package/await-timeout)
[![license](https://img.shields.io/npm/l/await-timeout.svg)](https://www.npmjs.com/package/await-timeout)

A [Promise]-based implementation of setTimeout / clearTimeout.

* [Installation](#installation)
* [Usage](#usage)
* [API](#api)
  * [new Timeout()](#api)
  * [.set()](#api)
  * [.clear()](#api)
* [Motivation](#motivation)
* [Related resources](#related-resources)
* [License](#license)

## Installation
```bash
npm install await-timeout --save
```

## Usage
Example of fetching resource with timeout (using ES7 [async / await]): 
```js
import Timeout from 'await-timeout';

async function foo() {
  const timeout = new Timeout();
  try {
    const mainPromise = fetch('https://example.com');
    const timerPromise = timeout.set(1000, 'Rejected by timeout');
    await Promise.race([mainPromise, timerPromise]);
  } catch (e) {
    console.error(e);
  } finally {
    timeout.clear();
  }
}
```
The same example with Promise chain:
```js
function foo() {
  const timeout = new Timeout();
  return Promise.race([
    fetch('https://example.com'), 
    timeout.set(1000, 'Rejected by timeout')
  ])
  .then(result => {
    timeout.clear();
    return result;
  })
  .catch(e => {
    timeout.clear();
    console.error(e);
  });
}
```

## API
### new Timeout()
Constructs new timeout instance. It does not start timer but creates variable for timer manipulation.
```js
const timeout = new Timeout();
```
> Note: having separate variable is useful for clearing timeout in `finally` block 

### timeout.set(ms, [message]) ⇒ `Promise`
Starts new timer like `setTimeout()` and returns promise. The promise will be resolved after `ms`:
```js
timeout.set(1000)
  .then(() => console.log('1000 ms passed.'));
```
If you need to reject after timeout:
```js
timeout.set(1000)
  .then(() => {throw new Error('Timeout')});
```
Or reject with custom error:
```js
timeout.set(1000)
  .then(() => {throw new MyTimeoutError()});
```
The second parameter `message` is just convenient way to reject with `new Error(message)`:
```js
timeout.set(1000, 'Timeout');
// the same as
timeout.set(1000).then(() => {throw new Error('Timeout')});
```

### timeout.clear()
Clears existing timeout like `clearTimeout()`.
```js
const timeout = new Timeout();
timeout.set(1000)
  .then(() => console.log('This will never be called!'));
timeout.clear();
```
With [async / await] `.clear()` can be used in `finally` block:
```js
async function foo() {
  const timeout = new Timeout();
  try {
    // some async stuff
  } finally {
    timeout.clear();
  }
}
```

## Motivation
Before making this library I've researched [many similar packages on Npm](https://www.npmjs.com/search?q=promise%20timeout).
But no one satisfied all my needs together:

1. Convenient way to cancel timeout. I typically use it with [Promise.race()] and don't want timer to trigger
   if main promise is fulfilled first.
2. API similar to `setTimeout` / `clearTimeout`. I get used to these functions and would like to have mirror syntax.
3. Easy rejection of timeout promise. Passing error message should be enough.
4. No monkey-patching of Promise object.
5. Zero dependencies.

## Related resources
* https://italonascimento.github.io/applying-a-timeout-to-your-promises/
* https://stackoverflow.com/questions/22707475/how-to-make-a-promise-from-settimeout

## License
MIT @ [Vitaliy Potapov](https://github.com/vitalets)

[Promise]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise
[Promise.race()]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race
[async / await]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function