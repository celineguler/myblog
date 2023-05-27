---
layout: post
title: "Hank Green"
date: 2023-05-21 13:19:15 +0300
categories: jekyll update
---
{% raw %}

<!-- Add the following code to display a list of posts with excerpts -->
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>

<!-- Add the excerpt separator to your post content -->
---
excerpt_separator: <!--more-->
---

<!-- Add your post content here -->
Excerpt with multiple paragraphs

Here's another paragraph in the excerpt.
<!--more-->
Out-of-excerpt

{% endraw %}
