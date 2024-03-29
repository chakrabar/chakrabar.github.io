---
layout: page
title: "Hello there!"
excerpt: "Technical & non-technical titbits of everyday life"
---

Welcome to the technical & everyday-life notes from AC. Here I write about all things related to programming, product development & design. Mostly, I maintain some random developer notes. Sometimes I also write about non-technical, normal, everyday stuffs.

If you want to know more anout my profile or get in touch, go to the [About](/about) section.

NOTE: Some of the posts are written by other guests, as mentioned in the individual posts. I do not own them or deserve any credit for those posts, they belong to their respective authors 😄

----

👨🏻‍💻 I'm a software developer, working mostly as full-stack developer (if you believe that is a thing), deal with all things product engineering on a day to day basis. I write `tools`, `web apps`, `REST services` and do lot of `automation`, using `.Net`, `TypeScript`, `C#`, `JavaScript`, `Nodejs` and other related tools & technologies. I have keen interest in `Software Architecture` and I'm pretty serious about ***good design, great user experience & simple, clutter free, maintainable code***.

Check my tech-shots, or technical articles on the [Tech](/articles/) menu 💻.

----

**Apart from coding**, I take great interest in `travel`, `food`, `art` & `photography`. If you too are interested in some of them, go check the [Blog](/blog/) menu 🎨📸.

----

#### Some of the recent posts. See more in Tech/Blog menu.

<!--site.posts >> site.categories.articles-->
<!--
{% unless post.categories contains "notes"%}
html
{% endunless %}
-->
<ul class="post-list">
{% for post in site.posts limit:10 %}
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">[{{ post.categories.first | replace:'articles','tech' | upcase }}] {{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>