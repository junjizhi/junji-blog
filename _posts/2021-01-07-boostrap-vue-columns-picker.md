---
layout: post
title: "Building A Bootstrap-Vue table columns picker"
categories: [All, Technical]
tags: [vue, vuejs, bootstrap, bootstrap-vue, bootstrap tables, btable, BTable]
fullview: false
excerpt: How to build a columns picker UI for Bootstrap Vue Tables (b-table)
comments: true
---

Recently I've been using [bootstrap-vue](https://bootstrap-vue.org/) and found it a joy to build UI components with Vue.

One thing I needed was a [\<btable\>](https://bootstrap-vue.org/docs/components/table) columns picker. **I had a table with lots of columns**, and don't want to show all columns at once. User can customize the views themselves with this old-style UI.

The design is to have a columns picker like the one shown below. **Users can choose which columns to show on the table.**
![Columns picker UI screenshot](https://user-images.githubusercontent.com/2715151/103849872-db8f6100-5073-11eb-8427-4c1bdaa46b5c.png)

Editing the table columns is a one-off task. Doing so shouldn't need to leave the current page. After the configuration step, users can see the updated table. At the end, I made it as a modal.

I used the [\<b-modal\>](https://bootstrap-vue.org/docs/components/modal#modals) together with [\<b-select\>](https://bootstrap-vue.org/docs/components/form-select#form-select).

The core part of the code is like below:

```vue
<template>
  <b-modal
    :id="id"
    size="lg"
    centered
    title="Columns Picker"
    :static="true"
    @show="showCurrentColumnConfigs"
    @ok="applyColumnConfigs">
    <template #default="">
      <b-container>
        <b-row cols="11">
          <b-col cols="5">
            <b-form-group
            label="Select multiple columns to show"
            label-for="selected-hidden-columns"
          >
              <b-form-select
                id="selected-hidden-columns"
                v-model="modal.selectedHiddenColumns"
                :options="modal.hiddenColumns"
                multiple
                :select-size="20"
              />
          </b-form-group>
          </b-col>
          <b-col cols="1">
          <b-button-group vertical class="mt-5">
            <!-- ... render button icons here  -->
          </b-button-group>
          </b-col>
          <b-col cols="5">
            <b-form-group
            label="Shown"
            label-for="selected-shown-columns"
          >
            <b-form-select
              id="selected-shown-columns"
              v-model="modal.selectedShownColumns"
              :options="modal.shownColumns"
              multiple
              :select-size="20"
            />
          </b-form-group>
          </b-col>
        </b-row>
      </b-container>
    </template>
      <template #modal-footer="{ ok, cancel }">
      <b-container>
        <b-row align-h="center">
          <b-button
            @click="ok()"
            class="mr-4"
            :disabled="modal.shownColumns.length === 0">
            Apply
          </b-button>
          <b-button @click="cancel()">
            Cancel
          </b-button>
        </b-row>
      </b-container>
    </template>
  </b-modal>
</template>
```

To use this component, you can do:

```vue
<template>
  <div>
    <b-button variant="primary" class="mb-2" v-b-modal.columns-config-modal>
      Show Columns Picker
    </b-button>
    <BTableColumnsPicker
      :allColumns="allColumns()"
      :currentColumns="columns"
      :id="'columns-config-modal'"
      @apply="applyColumnConfigs"
    />
    <b-table
      id="dataList"
      striped
      bordered
      sticky-header="800px"
      head-variant="light"
      hover
      :items="items"
      :fields="columns"
    >
    </b-table>
  </div>
</template>
```

Some explanations:
- The component is a special [\<b-modal\>](https://bootstrap-vue.org/docs/components/modal#modals) that we can trigger with a button.
- The component takes in two column arrays which represent:
  - the current columns
  - all the available columns for picking
- Modal has two built-in custom events `@show` and `@ok` that we can listen and prepare rendering the two columns selectors
- When users click the `apply` button, the widget emits an `@apply` event and the parent component can handle it accordingly.
- To make the component test friendly, I added `:static="true"` to the `b-modal`. This renders the modal content in-place in the DOM instead of appending it to the body. With this, the [jest test](https://github.com/junjizhi/btable-columns-picker/blob/main/src/components/__tests__/BTableColumnsPicker.spec.js) can examine the content and make assertions.
- The current UI uses two multi-select components from bootstrap. And it doesn't support reordering the selection easily. If you need reordering, consider using the two-list [vue-draggable](https://sortablejs.github.io/Vue.Draggable/#/simple).

The rest of the logic is available in [this repo](https://github.com/junjizhi/btable-columns-picker).

P.S. I tried to [follow the instructions](https://vuejs.org/v2/cookbook/packaging-sfc-for-npm.html) and publish this as an NPM package. But I didn't get  the `bootstrap-vue` dependency to work nicely with `rollup`. If you have done it before or have thoughts, feel free to drop me a note!