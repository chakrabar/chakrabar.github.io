---
layout: page
title: "Home"
---

## Welcome to Arghya C's tech shots

Here I talk about some daily tips & trick of programming. 
Mostly I maintain some random developer notes.

If you want to see my profile visit my [Github profile](https://github.com/chakrabar). Or go to [About](/about) section for more details.

----

I'm a lead/senior developer, working mostly as Fullstack developer, but I deal with more of the backend stuffs on a day to day basis. I write `tools, web apps, REST services` etc. using `.Net, JavaScript` and other related tools & technologies. I have keen interest in `Software Architecture` and I'm pretty serious about *`good design, great user experience & simple, clutter free, maintainable code`*.

Check my tech-shots, or short technical articles on the posts panel.

----

**Apart from coding**, I take great interest in `travel`, `food`, `art` & `photography`. In future I might add separate sections as well, for these areas of my interest.

----

<ul class="post-list">
{% for post in site.posts limit:10 %}
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">{{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>