---
layout: post
title: "Angular: Migrate to Jest, from Karma + Jasmine"
author: "Phitrapy"
date:   2022-08-07 16:18:00 +0200
tags: angular jest jasmine karma testing
---

__Intro: Angular Unit Testing environment__

// TODO

Default: karma x jasmine
Downsides of karma x jasmine
A new project should run with Jest
Migrating to Jest can be cumbersome


## Why Jest
// TODO

Performance - Lighter & Faster tests (up to x3)
Better integration with IDEs
Integrations with Nx
* computation caching
Future proof

## Migrate!
// TODO

Steps:
1. Run the Briebug schematic
2. Update your configuration if needed
3. Migrate the outdated API
4. Misc

### Getting Started - Briebug schematic

Find the repo of the schematic [here](https://github.com/briebug/jest-schematic)

The readme is instructive

Begin the migration running:
```sh
ng add @briebug/jest-schematic
```

If you are running a brand new project, that's it! If not, you will need to follow a few more steps...


### Update your configuration if needed

#### __.gitignore__

The default output folder for test reports is `./coverage`. Make sure to add it to your `.gitignore` file!


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


### Migrate the API

// TODO

### Misc

// TODO
