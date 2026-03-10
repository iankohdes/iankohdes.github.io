---
title: "Change line height of alert boxes"
date: 2026-03-11
draft: false
tags: ["alert boxes", "blowfish", "shortcodes"]
series: ["Setting up a Hugo website with the Blowfish theme"]
series_order: 4
---

As mentioned in [this section](https://iankohdes.github.io/posts/0001-setting-up-hugo-and-blowfish/#alert-boxes-or-callouts), alert boxes (also known as callouts or admonitions) help draw readers’ attention to specific portions of text. I use this all the time when making my personal notes in Obsidian, and what I particularly like about this feature is the ability to create different “categories” of alert boxes by using different icons.

By default, Blowfish’s alert boxes use a line height that is quite tall. It makes the text within these boxes a little uncomfortable to read, especially since it is taller than the regular text. For a long time, however, I didn’t know how to fix this as a non-frontend person. I _did_ know that I didn’t like text in narrow(er) columns with tall line heights. It irks me the same way as section headers or titles with tall line heights – very distracting.

Eventually I figured it out.

The trick was to add the following CSS snippet to this file (or at least I think it’s CSS): `themes/blowfish/layouts/shortcodes/alert.html`.

```html
<style>
  div {
    line-height: 1.5rem;
  }
</style>
```

Doing so also contradicts what I stated [in this section](https://iankohdes.github.io/posts/0002-overriding-default-fonts-in-blowfish/#making-changes-to-headhtml), which was to keep CSS and HTML customisations in separate files, and therefore not modify files inside the `themes/blowfish/` folder. But then… whatever. I got the line height of my alert boxes and regular text standardised without too much fuss, and for me that was worth the violation of convention.