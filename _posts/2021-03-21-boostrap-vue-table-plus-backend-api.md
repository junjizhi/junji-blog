---
layout: post
title: "Bootstrap-Vue table itemsProvider + Vuex + Backend API"
categories: [All, Technical]
tags: [vue, vuejs, vuex, items provider, bootstrap, bootstrap-vue, bootstrap tables, btable, BTable]
fullview: false
excerpt: I present a work-around to get Bootstrap-Vue table to work with Vuex.
comments: true
---

Recently I've been using [bootstrap-vue](https://bootstrap-vue.org/) tables (aka. `b-table`). I [blogged about buidling a column picker](https://blog.junjizhi.com/all/technical/2021/01/07/boostrap-vue-columns-picker.html).

This post is about integrating `b-table` with Vuex + API. Bootstrap-Vue has [good documentation](https://bootstrap-vue.org/docs/components/table), but there are some hoops to jump through. So hopefully this post can help you.

## Context: Table changes affect other components
The most common use case for `b-table` is that we provide all the data to the `items`  prop, and it handles all the filtering, paging, or sorting in the front end.

But in my use case, **whenever user updates the table data, regardless it is sorting a column, pressing the next page, filtering a column by a keyword, it affects other components that would need to access the table configurations later**.

As a result, I would need to store the result in a shared store. I used [Vuex](https://vuex.vuejs.org/) for that.

## Using itemsProvider with Vuex
I found the [`itemsProvider`](https://bootstrap-vue.org/docs/components/table#using-items-provider-functions) function contains the current table configurations I need to store in Vuex store, including

- currentPage
- perPage
- filter
- sortBy
- sortDesc

We know that, whenever table configurations change, it calls the `itemsProvider` function again to get the latest data.

A natural thought is to dispatch a [Vuex action](https://vuex.vuejs.org/guide/actions.html) to the store and get the updated data.

Note that, we need to wait for the data in `itemProvider` function, otherwise b-table would have no data to render.

Luckily, we can return a promise in the `itemProvier` , so we have something like below:

```js
  itemsProvider(tableConfigs) {
    const promise = this.$store.dispatch("updateData", { tableConfigs })

    return promise.then(() => {
      const data = this.$store.data
      this.refreshTableOptions(events.currentPage, events.total)
      return data || []
    }).catch(error => {
      console.error(error)
      return []
    })
  }
}
```

This assumes that we return a promise in the Vuex action:

```js
Vuex.Store({
  // state and mutation defitions here
  // ...
  actions: {
    updateData({ commit, state }, { tableConfigs }) {
      // Store the current configs for later use
      commit('updateTableConfigs', { tableConfigs })

      // Fetch new data from API
      const promise = axios.get('/some/url?page=' + tableConfigs.currentPage + '&size=' + tableConfigs.perPage)

      // Return a promise. When resolved, it updates the store,
      // so the new data willl be available by then.
      return promise.then(data => {
        commit('updateApiData', { data })
        return data || []
      })
    }
  }
})
```

In this way, we store the fetched data as well as the current table configs in the Vuex store for later use as well.

## Refresh table because of other component changes?

Because the way we write `itemsProvider` here, referencing any Vuex store variable inside that function may make it observe any store changes.

However, this is not the case in my test. The solution I eventually arrived at is a manual approach: **Just watch any store variable changes and emit an event to [force table update](https://bootstrap-vue.org/docs/components/table#force-refreshing-of-table-data)**.

The code looks like something below:

```js
// inside the Vue component
export default {
  // data() and other stuffs
  computed: {
    ...mapState(["globalQuery"])
  },
  watch: {
    globalQuery() {
      // Force refreshing the data table.
      this.$root.$emit('bv::refresh::table', 'data-table')
    }
  }
}
```


## Architecture diagram
The data flow diagram looks like below:

![Blank diagram - Page 2](https://user-images.githubusercontent.com/2715151/111934789-8078e200-8a98-11eb-8bd7-baabfb16a436.png)

## Summary

In this post, I present a work-around to get Bootstrap-Vue table to work with Vuex. This is specific to my use case where I have some external components (e.g., a global search) that affects the data table, and also needs to reference the current table configurations.

The work-around mainly involves using `itemsProvider` function and using `watch` to force update the table when an external events happen. The data flow is a bit complicated but it gets the job done.

If you have better solutions, please drop me a note. I'm all ears!