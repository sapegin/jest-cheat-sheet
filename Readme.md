# Jest cheat sheet

## Basic test

```js
describe('makePoniesPink', () => {
  test('make each pony pink', () => {
    const actual = fn(['Alice', 'Bob', 'Eve']);
    expect(actual).toEqual(['Pink Alice', 'Pink Bob', 'Pink Eve']);
  });
});
```

## Matchers

```js
expect.assertions(28) // Exactly 28 assertions are called during a test
expect.hasAssertions() // At least one assertion is called during a test

expect(42).toBe(42) // ===
expect(42).not.toBe(3) // !==
expect([1, 2]).toEqual([1, 2]) // Deep equality
expect('').toBeFalsy() // false, 0, '', null, undefined, NaN
expect('foo').toBeTruthy() // Not false, 0, '', null, undefined, NaN
expect(null).toBeNull() // === null
expect(undefined).toBeUndefined() // === undefined
expect(7).toBeDefined() // !== undefined

expect('long string').toMatch('str')
expect('coffee').toMatch(/ff/)
expect('pizza').not.toMatch('coffee')
expect(['pizza', 'coffee']).toEqual([expect.stringContaining('zz'), expect.stringMatching(/ff/)])

expect(new A()).toBeInstanceOf(A)
expect(() => {}).toEqual(expect.any(Function))
expect('pizza').toEqual(expect.anything())

expect({a: 1}).toHaveProperty('a')
expect({a: 1}).toHaveProperty('a', 1)
expect({a: {b: 1}}).toHaveProperty('a.b')
expect({a: 1, b: 2}).toMatchObject({a: 1})
expect({a: 1, b: 2}).toMatchObject({a: expect.any(Number), b: expect.any(Number)})
expect([{a: 1}, {b: 2}]).toEqual([expect.objectContaining({a: expect.any(Number)}), expect.anything()])

expect(2).toBeGreaterThan(1)
expect(1).toBeGreaterThanOrEqual(1)
expect(1).toBeLessThan(2)
expect(1).toBeLessThanOrEqual(1)
expect(0.2 + 0.1).toBeCloseTo(0.3, 5)

expect(['Alice', 'Bob', 'Eve']).toHaveLength(3)
expect(['Alice', 'Bob', 'Eve']).toContain('Alice')
expect([{a: 1}, {a: 2}]).toContainEqual({a: 1})
expect(['Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(['Alice', 'Bob']))

// const fn = () => { throw new Error('Out of cheese!') }
expect(fn).toThrow()
expect(fn).toThrow('Out of cheese')
expect(fn).toThrowErrorMatchingSnapshot()

expect(node).toMatchSnapshot()

// const fn = jest.fn()
fn.mockClear() // Clear number of calls
expect(fn).toBeCalled() // Function was called
expect(fn).not.toBeCalled() // Function was *not* called
expect(fn).toHaveBeenCalledTimes(1) // Function was called only once
expect(fn).toBeCalledWith('first arg', 'second arg') // Any of calls was with these arguments
expect(fn).toHaveBeenLastCalledWith('first arg', 'secon arg') // Last call was with these arguments
expect(fn.mock.calls).toEqual([['first', 'call', 'args'], ['second', 'call', 'args']]) // Multiple calls
expect(fn.mock.calls[0][0](1)).toBe(2) // fn.mock.calls[0][0] — the first argument of the first call
```

[Matchers docs](https://facebook.github.io/jest/docs/expect.html)

### Aliases

* `toBeCalled` → `toHaveBeenCalled`
* `toBeCalledWith` → `toHaveBeenCalledWith`
* `lastCalledWith` → `toHaveBeenLastCalledWith`
* `toThrowError` → `toThrow`

### Promise matchers (Jest 20+)

```js
test('resolve to lemon', () => {
  expect.assertions(1);
  // Make sure to add a return statement
  return expect(Promise.resolve('lemon')).resolves.toBe('lemon');
  // return expect(Promise.reject('octopus')).rejects.toBeDefined();
});
```

Or with async/await:

```js
test('resolve to lemon', async () => {
  expect.assertions(2);
  await expect(Promise.resolve('lemon')).resolves.toBe('lemon');
  await expect(Promise.resolve('lemon')).resolves.not.toBe('octopus');
});
```

[resolves docs](https://facebook.github.io/jest/docs/en/expect.html#resolves)

## Mocks

### Mock functions

```js
test('call the callback', () => {
  const callback = jest.fn();
  fn(callback);
  expect(callback).toBeCalled();
  expect(callback.mock.calls[0][1].baz).toBe('pizza'); // Second argument of the first call
});
```

You can also pass an implementation to `jest.fn` function:

```js
const callback = jest.fn(() => true);
```

[Mock functions docs](https://facebook.github.io/jest/docs/mock-function-api.html)

### Mock modules using `jest.mock` method

```js
jest.mock('lodash/memoize', () => a => a); // The original lodash/memoize should exist
jest.mock('lodash/memoize', () => a => a, { virtual: true });  // The original lodash/memoize isn’t required
```

[jest.mock docs](https://facebook.github.io/jest/docs/jest-object.html#jestmockmodulename-factory-options)

> Note: When using `babel-jest`, calls to `jest.mock` will automatically be hoisted to the top of the code block. Use `jest.doMock` if you want to explicitly avoid this behavior.

### Mock modules using a mock file

1. Create a file like `__mocks__/lodash/memoize.js`:

   ```js
   module.exports = a => a;
   ```

2. Add to your test:

   ```js
   jest.mock('lodash/memoize');
   ```

> Note: When using `babel-jest`, calls to `jest.mock` will automatically be hoisted to the top of the code block. Use `jest.doMock` if you want to explicitly avoid this behavior.

[Manual mocks docs](https://facebook.github.io/jest/docs/manual-mocks.html)

### Accessing the original module when using mocks

```js
jest.mock('fs')
const fs = require('fs') // Mocked module
const fs = require.requireActual('fs') // Original module
```

## Resources

* [Jest site](https://facebook.github.io/jest/)
* [Testing React components with Jest and Enzyme](http://blog.sapegin.me/all/react-jest) by Artem Sapegin
* [Testing React Applications](https://youtu.be/59Ndb3YkLKA) by Max Stoiber
* [Migrating to Jest](https://medium.com/@kentcdodds/migrating-to-jest-881f75366e7e#.pc4s5ut6z) by Kent C. Dodds
* [Migrating AVA to Jest](http://browniefed.com/blog/migrating-ava-to-jest/) by Jason Brown
* [How to Test React and MobX with Jest](https://semaphoreci.com/community/tutorials/how-to-test-react-and-mobx-with-jest)
* [Testing with Jest: 15 Awesome Tips and Tricks](https://medium.com/@stipsan/testing-with-jest-15-awesome-tips-and-tricks-42150ec4c262) by Stian Didriksen
* Taking Advantage of Jest Matchers by Ben McCormick: [Part 1](https://benmccormick.org/2017/08/15/jest-matchers-1/), [Part 2](https://benmccormick.org/2017/09/04/jest-matchers-2/)

***

## You may also like

* [Opinionated list of React components](https://github.com/sapegin/react-components)

## Contributing

Improvements are welcome! Open an issue or send a pull request.

## Author and license

[Artem Sapegin](http://sapegin.me/), a frontend developer at [Here](https://here.com/en) and the creator of [React Styleguidist](https://github.com/styleguidist/react-styleguidist). I also write about frontend at [my blog](http://blog.sapegin.me/).

CC0 1.0 Universal license, see the included [License.md](/License.md) file.
