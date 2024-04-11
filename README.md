Base Ts Result
===========
Better error handling stolen from rust

<!-- shields -->

![npm](https://img.shields.io/npm/v/base-ts-result)
![npm](https://img.shields.io/npm/dm/base-ts-result)
![NPM](https://img.shields.io/npm/l/base-ts-result)

* [Install](#Install)
* [Overview](#Overview)
* [Example](#Example)
* [Sources](#Sources)

## Install
```
npm i base-ts-result
yarn add base-ts-result
pnpm add base-ts-result
```

## Overview

### Result
Interface that contains operation result and interaction methods
``` ts
interface Result<Val, Err> {
    // Contained Promise
    value: Val | Err;

    // Queries
    unwrap(): Val;
    unwrapErr(): Err;
    unwrapOr(altVal: Val): Val;
    unwrapOrElse(altValFactory: (err: Err) => Val): Val;
    expect(msg: string): Val;
    expectErr(msg: string): Err;
    isOk(): this is OK<Val>;
    isErr(): this is ERR<Err>;
    ok(): Val|undefined,
    err(): Err|undefined,

    // Mappers
    map<MappedVal>(mapper: (val: Val) => MappedVal): Result<MappedVal, Err>;
    mapOrElse<MappedVal>(mapper: (val: Val) => MappedVal, fallback: (err: Err) => MappedVal): Result<MappedVal, Err>;
    mapErr<MappedErr>(mapper: (err: Err) => MappedErr): Result<Val, MappedErr>;

    // Utilities
    inspect(inspector: (val: Val) => any): Result<Val, Err>;
    inspectErr(inspector: (err: Err) => any): Result<Val, Err>;
    toAsync(): AsyncResult<Val, Err>;
}

// Result implementations
class OK implements Result;
class ERR implements Result;
```

### AsyncResult
Class that contains operation result and interaction methods for async code
``` ts
type AsyncMapped<T> = T|Promise<T>
// Async result implementation for more ergonomic usage of Results with async code
class AsyncResult<Val, Err> {
    // Contained promise
    promise: Promise<Result<Val, Err>>; 

    // Creators
    static fromPromise<Val>(promise: Promise<Val>): AsyncResult<Val, unknown>;
    static fromResult<Val, Err>(result: Result<Val, Err>): AsyncResult<Val, Err>;

    // Queries
    async unwrap(): Promise<Val>;
    async unwrapErr(): Promise<Err>;
    async unwrapOr(altVal: Val): Promise<Val>;
    async unwrapOrElse(altValFactory: (err: Err) => AsyncMapped<Val>): Promise<Val>;
    async expect(msg: string): Promise<Val>;
    async expectErr(msg: string): Promise<Err>;
    async isOk(): Promise<boolean>;
    async isErr(): Promise<boolean>;
    async ok(): Promise<Val|undefined>;
    async err(): Promise<Err|undefined>;

    // Mappers
    map<NewVal>(mapper: (val: Val) => AsyncMapped<NewVal>): AsyncResult<NewVal, Err>;
    mapOrElse<NewVal>(
        mapper: (val: Val) => AsyncMapped<NewVal>,
        fallback: (err: Err) => AsyncMapped<NewVal>
    ): AsyncResult<NewVal, Err>;
    mapErr<NewErr>(mapper: (err: Err) => AsyncMapped<NewErr>): AsyncResult<Val, NewErr>;

    // Utilities
    inspect(inspector: (val: Val) => any): AsyncResult<Val, Err>;
    inspectErr(inspector: (err: Err) => any): AsyncResult<Val, Err>;
}
```

### Constructors
Handy functions to create Result objects

```ts
// create success Result
function Ok<Val>(res: Val): OK<Val>;

// create error Result
function Err<Err>(err: Err): ERR<Err>;
```

### Helpers
```ts
// Convert exceptions into errors & function result into Ok
function toResult<Val, ERR>(fn: () => Val): Result<Val, ERR>;

// Convert function returning T and error mapper returning E to new function returning Result<T, E>
function resultify<T, E, F extends Fn<T>>(fn: F, mapErr: (err: unknown) => E): ResFn<T, E, F>;

// Convert function returning Promise<T> and error mapper returning E to new function returning AsyncResult<T, E>
function asyncResultify<T, E, F extends AsyncFn<T>>(fn: F, mapErr: (err: unknown) => E): AsyncResFn<T, E, F>;
```

## Example
```ts
import { Ok, Err, Result, toResult, toResultAsync } from 'base-ts-result';

const generateNumber = (): Result<number, string> => {
    const num = Math.round(Math.random() * 100);

    if (num > 50) {
        // return ok result
        return Ok(num);
    }

    // return error result
    return Err('Number below 50');
};

const handleResult = (res: Result<number, string>) => {
    // Result is OK
    // ===============
    res.unwrap(); // 15
    res.unwrapOr(); // 15
    res.expect('Custom exception msg'); // 15
    res.unwrapErr(); // Exception: tried to get value as error from Ok result

    // Result is ERR
    // ===============
    res.unwrap(); // Exception: Unwrap error Result
    res.unwrapOr(-1); // -1
    res.expect('Custom exception msg'); // Exception: Custom exception msg
    res.unwrapErr(); // Error object

    // If you wanna handle error in higher function, just check 
    // =========================================================
    if (res.isError) {
        return res;
    }

    // some regular logic..
    return Ok('success handle result');
}

const getResult = () => {
    // res.isError = true;
    let res = toResult(() => {
        throw new Error('like to throw exceptions');
    });

    // res.isError = false;
    res = toResult(() => {
        return 7;
    });
};

const getResultAsync = async() => {
    // imitating promise with success result
    function timer<T>(val: T): Promise<T> {
        return new Promise(res => {
            setTimeout(() => {
                res(val);
            }, 100);
        });
    }

    // imitating promise with error result 
    function errTimer(): Promise<never> {
        return new Promise((_, rej) => {
            setTimeout(() => {
                rej();
            }, 100);
        });
    }

    // res.isError = false; res.unwrap() = 1;
    let res = await toResultAsync(() => {
        return timer(1);
    });

    // res.isError = true;
    res = await toResultAsync(() => {
        return errTimer();
    });

    // Or you can pass a promise
    const timerPromise = timer(2);
    const errTimerPromise = errTimer();

    // res.isError = false; res.unwrap = 2;
    res = await toResultAsync(timerPromise);

    // res.isError = true;
    res = await toResultAsync(errTimerPromise);
}

```

## Sources
- [:package: this package on npm](https://www.npmjs.com/package/base-ts-result)
- [:octocat: this package on github](https://github.com/Kostayne/base-ts-result)
