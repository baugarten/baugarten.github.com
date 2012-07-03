---
layout: page
title: My Posts!
tagline: 
---
{% include JB/setup %}
<ul class="posts">
{% for post in site.posts limit: 5 %}
  <div class="post_info">
    <li style="list-style: none">
           <h3 style="display:inline;"> <a href="{{ post.url }}">{{ post.title }}</a></h3>
            <span>({{ post.date | date:"%Y-%m-%d" }})</span>
    </li>
    </br> <em>{{ post.excerpt }} </em>
    </div>
  {% endfor %}
</ul>
