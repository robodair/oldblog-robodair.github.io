---
layout: archive
permalink: /posts/
---

{% include base_path %}

# {{ site.data.ui-text[site.locale].recent_posts | default: "Recent Posts" }}

{% for post in site.posts %}
  {% include archive-single.html %}
{% endfor %}

