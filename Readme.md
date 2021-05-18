<div align="center" markdown="1">

<img src="https://github.com/facebook/jest/raw/master/website/static/img/jest.png" width="200">

<h1>Jest cheat sheet</h1>

</div>

_I recommend [Mrm](https://github.com/sapegin/mrm/tree/master/packages/mrm-task-jest) and [jest-codemods](https://github.com/skovhus/jest-codemods) for single-command Jest installation and easy migration from other frameworks._

<!-- To reformat run: npx prettier --print-width 100 --single-quote --no-semi --prose-wrap never --write Readme.md -->

<!-- To update TOC run: npx markdown-toc --maxdepth 3 -i Readme.md -->

<!-- toc -->

- [Test structure](#test-structure)
- [Matchers](#matchers)
  - [Basic matchers](#basic-matchers)
  - [Truthiness](#truthiness)
  - [Numbers](#numbers)
  - [Strings](#strings)
  - [Arrays](#arrays)
  - [Objects](#objects)
  - [Exceptions](#exceptions)
  - [Snapshots](#snapshots)
  - [Mock functions](#mock-functions)
  - [Misc](#misc)
  - [Promise matchers (Jest 20+)](#promise-matchers-jest-20)
- [Async tests](#async-tests)
  - [async/await](#asyncawait)
  - [Promises](#promises)
  - [done() callback](#done-callback)
- [Mocks](#mocks)
  - [Mock functions](#mock-functions-1)
  - [Returning, resolving and rejecting values](#returning-resolving-and-rejecting-values)
  - [Mock modules using `jest.mock` method](#mock-modules-using-jestmock-method)
  - [Mock modules using a mock file](#mock-modules-using-a-mock-file)
  - [Mock object methods](#mock-object-methods)
  - [Mock getters and setters (Jest 22.1.0+)](#mock-getters-and-setters-jest-2210)
  - [Mock getters and setters](#mock-getters-and-setters)
  - [Clearing and restoring mocks](#clearing-and-restoring-mocks)
  - [Accessing the original module when using mocks](#accessing-the-original-module-when-using-mocks)
  - [Timer mocks](#timer-mocks)
- [Data-driven tests (Jest 23+)](#data-driven-tests-jest-23)
- [Skipping tests](#skipping-tests)
- [Testing modules with side effects](#testing-modules-with-side-effects)
- [Usage with Babel and TypeScript](#usage-with-babel-and-typescript)
- [Resources](#resources)
- [You may also like](#you-may-also-like)
- [Contributing](#contributing)
- [Sponsoring](#sponsoring)
- [Author and license](#author-and-license)

<!-- tocstop -->

## Test structure

```js
describe('makePoniesPink', () => {
  beforeAll(() => {
    /* Runs before all tests */
  })
  afterAll(() => {
    /* Runs after all tests */
  })
  beforeEach(() => {
    /* Runs before each test */
  })
  afterEach(() => {
    /* Runs after each test */
  })

  test('make each pony pink', () => {
    const actual = fn(['Alice', 'Bob', 'Eve'])
    expect(actual).toEqual(['Pink Alice', 'Pink Bob', 'Pink Eve'])
  })
})
```

## Matchers

[Using matchers](http://jestjs.io/docs/en/using-matchers), [matchers docs](https://jestjs.io/docs/en/expect)

### Basic matchers

```js
expect(42).toBe(42) // Strict equality (===)
expect(42).not.toBe(3) // Strict equality (!==)
expect([1, 2]).toEqual([1, 2]) // Deep equality
expect({ a: undefined, b: 2 }).toEqual({ b: 2 }) // Deep equality
expect({ a: undefined, b: 2 }).not.toStrictEqual({ b: 2 }) // Strict equality (Jest 23+)
```

### Truthiness

```js
// Matches anything that an if statement treats as true (not false, 0, '', null, undefined, NaN)
expect('foo').toBeTruthy()
// Matches anything that an if statement treats as false (false, 0, '', null, undefined, NaN)
expect('').toBeFalsy()
// Matches only null
expect(null).toBeNull()
// Matches only undefined
expect(undefined).toBeUndefined()
// The opposite of toBeUndefined
expect(7).toBeDefined()
// Matches true or false
expect(true).toEqual(expect.any(Boolean))
```

### Numbers

```js
expect(2).toBeGreaterThan(1)
expect(1).toBeGreaterThanOrEqual(1)
expect(1).toBeLessThan(2)
expect(1).toBeLessThanOrEqual(1)
expect(0.2 + 0.1).toBeCloseTo(0.3, 5)
expect(NaN).toEqual(expect.any(Number))
```

### Strings

```js
expect('long string').toMatch('str')
expect('string').toEqual(expect.any(String))
expect('coffee').toMatch(/ff/)
expect('pizza').not.toMatch('coffee')
expect(['pizza', 'coffee']).toEqual([expect.stringContaining('zz'), expect.stringMatching(/ff/)])
```

### Arrays

```js
expect([]).toEqual(expect.any(Array))
expect(['Alice', 'Bob', 'Eve']).toHaveLength(3)
expect(['Alice', 'Bob', 'Eve']).toContain('Alice')
expect([{ a: 1 }, { a: 2 }]).toContainEqual({ a: 1 })
expect(['Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(['Alice', 'Bob']))
```

### Objects

```js
expect({ a: 1 }).toHaveProperty('a')
expect({ a: 1 }).toHaveProperty('a', 1)
expect({ a: { b: 1 } }).toHaveProperty('a.b')
expect({ a: 1, b: 2 }).toMatchObject({ a: 1 })
expect({ a: 1, b: 2 }).toMatchObject({
  a: expect.any(Number),
  b: expect.any(Number),
})
expect([{ a: 1 }, { b: 2 }]).toEqual([
  expect.objectContaining({ a: expect.any(Number) }),
  expect.anything(),
])
```

### Exceptions

```js
// const fn = () => { throw new Error('Out of cheese!') }
expect(fn).toThrow()
expect(fn).toThrow('Out of cheese')
expect(fn).toThrowErrorMatchingSnapshot()
```

<details>
  <summary>Aliases</summary>

- `toThrowError` → `toThrow`
  </details>

### Snapshots

```js
expect(node).toMatchSnapshot()
// Jest 23+
expect(user).toMatchSnapshot({
  date: expect.any(Date),
})
expect(user).toMatchInlineSnapshot()
```

### Mock functions

```js
// const fn = jest.fn()
// const fn = jest.fn().mockName('Unicorn') -- named mock, Jest 22+
expect(fn).toBeCalled() // Function was called
expect(fn).not.toBeCalled() // Function was *not* called
expect(fn).toHaveBeenCalledTimes(1) // Function was called only once
expect(fn).toBeCalledWith(arg1, arg2) // Any of calls was with these arguments
expect(fn).toHaveBeenLastCalledWith(arg1, arg2) // Last call was with these arguments
expect(fn).toHaveBeenNthCalledWith(callNumber, args) // Nth call was with these arguments (Jest 23+)
expect(fn).toHaveReturnedTimes(2) // Function was returned without throwing an error (Jest 23+)
expect(fn).toHaveReturnedWith(value) // Function returned a value (Jest 23+)
expect(fn).toHaveLastReturnedWith(value) // Last function call returned a value (Jest 23+)
expect(fn).toHaveNthReturnedWith(value) // Nth function call returned a value (Jest 23+)
expect(fn.mock.calls).toEqual([
  ['first', 'call', 'args'],
  ['second', 'call', 'args'],
]) // Multiple calls
expect(fn.mock.calls[0][0]).toBe(2) // fn.mock.calls[0][0] — the first argument of the first call
```

<details>
  <summary>Aliases</summary>

- `toBeCalled` → `toHaveBeenCalled`
- `toBeCalledWith` → `toHaveBeenCalledWith`
- `lastCalledWith` → `toHaveBeenLastCalledWith`
- `nthCalledWith` → `toHaveBeenNthCalledWith`
- `toReturnTimes` → `toHaveReturnedTimes`
- `toReturnWith` → `toHaveReturnedWith`
- `lastReturnedWith` → `toHaveLastReturnedWith`
- `nthReturnedWith` → `toHaveNthReturnedWith`
  </details>

### Misc

```js
expect(new A()).toBeInstanceOf(A)
expect(() => {}).toEqual(expect.any(Function))
expect('pizza').toEqual(expect.anything())
```

### Promise matchers (Jest 20+)

```js
test('resolve to lemon', () => {
  expect.assertions(1)
  // Make sure to add a return statement
  return expect(Promise.resolve('lemon')).resolves.toBe('lemon')
  return expect(Promise.reject('octopus')).rejects.toBeDefined()
  return expect(Promise.reject(Error('pizza'))).rejects.toThrow()
})
```

Or with async/await:

```js
test('resolve to lemon', async () => {
  expect.assertions(2)
  await expect(Promise.resolve('lemon')).resolves.toBe('lemon')
  await expect(Promise.resolve('lemon')).resolves.not.toBe('octopus')
})
```

[resolves docs](https://jestjs.io/docs/en/expect#resolves)

## Async tests

See [more examples](https://jestjs.io/docs/en/tutorial-async) in Jest docs.

It’s a good practice to specify a number of expected assertions in async tests, so the test will fail if your assertions weren’t called at all.

```js
test('async test', () => {
  expect.assertions(3) // Exactly three assertions are called during a test
  // OR
  expect.hasAssertions() // At least one assertion is called during a test

  // Your async tests
})
```

Note that you can also do this per file, outside any `describe` and `test`:

```js
beforeEach(expect.hasAssertions)
```

This will verify the presense of at least one assertion per test case. It also plays nice with more specific `expect.assertions(3)` declarations.

### async/await

```js
test('async test', async () => {
  expect.assertions(1)
  const result = await runAsyncOperation()
  expect(result).toBe(true)
})
```

### Promises

_Return_ a Promise from your test:

```js
test('async test', () => {
  expect.assertions(1)
  return runAsyncOperation().then((result) => {
    expect(result).toBe(true)
  })
})
```

### done() callback

Wrap your assertions in try/catch block, otherwise Jest will ignore failures:

```js
test('async test', (done) => {
  expect.assertions(1)
  runAsyncOperation()
  setTimeout(() => {
    try {
      const result = getAsyncOperationResult()
      expect(result).toBe(true)
      done()
    } catch (err) {
      done.fail(err)
    }
  })
})
```

## Mocks

### Mock functions

```js
test('call the callback', () => {
  const callback = jest.fn()
  fn(callback)
  expect(callback).toBeCalled()
  expect(callback.mock.calls[0][1].baz).toBe('pizza') // Second argument of the first call
  // Match the first and the last arguments but ignore the second argument
  expect(callback).toHaveBeenLastCalledWith('meal', expect.anything(), 'margarita')
})
```

You can also use snapshots:

```js
test('call the callback', () => {
  const callback = jest.fn().mockName('Unicorn') // mockName is available in Jest 22+
  fn(callback)
  expect(callback).toMatchSnapshot()
  // ->
  // [MockFunction Unicorn] {
  //   "calls": Array [
  // ...
})
```

And pass an implementation to `jest.fn` function:

```js
const callback = jest.fn(() => true)
```

[Mock functions docs](https://jestjs.io/docs/en/mock-function-api)

### Returning, resolving and rejecting values

Your mocks can return values:

```js
const callback = jest.fn().mockReturnValue(true)
const callbackOnce = jest.fn().mockReturnValueOnce(true)
```

Or resolve values:

```js
const promise = jest.fn().mockResolvedValue(true)
const promiseOnce = jest.fn().mockResolvedValueOnce(true)
```

They can even reject values:

```js
const failedPromise = jest.fn().mockRejectedValue('Error')
const failedPromiseOnce = jest.fn().mockRejectedValueOnce('Error')
```

You can even combine these:

```js
const callback = jest.fn().mockReturnValueOnce(false).mockReturnValue(true)

// ->
//  call 1: false
//  call 2+: true
```

### Mock modules using `jest.mock` method

```js
jest.mock('lodash/memoize', () => (a) => a) // The original lodash/memoize should exist
jest.mock('lodash/memoize', () => (a) => a, { virtual: true }) // The original lodash/memoize isn’t required
```

[jest.mock docs](https://jestjs.io/docs/en/jest-object#jestmockmodulename-factory-options)

> Note: When using `babel-jest`, calls to `jest.mock` will automatically be hoisted to the top of the code block. Use `jest.doMock` if you want to explicitly avoid this behavior.

### Mock modules using a mock file

1.  Create a file like `__mocks__/lodash/memoize.js`:

    ```js
    module.exports = (a) => a
    ```

2.  Add to your test:

    ```js
    jest.mock('lodash/memoize')
    ```

> Note: When using `babel-jest`, calls to `jest.mock` will automatically be hoisted to the top of the code block. Use `jest.doMock` if you want to explicitly avoid this behavior.

[Manual mocks docs](https://jestjs.io/docs/en/manual-mocks)

### Mock object methods

```js
const spy = jest.spyOn(console, 'log').mockImplementation(() => {})
expect(console.log.mock.calls).toEqual([['dope'], ['nope']])
spy.mockRestore()
```

```js
const spy = jest.spyOn(ajax, 'request').mockImplementation(() => Promise.resolve({ success: true }))
expect(spy).toHaveBeenCalled()
spy.mockRestore()
```

### Mock getters and setters (Jest 22.1.0+)

```js
const location = {}
const getTitle = jest.spyOn(location, 'title', 'get').mockImplementation(() => 'pizza')
const setTitle = jest.spyOn(location, 'title', 'set').mockImplementation(() => {})
```

### Mock getters and setters

```js
const getTitle = jest.fn(() => 'pizza')
const setTitle = jest.fn()
const location = {}
Object.defineProperty(location, 'title', {
  get: getTitle,
  set: setTitle,
})
```

### Clearing and restoring mocks

For one mock:

```js
fn.mockClear() // Clears mock usage date (fn.mock.calls, fn.mock.instances)
fn.mockReset() // Clears and removes any mocked return values or implementations
fn.mockRestore() // Resets and restores the initial implementation
```

> Note: `mockRestore` works only with mocks created by `jest.spyOn`.

For all mocks:

```js
jest.clearAllMocks()
jest.resetAllMocks()
jest.restoreAllMocks()
```

### Accessing the original module when using mocks

```js
jest.mock('fs')
const fs = require('fs') // Mocked module
const fs = require.requireActual('fs') // Original module
```

### Timer mocks

Write synchronous test for code that uses native timer functions (`setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`).

```js
// Enable fake timers
jest.useFakeTimers()

test('kill the time', () => {
  const callback = jest.fn()

  // Run some code that uses setTimeout or setInterval
  const actual = someFunctionThatUseTimers(callback)

  // Fast-forward until all timers have been executed
  jest.runAllTimers()

  // Check the results synchronously
  expect(callback).toHaveBeenCalledTimes(1)
})
```

Or adjust timers by time with [advanceTimersByTime()](https://jestjs.io/docs/en/timer-mocks#advance-timers-by-time):

```js
// Enable fake timers
jest.useFakeTimers()

test('kill the time', () => {
  const callback = jest.fn()

  // Run some code that uses setTimeout or setInterval
  const actual = someFunctionThatUseTimers(callback)

  // Fast-forward for 250 ms
  jest.advanceTimersByTime(250)

  // Check the results synchronously
  expect(callback).toHaveBeenCalledTimes(1)
})
```

Use [jest.runOnlyPendingTimers()](https://jestjs.io/docs/en/timer-mocks#run-pending-timers) for special cases.

**Note:** you should call `jest.useFakeTimers()` in your test case to use other fake timer methods.

## Data-driven tests (Jest 23+)

Run the same test with different data:

```js
test.each([
  [1, 1, 2],
  [1, 2, 3],
  [2, 1, 3],
])('.add(%s, %s)', (a, b, expected) => {
  expect(a + b).toBe(expected)
})
```

Or the same using template literals:

```js
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${1} | ${2} | ${3}
  ${2} | ${1} | ${3}
`('returns $expected when $a is added $b', ({ a, b, expected }) => {
  expect(a + b).toBe(expected)
})
```

Or on `describe` level:

```js
describe.each([['mobile'], ['tablet'], ['desktop']])('checkout flow on %s', (viewport) => {
  test('displays success page', () => {
    //
  })
})
```

[describe.each() docs](https://jestjs.io/docs/en/api#describeeachtablename-fn-timeout), [test.each() docs](https://jestjs.io/docs/en/api#testeachtablename-fn-timeout),

## Skipping tests

Don’t run these tests:

```js
describe.skip('makePoniesPink'...
tests.skip('make each pony pink'...
```

Run only these tests:

```js
describe.only('makePoniesPink'...
tests.only('make each pony pink'...
```

## Testing modules with side effects

Node.js and Jest will cache modules you `require`. To test modules with side effects you’ll need to reset the module registry between tests:

```js
const modulePath = '../module-to-test'

afterEach(() => {
  jest.resetModules()
})

test('first test', () => {
  // Prepare conditions for the first test
  const result = require(modulePath)
  expect(result).toMatchSnapshot()
})

test('second text', () => {
  // Prepare conditions for the second test
  const fn = () => require(modulePath)
  expect(fn).toThrow()
})
```

## Usage with Babel and TypeScript

Add [babel-jest](https://github.com/facebook/jest/tree/master/packages/babel-jest) or [ts-jest](https://github.com/kulshekhar/ts-jest). Check their docs for installation instructions.

## Resources

- [Jest site](https://facebook.github.io/jest/)
- [Modern React testing, part 1: best practices](https://blog.sapegin.me/all/react-testing-1-best-practices/) by Artem Sapegin
- [Modern React testing, part 2: Jest and Enzyme](https://blog.sapegin.me/all/react-testing-2-jest-and-enzyme/) by Artem Sapegin
- [Modern React testing, part 3: Jest and React Testing Library](https://blog.sapegin.me/all/react-testing-3-jest-and-react-testing-library/) by Artem Sapegin
- [React Testing Examples](https://react-testing-examples.com/)
- [Testing React Applications](https://youtu.be/59Ndb3YkLKA) by Max Stoiber
- [Effective Snapshot Testing](https://blog.kentcdodds.com/effective-snapshot-testing-e0d1a2c28eca) by Kent C. Dodds
- [Migrating to Jest](https://medium.com/@kentcdodds/migrating-to-jest-881f75366e7e#.pc4s5ut6z) by Kent C. Dodds
- [Migrating AVA to Jest](http://browniefed.com/blog/migrating-ava-to-jest/) by Jason Brown
- [How to Test React and MobX with Jest](https://semaphoreci.com/community/tutorials/how-to-test-react-and-mobx-with-jest) by Will Stern
- [Testing React Intl components with Jest and Enzyme](https://medium.com/@sapegin/testing-react-intl-components-with-jest-and-enzyme-f9d43d9c923e) by Artem Sapegin
- [Testing with Jest: 15 Awesome Tips and Tricks](https://medium.com/@stipsan/testing-with-jest-15-awesome-tips-and-tricks-42150ec4c262) by Stian Didriksen
- Taking Advantage of Jest Matchers by Ben McCormick: [Part 1](https://benmccormick.org/2017/08/15/jest-matchers-1/), [Part 2](https://benmccormick.org/2017/09/04/jest-matchers-2/)

---

## You may also like

- [Opinionated list of React components](https://github.com/sapegin/react-components)

## Contributing

Improvements are welcome! Open an issue or send a pull request.

## Sponsoring

This software has been developed with lots of coffee, buy me one more cup to keep it going.

<a href="https://www.buymeacoffee.com/sapegin" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/lato-orange.png" alt="Buy Me A Coffee" height="51" width="217" ></a>

## Author and license

[Artem Sapegin](http://sapegin.me/), a frontend engineer at [Omio](https://omio.com/) and the creator of [React Styleguidist](https://react-styleguidist.js.org/). I also write about frontend at [my blog](https://blog.sapegin.me/).

CC0 1.0 Universal license, see the included [License.md](/License.md) file.
