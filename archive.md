---
layout: page
title: Archive
---

<div class="related">
<!--  <h2>Related Posts</h2> -->
  <ul class="related-posts">
    {% for post in site.posts limit:100 %}
      <li>
        <h3>
           <small>{{ post.date | date_to_string }}</small> 

          <a href="{{ site.baseurl }}{{ post.url }}">
            {{ post.title }}
            
          </a>
        </h3>
      </li>
    {% endfor %}
  </ul>
</div>

