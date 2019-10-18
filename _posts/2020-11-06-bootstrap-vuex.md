---
layout: post
title: "BootstrapVue Form Input and Vuex"
categories: [All, Technical]
tags: [vue, vuejs, vuex, bootstrap, bootstrapvue, b-form, b-form-input, state management]
fullview: false
excerpt: How I get BootstrapVue form input to work together with Vuex states.
comments: true
---

## Introduction

Recently I have been playing with Vue (2.x) and using it to build a search bar. The UI looks a bit like this:

<img width="836" alt="image" src="https://user-images.githubusercontent.com/2715151/98372156-5227c800-200b-11eb-8237-2a9ff65535fd.png">

My goal is to render the search results in different tabs. Each tab is a different representation of the same results (tables, charts, etc.). This mandates that tabs share the same set of search results.

When users search a new keyboard in the global search bar, all tabs are updated with the new results.

To reuse the same search results, I need to store the results in a shared state, which leads me to [Vuex](https://vuex.vuejs.org/).

To build the UI, I use [BootstrapVue](https://bootstrap-vue.org/).

One tricky thing is to deal with the integration with Vuex because its [form handling](https://vuex.vuejs.org/guide/forms.html) is tricky.

## My solution

The Vue template code:

```vue
<template>
  <b-form @submit="search">
      <b-input-group prepend="Global Search">
        <b-form-input
          v-model="query"
          debounce="500"
          placeholder="Enter your keywords"
        ></b-form-input>
      </b-input-group>
    </b-form>
</template>
```

I use a [two-way computed property](https://vuex.vuejs.org/guide/forms.html#two-way-computed-property).

```js
<script>
import { mapState } from "vuex";

export default {
  name: "Homepage",
  computed: {
    query: {
      get() {
        return this.$store.state.query;
      },
      set(query) {
        this.$store.commit("updateQuery", { query: query });
      },
    },
    ...mapState(["results"]),
  },
  methods: {
    search(e) {
      e.preventDefault();
      this.$store.dispatch("search");
    }
  },
};
</script>
```

Then it's my vuex store:

```js
import Vuex from "vuex";
import api from './api';

const store = new Vuex.Store({
  state: {
    query: "",
    queryChanged: false,
    results: [],
  },
  mutations: {
    updateQuery(state, payload) {
      if (payload.query != state.query) {
        state.query = payload.query;
        state.queryChanged = true;
      }
    },
    updateResults(state, payload) {
      state.queryChanged = false;
      state.results = payload.results;
    },
  },
  actions: {
    search({ commit, state }) {
      if (state.queryChanged) {
        const results = api.search(state.query);
        commit("updateResults", { results });
      }
    },
  },
});

export default store;
```

_Note: To do less API calls, I made a state to keep track of whether query has been changed. The store update the results only when query has been changed._

## Alternative approach: `v-model` + local state

Another approach is to use the `v-model` + local state. In this case, you would have `data`, `computed` and `methods`:

```js
data() {
  return { query: "" };
},
computed: mapState(["results"]),

methods: {
  search(e) {
    e.preventDefault();
    this.$store.dispatch("search", { query: this.query });
  }
}
```
This approach requires vuex `store` to update the query.

I don't like this alternative because it means I always have to maintain the local query and the store state in sync, which comes for free in the my current approach.