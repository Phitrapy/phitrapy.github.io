---
layout: post
title: "Nx: Run all your stories at once"
author: "Phitrapy"
date:   2022-08-12 16:37:00 +0200
tags: dev storybook nx stories workspace front-end monorepo
lastModifiedDate: 2022-08-22 16:34:16 +0200
---

Storybook is a great tool for component development, review, and documentation.

Fortunately, the Nx monorepo tools are so handy they also include `generators` to add Storybook to your libraries!

If you're a total beginner with storybook on an Nx workplace, you can start here:
* For angular: [read the overview](https://nx.dev/storybook/overview-angular)
* For react: [read the overview](https://nx.dev/storybook/overview-react)

## What the fuss?

The thing is... You will have to run a distinct Storybook instance per library, splitting your stories among them.

So, is there a way to **keep our library architecture** *and* **display all our stories in a single Storybook instance** at the same time?

How do we figure this out?

## The workaround

Fortunately, there is a way to solve this problem quite easily!

We can add a new library to the workspace, and use it as a tool to configure a storybook instance to reach every story.

> ⚠️ In this exemple, I do not provide the whole generator collection name, (in my case `@nrwl/angular`). I suppose you have setup a default collection.

> ⚠️ I also suppose you already have at least an app or library with a working `stories.ts` file in your workspace.

### 1. Generate the tool library

Let's call this library `all-stories`. We can generate it using this command:
```shell
nx g lib all-stories
```

### 2. Add basic configuration

Now, let's generate the default configuration for our library and workspace!
```shell
nx g storybook-configuration all-stories
```

### 3. Target all stories in the workspace

In the library `all-stories`, open the `.storybook` folder. You can configure how storybook behaves by modifying the content of `main.js`.

Add the constant `allStories` at the root level of the file, which tells Storybook to look everywhere in the workspace for stories:
```javascript
const allStories = ['../../../**/*.stories.ts'];
```

Below, modify the `module.exports` object, and fill the `stories` attribute:
```diff
module.exports = {
  // other configuration...
  
-  stories: [
-    ...rootMain.stories,
-    '../src/lib/**/*.stories.mdx',
-    '../src/lib/**/*.stories.@(js|jsx|ts|tsx)',
-  ],
+   stories: allStories,

  // other configuration...
};
```

### 4. Test it!

To test if everything works fine, run storybook for the project "all-stories":
```shell
nx storybook all-stories
```

With default configuration, you should be accessing all your stories on http://localhost:4400 !


