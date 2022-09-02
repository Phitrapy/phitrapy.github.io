---
layout: post
title: "Angular: Boost Karma with Karma-Parallel"
author: "Phitrapy"
date:  2022-09-02 17:57:00 +0200
tags: angular karma parallel unit testing
lastModifiedDate: 2022-09-02 17:58:43 +0200
---

If you are using karma as a test runner for your project, and try to improve your test performance, you may be thinking about migrating to Jest!

Though, in some case, it might be too complex. Fortunately, there is an alternative with minimal configuration: [the `karma-parallel` plugin](https://www.npmjs.com/package/karma-parallel).

## Performance 

This plugin will basically share the test suites between multiple karma instances, which will run at the same time.

During my experimentations, I could run my tests suite up to **30% faster** with this plugin.

## Limitations

Be carefull, the plugin didn't recieved any update since 4 years. Considering this to be a long term solution would not be reasonnable.

As the plugin *will run multiple instances of karma* at the same time, you may experience performance issues above 4 instances.

## Configuration example (angular)

```typescript
module.exports = function (config) {
  config.set({
    // Add "parallel" as the first item in the frameworks array
    frameworks: ['parallel', 'jasmine', '@angular-devkit/build-angular'],
    // Add the plugin in the plugins array
    plugins: [
      require('karma-parallel'),
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-sonarqube-unit-reporter'),
      require('karma-coverage'),
      require('@angular-devkit/build-angular/plugins/karma'),
    ],
    ...,
    // Customise the options of the plugin
    parallelOptions: {
      executors: 4, // Number of karma instance to run in parallel
      shardStrategy: 'description-length', // Read the documentation of the plugin
    },
  });
};
```

## Conclusion

`karma-parallel` is a great tool to speed-up your test run with minimal configuration.

Though you still might consider to use another solution in the long run, as this plugin is not maintained anymore.
