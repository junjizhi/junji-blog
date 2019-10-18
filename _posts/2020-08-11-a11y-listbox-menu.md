---
layout: post
title: "Accessibility Comparison: Listbox vs. Menu"
categories: [All, Technical]
tags: [accessibility, a11y, listbox, menu, aria, aria-label, aria widget]
fullview: false
excerpt: When you press the Space key, what should happen to an accessible listbox or menu? I was often confused by these two widgets. So I decided to write about them.
comments: true
---

## Why I write this post?
If you are fixing the accessibility (aka. **a11y**) issues of an existing site, and seeing a widget like below:

<img width="847" alt="image" src="https://user-images.githubusercontent.com/2715151/89962869-ff10c580-dc13-11ea-8836-c187b8881643.png">
(Image source: https://ca.puma.com/)

Now answer this question: **Which design pattern is this widget using?**

Why is the question important? Because different widgets has different requirements on keyboard interactions, tag properties, states, etc.!

For example, you should be able to use the **Space** key to activate a dropdown if it is a **menu**, but there is no such key requirement for a **listbox**.

Another example: If a menu has submenu, your implementation should support **left / right arrow keys** to navigate to the submenus. Again, no such a thing for listboxes.

[W3 ARIA](https://www.w3.org/TR/wai-aria-practices-1.1/) site lists a handful of design patterns and widgets. But I was confused by these two particular ones: **[Listbox](https://www.w3.org/TR/wai-aria-practices-1.1/#Listbox)** and **[Menu](https://www.w3.org/TR/wai-aria-practices-1.1/#menu)**, because both of them can be implemented as a type of dropdown widget. So I decide to write a blog post to compare the two.

## Which type of listboxes are we talking about?

[Collapsable listboxes](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-collapsible.html), like this one:

<img width="271" alt="image" src="https://user-images.githubusercontent.com/2715151/89962276-40a07100-dc12-11ea-843a-6a2e1aaa9906.png">


There are other types of listboxes, like [scrollable listbox](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-scrollable.html), where the options are listed in a flatten manner:

![scrollable listbox screenshot](https://user-images.githubusercontent.com/2715151/89962317-5e6dd600-dc12-11ea-9d21-492881930a8c.png)


There are also [multi-select listboxes](https://www.w3.org/TR/wai-aria-practices-1.1/examples/listbox/listbox-rearrangeable.html):

![multi-select listbox screenshot](https://user-images.githubusercontent.com/2715151/89962483-ccb29880-dc12-11ea-8285-4cbfd609794a.png)

You can see the difference easily between a dropdown and other types of listbox, so there's no need to talk them here.

## Single Select Listboxes vs. Menus
The comparisons are mainly from the [W3 ARIA](https://www.w3.org/TR/wai-aria-practices-1.1/) site.

- Menu button
    - choosing an item can
        - have sub-menu
        - trigger an action
        - menu closes (list boxes may not close if it is multi-select)
    - menu buttons are usually organized as menu bar
        - usually stays atop of the content area
    - Example: Top nav bar is usually menu dropdown examples

- Keyboard interaction differences
    - listbox
        - Up / down arrow keys support
        - Type a character and it will focus on the option item
        - Home / End keys support are optional
    - menu button
        - Press enter to activate submenu or select the item, and closes the menu
        - Space key is activate dropdown or submenu
        - Up / down / left / right arrows
        - Home / end keys
        - Esc -> Close the menu
        - Tab (shift + tab) -> move to the next element
- ARIA states and properties
    - listbox
        - `role` = listbox
        - Option element has `role` = option
        - Each option is either a descendent of listbox element, or referenced by aria-owns element
        - If an option is selected, then aria-selected = true
    - menu button
        - `role` = button
        - `aria-haspopup` = true / menu
        - If a dropdown is shown, aria-expanded = true
        - item element has role = menuitem / menuitemcheckbox / menuitemradio
        - To make focus work correctly, `tabindex` needs to set correctly

## Differences Summary

From the above comparison, you can see that:
- Listbox has less keyboard interactions to implement than menu button
- Listbox also has less complicated states and properties to implement

Another visual difference is that listboxes often come with scroll bar while menu usually has a flat layout. As a result, if you have long menu, it should probably be a listbox.

## Going Back to the Beginning Question
Which widget is this?

<img width="847" alt="image" src="https://user-images.githubusercontent.com/2715151/89962869-ff10c580-dc13-11ea-8836-c187b8881643.png">

To me, because it's in a menu bar and it's clear that it's for navigation purposes, so it is a **menu**.

That means, we are supposed to at least use `Space` key to activate the dropdown menu. Unfortunately, that doesn't work. Even a big brand like [Puma](https://ca.puma.com/) doesn't get this right. 🤷‍♂️

We see such navigation menu bars in many ecommerce sites, such as [Amazon](https://www.amazon.com/), [Sephora](https://www.sephora.com/?country_switch=ca&lang=en), [Ebay](https://www.ebay.ca/). And none of the navigation bars support using the Space key to expand dropdowns.

To my surprise, [Kijiji](https://www.kijiji.ca/) navigation bar actually has **Space** key support. Esc and arrow keys also work great. Kudos to Kijiji:

<img width="849" alt="image" src="https://user-images.githubusercontent.com/2715151/89975209-2d51cd80-dc33-11ea-84a7-8d0381c7f0ab.png">

Obviously we have a lot of room to improve in terms of accessibility!

Disagree? Other thoughts? Drop me a comment below!
