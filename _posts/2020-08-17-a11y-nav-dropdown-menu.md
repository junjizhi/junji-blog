---
layout: post
title: "Hover-and-Expand Navigation Menu Is Bad for Accessibility"
categories: [All, Technical]
tags: [accessibility, a11y, menu, dropdown, hover-and-expand, navigation bar, navigation menu, flyout menu]
fullview: false
excerpt: Hover-and-expand menus are bad for web accessibility because it is based on a desktop application UI pattern. We should avoid such a design if possible.
comments: true
---

## Introduction
Many UI widgets have problems if you reason it from the angle of web accessibility.

Hover-and-expand menu is one example. Because the top level element is a link, users expect link behaviours. But we also have a submenu to fly out when users hover the mouse.

Essentially, **we are mixing a link and a menu button**. Because of this, it's awkward for keyboard and screen reader (e.g., [NVDA](https://www.nvaccess.org/)) users. **Do you expect to activate a link or see a submenu when you press `Enter` or `Space`**?

There are some work-arounds, but we should avoid such a design at the first place, if possible. I also talk about other suggestions as well at the end of the post.

## Many sites use this design
If you go to big brand sites like [Puma](https://ca.puma.com/en_CA/home), you see UI like this:

![On Puma's homepage, hover your mouse over a navigation item, a rich dropdown menu flies out.](https://user-images.githubusercontent.com/2715151/90303587-32a25880-de7d-11ea-93f1-4743125c1357.gif)

I get it. They have lots of products and it's impossible to list them all in one page. So they build a taxonomy and neat navigation menu for people to find the awesome products.

The UI is seemingly simple for sighted users: Hover the mouse over any top category, say `Women`, and a rich, well-organized sub-menu flies out. There are plenty of stuffs there and they hope you find what you need!

**But what about keyboard users, or visually impaired users who use screen readers? How can they see a submenu?**

You can use Javascript code to bind a key listener. When a user presses certain key, the menu shows up.

While it sounds easy, but there is only one catch: **The top navigation element is also a link, which complicates things!**

For example, normal users can click on it (e.g., `Women`) and go to that category:

![Puma homepage, the Women's category link is clicked](https://user-images.githubusercontent.com/2715151/90303585-29b18700-de7d-11ea-968f-4ac8ed41d7f2.png)

This leads to the following question:

## Are you building a menu button?
As mentioned in [my previous blog](https://blog.junjizhi.com/all/technical/2020/08/11/a11y-listbox-menu.html), menu button is a pattern that requires the top element to have `role="button"`.

So you may think of such a solution:

```html
<nav>
  <a id="top-link" href="/women" role="button" aria-expanded="false" aria-haspopup="true">Women</a>
  <ul role="menu">
    <li> <a href="/shoes" role="menuitem"> Shoes </a></li>
    <li> <a href="/clothing" role="menuitem"> Clothing </a></li>
    ...
  </ul>
</nav>
```

To satisfy the space key requirements, you also add the space key handler to the button:

```javascript
$("#top-link").on("keydown", event => {
  switch (event.key) {
    case " ":
      _showDropdown();
      event.preventDefault();
      break;
  }
});
```

If you test this piece of code, it works on Mac OS X / Safari / VoiceOver setup.

However, there are two gotchas:

**Gotcha 1: It's bad for touch screen users.**

[Heydon's post](https://www.smashingmagazine.com/2017/11/building-accessible-menu-systems/) discusses this in detail.

**Gotcha 2: We need to let the user know that using the `Space` key will activate the dropdown instead of going to the link**.

A hack to this problem is add the instruction in the <a> `aria-label`:

```html
<a id="top-link"
  href="/women"
  role="button"
  aria-expanded="false"
  aria-haspopup="true"
  aria-label="Women. Press the Space key to show submenus.">
    Women
</a>
```

But this approach hardcodes the message in English, so it is not good for internationlization. Also it doesn't feel native to the users as you can't adjust the tone or message based on the type of screen-reading software.

**Gotcha 3: Binding space key handler with Javascript won't work in NVDA + Firefox + Windows!**

From my testing, NVDA intercepts space key on the `<a>` tag and **always** treats it as activating the link instead of executing our jquery code above.

There are two work-arounds:

**Work-around 1**: Wrap the top `<a>` link in a `<div>` and bind keydown and click listener to the div, instead of the link.

This way, NVDA won't treat the top element as a link by default, and pass the space key event correctly.

However, we still need to inform the users about the special keyboard interaction with the top element (Gotcha 2).

**Work-around 2**: Ask NVDA users to exit the browser mode and switch to focus mode by pressing: `Insert` + `Space` keys. This is definitely a bummer.

A deeper problem is that, links are fundamentally different from a button. We can't guarantee all screen readers implement it that way.

## What does ARIA say about this situation?
The [ARIA guide](https://www.w3.org/WAI/tutorials/menus/flyout/#use-button-as-toggle) recommends two approaches:

**Approach 1**: Use the top level item as a dropdown switch (instead of a web link)

[ARIA Navigation Menubar Example](https://www.w3.org/TR/wai-aria-practices-1.1/examples/menubar/menubar-1/menubar-1.html#) uses this approach. Clicking on the top level link goes to the link, which works fine. But pressing either enter or space expands the dropdown, which is **not** what we want.

**Approach 2**: Use a separate dropdown switch button

Both approaches need some negotiation with the design team. In other words, if your designer team proposes to use hover-and-expand menu, you may point them to ARIA site and suggest to think again in terms of accessibility.

In _Inclusive Design Patterns_ written by Heydon Pickering, if there are less than four items,  he recommends laying out the items directly, instead of hiding them in the dropdown.

In [a blog post](https://www.smashingmagazine.com/2017/11/building-accessible-menu-systems/), Heydon Pickering points out that ARIA dropdown menus are mainly for desktop applications. To support navigation menus on the web, **table of contents are more accessible UI** for sites with lots of content.

## Summary
A hover-and-expand menu is implementing a desktop application UI pattern. It's awkward to for a webpage's accessibility. We need work-arounds to get the ARIA menu's keyboard interactions working correctly iin all screen-readers / OS combos, e.g., NVDA + Windows.

Specifically, accessible menu users can use enter and space keys to perform different functions. But with top level as a link instead of a button, hover-and-expand menus don't fit well in the category of menu buttons. That's why it's awkward to support a mix of link and button keyboard interactions.

If you are tasked with implementing / fixing an accessible hover-and-expand menus, consider the following:
- If <4 items, avoid dropdowns and lay it out directly (save user interactions)
- [Use table of contents or topic pages to avoid dropdowns](https://www.smashingmagazine.com/2017/11/building-accessible-menu-systems/), or
- Add a dropdown button besides the top link ([ARIA recommended Approach 2](https://www.w3.org/WAI/tutorials/menus/flyout/#use-button-as-toggle))
- If the above doesn't apply, wrap the top link in a `<div>` and implement keydown and click listeners.

## References
- _Inclusive Design Patterns_ by Heydon Pickering
- [ARIA dropdown implementation guide](https://www.w3.org/WAI/tutorials/menus/flyout/#use-button-as-toggle)
- [Building Accessible Menu Systems](https://www.smashingmagazine.com/2017/11/building-accessible-menu-systems/) by Heydon Pickering