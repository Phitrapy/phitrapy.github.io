---
layout: post
title: "Angular: Migrate to Jest, from Karma + Jasmine"
author: "Phitrapy"
date:   2022-08-23 18:12:00 +0200
tags: angular jest jasmine karma testing
lastModifiedDate: 2022-08-23 18:17:43 +0200
---

If you have been running an Angular project for a couple years, you may still be using Karma and Jasmine for your unit tests. Actually, as of 2022, the `Karma` + `Jasmine` combo are still the default tools pre-configured during the generation of a new project, using the Angular CLI.

The Karma + Jasmine ecosystem has proven to be robust and powerfull, though better alternatives have rised during the past years.

Among them, there is `Jest`.

## Why Migrating to Jest

There are many reasons you would migrate to jest:
* **Better performance**, with lighter and faster tests
* Improved test reports, easier to read
* Excellent integration with many tools such as `Nx`
* Better integration with IDEs, with code coverage
* The API is quite backward compatible

## Should I Migrate to Jest?

As the migration can be cumbersome, if it does not induce too much complexity, **you should migrate**.
* If your project is brand new, **do it**.
* If your project has up to ~2000 unit tests, **at least try it**.
* If your project has many, many unit tests, **consider it**, and try to find the best approach to achieve the migration.

## Migrate!

>If your project is brand new, only do the first step. It will be very quick.

We will try to migrate from Jasmine to Jest but still change the least the test code. 

In order to migrate to Jest, we will follow these steps:
1. Run the Briebug schematic
2. Update your configuration if needed
3. Migrate the outdated API
4. Misc

>Remember to **commit** your code after each progress, even between steps.

### 1. Getting Started - Briebug schematic

In order to do most of the configuration seamlessly, we will use biebug's jest schematic:
>More info on the repo [here](https://github.com/briebug/jest-schematic). The readme is very instructive.

Install the schematic using:
```sh
ng add @briebug/jest-schematic
```

Then  run it using:
```sh
ng g @briebug/jest-schematic:add
```

Try to run your project with `ng test`. If the output contains errors, you should find some help bellow, on the next steps. If not, congratulation, you are done with the migration!

### 2. Update your configuration if needed

#### __Aliases__

*Symptom: There are issues with imports with @aliases when you run the tests.*

In order to setup valid aliases for your tests, you may need to  modify `jest.config.js`

```js
module.exports = {
  // ...
  moduleNameMapper: {
    '@core/(.*)': '<rootDir>/src/path-to/core/$1',
    '@shared/(.*)': '<rootDir>/src/path-to/shared/$1',
    '@other/(.*)': '<rootDir>/src/path-to/other/$1',
  },
  // ...
};
```

#### __Locale__

*Symptom: APIs using locale as arguments stop to work.*

You may need to add some configuration to `setup-jest.ts`. Here, with the locale `fr` and the right imports:


```ts
import { registerLocaleData } from '@angular/common';
import localeFr from '@angular/common/locales/fr';
import { LOCALE_ID } from '@angular/core';
import { TestBed } from '@angular/core/testing';

// ...

// Configure Locale data for unit testing
registerLocaleData(localeFr, `fr`);

beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [
      {
        provide: LOCALE_ID,
        useValue: `fr`,
      },
    ],
  });
});
```

#### __Custom fix for IMask__

*Symptom: Your project uses IMask from angular-imask, and the Unit Test Suites using it don't work anymore.*

To solve this issue, make a custom iMask factory mock class:
```ts
export class DefaultImaskFactory implements IMaskFactory {
  create<Opts extends IMask.AnyMaskedOptions>(
    el: IMask.MaskElement | IMask.HTMLMaskingElement,
    opts: Opts,
  ) {
    return IMask(el, opts);
  }
}
```

Then, in yout unit test suite, provide it:

```ts
await TestBed.configureTestingModule({
  // ...
  providers: [
    {
      provide: IMaskFactory,
      useClass: DefaultImaskFactory
    },
    // other providers...
  ]
  // ...
};
```


### 3. Migrate the API

Unfortunately, some of the Jest API is not compatible with Jasmine. Here is a *match & fix* list:

#### __Add a toolbox__

```ts
import MaybeMocked = jest.MaybeMocked;

/**
 * Replace jasmine.createSpyObj(className, ...methodNames) usage
 * @param className the name of the class
 * @param methodNames the name of the methods to mock
 */
export const createSpyObj = <T>(className: string, methodNames: (keyof T)[]): SpyObj<T> => {
  const obj: T = {} as T;

  for (let i = 0; i < methodNames.length; i++) {
    obj[methodNames[i]] = jest.fn() as any;
  }

  return jest.mocked<T>(obj);
};

/**
 * Add a custom "fail()" method
 */
export const customFail = (err: any = `This test shouldn't have failed`) =>
  expect(err || true).toBe(!(err || true));
```

#### __Replace `Spy<T>` by `SpyInstance<T>`__

Replace all:

```diff
// imports
- import Spy  = jasmine.Spy;
+ import SpyInstance  = jest.SpyInstance;
// OR
+ import Spy = jest.SpyInstance;

// usages if needed: replace "Spy" by "SpyInstance"
```

#### __Replace `and.returnValue` by `mockReturnValue`__

Replace all:

```diff
-  someSpy.and.returnValue(valueToBeReturned);
+  someSpy.mockReturnValue(valueToBeReturned);
```

#### __Replace `toBeTrue()` by `toBe(true)`__

Replace all:

```diff
-  expect(someValue).toBeTrue();
+  expect(someValue).toBe(true);
```

#### __Replace `toBeFalse()` by `toBe(false)`__

Replace all:

```diff
-  expect(someValue).toBeFalse();
+  expect(someValue).toBe(false);
```

#### __Replace `expect().nothing()` by `expect.anything()`__

Replace all:

```diff
-  expect().nothing();
+  expect.anything();
```

#### __Replace `fail()` by the custom `customFail()`__

Remember to import the customFail method from the toolbox, above.
Replace all:

```diff
-  fail();
+  customFail();
```

#### __Replace `calls.reset()` by `mockClear()`__

Replace all:

```diff
-  someSpy.calls.reset();
+  someSpy.mockClear();
```

#### __Replace `toHaveBeenCalledOnceWith(value)` by `toHaveBeenCalledTimes(1)` and `toHaveBeenCalledWith(value)`__

Remember to import the customFail method from the toolbox, above.

Replace all:

```diff
-  expect(someSpy).toHaveBeenCalledOnceWith(someValue);
+  expect(someSpy).toHaveBeenCalledTimes(1);
+  expect(someSpy).toHaveBeenCalledWith(someValue);
```


### 4. Misc

#### __.gitignore__

The default output folder for test reports is `./coverage`. Make sure to add it to your `.gitignore` file!


## Conclusion

Congratulations, you've been through this migration, at least I hope so! 🙏

[Email-me](mailto:clement.begnaud@proton.me) or [submit an issue](https://github.com/Phitrapy/phitrapy.github.io/issues/new) if you would like to give some feedback!

Thank you for reading!