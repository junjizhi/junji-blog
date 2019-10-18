---
layout: post
title: "Building Data Analytics App with Vue"
categories: [All, Technical]
tags: [vue, vuejs, vuex, data analytics, bootstrap-vue, bootstrap tables, btable, server logs, echarts, d3, d3-vue]
fullview: false
excerpt: My experience of building the front end for a data analytics app using Vue / BootstrapVue / Vuex / ECharts / D3
comments: true
---

## Introduction
Recently I've been using Vue to build a data analytics web app. The app fetches data from a private API and render them in different forms, e.g., data tables, charts, graphs, and other visualizations.

This is my first project using Vue. After playing with Vue for a few months, I understand why such a well-structured, light-weight web framework becomes popular.

Web frameworks debate aside, I want to talk about some lessons learned about building data analytics apps.

This post is for you if you are:

- planning to build a data analytics, and
- evaluating which web framework to use (React vs. Vue), or have picked Vue, and
- wanting to know what gotchas are ahead.

## Server logs analytics app

Without revealing too much details, let's say I was building a server logs analytics app. A screenshot looks like below:

![vue-data-app-screenshots](https://user-images.githubusercontent.com/2715151/113874021-5bb48800-9783-11eb-800f-f8b83fa5ef70.png)

Users can:
- See the raw logs
- Group the logs data
- Show them in tables
- Visualize nodes in a network topology tab
- Run SQL queries (for advanced users)

Think of the app as a simplified version of [Datadog](https://www.datadoghq.com/).


## Comparison: Vuetify vs. Bootstrap-Vue vs. other UI libraries

The project would need to render data tables. I found [a good article](https://medium.com/nextux/design-better-data-tables-4ecc99d23356) with the following **criteria for good data table libraries**:

-  Fixed header
-  Horizontal scroll
-  Resizable columns
-  Row style (stripe)
-  Display density
-  Visual Table Summary
-  Pagination
-  Hover Actions
-  Inline Editing
-  Expandable Rows
-  Quick View
-  Modal
-  Sortable Columns
-  Basic Filtering
-  Filter Columns
-  Searchable Columns
-  Add Columns
-  Customizable Columns

When I first started, choosing a UI library was a no-brainer. It's 2021 and rendering basic UI data analytics components, like tables or charts, should have been solved.

I narrowed the choices to [BootstrapVue](https://bootstrap-vue.org/) and [Vuetify](https://vuetifyjs.com/en/). These two libs check most items in the list above.

In the end I selected [BootstrapVue](https://bootstrap-vue.org/) because I worked with Bootstrap before and felt comfortable with the CSS styling naming. Also Bootstrap provides more freedom and space to tinker than Materials design.

Another consideration is, using Vuetify makes the app's look-and-feel too  Google-like. I didn't want to get locked-in early on.

## Using Bootstrap-Vue

Using a UI library like [BootstrapVue](https://bootstrap-vue.org/) sped things up a lot. I could quickly build a basic UI skeleton with stub data and asked for user feedback.

Especially, using [b-table](https://bootstrap-vue.org/docs/components/table#tables) was convenient. Most of the controls are supported. It is matter of finding the right props / variants / events for customization or implementing behaviours.

And its [documentation](https://bootstrap-vue.org/docs/components/table#tables) is well-organized and useful.

## The difficult parts

I ran into some difficulties not long after: **It requires work to integrate [b-table](https://bootstrap-vue.org/docs/components/table#tables) with Vuex and the backend API**.

Let me expand on this.

Most examples given in the [b-table docs](https://bootstrap-vue.org/docs/components/table#tables) are using a `data()` which is assumed available in the component. To fetch data dynamically from the API, we need to use `itemsProvider` callbacks. I [blogged](https://blog.junjizhi.com/all/technical/2021/03/21/boostrap-vue-table-plus-backend-api.html) about the trickiness to get `itemsProvider` worked well with [Vuex](https://vuex.vuejs.org/).

I spent a lot of time to on planning, implementing, and refactoring the code for the following features:

* searching
* filtering
* sorting

For example, when implementing the global search bar, I assume the search results would apply globally. In other words, global search affects all tabs.

To translate into the Vue terms, the table component needs to **watch the search input**.

As mentioned, `b-table` manages data by itself, so to make it play nicely with global search, I had to [force refresh the table](https://blog.junjizhi.com/all/technical/2021/03/21/boostrap-vue-table-plus-backend-api.html) while taking extra care to preserve all current table configurations.


### Data live in the front or back end?

During the implementation, this question came up a lot.

In particular, since I used Vuex to share data among components, it is easy to treat Vuex store as a _cache_.

But with the Vue reactivity design, [the UI soon became laggy even though the data array is not so large](https://forum.vuejs.org/t/how-could-i-improve-my-app-performance-with-huge-data/35524)!

A solution to this problem is make the large data array (which comes from the backend) **immutable**. Namely, I used `Object.freeze(...)` before passing the long array to Vuex store.

There are also [other best practices](https://stackoverflow.com/questions/52671666/efficiently-working-with-large-data-sets-in-vue-applications-with-vuex) to organize the Vuex store logic:


* Skip get & commit mechanism;
* Use actions to centralize CRUD
* (Advanced) Use indexing in Vuex store to improve perf

Overall, it needs some care and planning to get reasonable performance when rendering the data.

## Rendering data in charts and other visualization

I picked [vue-echarts](https://github.com/ecomfe/vue-echarts) for rendering the time series data in line or bar charts. These two turned out to be simple and asked most often.

For topology graphs, I used [vue-d3-network](https://github.com/emiliorizzo/vue-d3-network) which is a thin wrapper around [d3](https://d3js.org/) in Vue. I picked d3 because it seemed the most mature and powerful graph visualization chart I found. It opens lots of customization opportunities and potentially can be applied to other projects.

## Short notes about the back end API

As I mentioned above, I was using a private backend API, which takes care of  all data ingestion, modelling, indexing and supporting querying data via API.

These are hard data engineering problems, especially when dealing with large volume of real-time graph data, together with the requirements to support historical data. There's no one size for all solution.

The complexity of dealing with all these problem is out of scope.

## Summary
This post talks about my experience of implementing the front end for a data analytics application using Vue.

I used [Bootstrap-Vue](https://bootstrap-vue.org/), Vuex, together with a private API.

During the implementation, I ran into tricky problems for data wrangling tasks like searching / filtering / sorting.

For data visualization, I used [vue-echarts](https://github.com/ecomfe/vue-echarts) and [vue-d3-network](https://github.com/emiliorizzo/vue-d3-network)

Overall, Vue helped me spin up a quick front end scaffold and lays the foundation for future customization.