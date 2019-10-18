---
layout: post
title: "Adding Jest to Vue and Write Component Tests"
categories: [All, Technical]
tags: [vue, vuejs, jest, unit test, component test]
fullview: false
excerpt: The steps to add Jest tests to an existing Vue project. And how to follow Jest conventions.
comments: true
---

## Introduction

I have been playing with Vue recently and wonder how to write component tests for components.

If you are a Vue newbie like me, you would find the steps helpful to quickly get tests running.

My goals is to follow Vue / Jest conventions (e.g., naming test files, and test file locations), so that we start testing a new Vue project in the _right_ way.

## Context: I have an existing Vue app and want to add tests
If you are like me, I followed a bunch of Vue tutorials and used [vue-cli](https://cli.vuejs.org/guide/creating-a-project.html) to create a project.

_Note: If the skeleton project that vue-cli created comes with the stub files and _any_ conventional test file structure, then you don't need to read this post._

Now I want to learn how to add tests, because a project without test coverage is basically running naked and cause endless headaches.

## The steps that work
While the Vue documentation is generally helpful, for adding tests, [their documentation](https://vuejs.org/v2/guide/testing.html) doesn't have step-by-step guide.

The most helpful guide is from [DigitalOcean](https://www.digitalocean.com/community/tutorials/vuejs-vue-testing). And this post is mostly based on their article.

### Install necessary libraries
```bash
$ yarn add --dev jest @vue/test-utils @vue/cli-plugin-unit-jest
```

### Create `jest.config.js`
```js
module.exports = {
  preset: '@vue/cli-plugin-unit-jest',
  verbose: true
}
```

### Testing file and directory structure convention

For this one, I prefer the convention to add the `__tests__` folder close to the components. For example, for `Homepage` component, I have the following structure:

```
src/components
├── Homepage.vue
└── __tests__
    └── Homepage.spec.js
```

### Create test files in __tests__ folder
`Homepage.spec.js` content:

```js
import { mount } from '@vue/test-utils'
import Homepage from '../Homepage.vue'

test('renders a homepage with a keyword', () => {
  const wrapper = mount(Homepage, {
    propsData: {
      message: "Home"
    }
  })

  expect(wrapper.text()).toContain('Home')
})
```

Now you are in the [vue-test-utils](https://vue-test-utils.vuejs.org/guides/#getting-started) territory and you can follow their documentation there.

### Run tests
Two ways:

```bash
$ yarn jest
# or
$ yarn vue-cli-service test:unit
```

Or add the test script in the following to your `package.json`:

```diff
   "scripts": {
     "serve": "vue-cli-service serve",
     "build": "vue-cli-service build",
-    "lint": "vue-cli-service lint"
+    "lint": "vue-cli-service lint",
+    "test": "jest"
   },
```

And then run:
```bash
$ yarn test

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.35s, estimated 2s
Ran all test suites.
```