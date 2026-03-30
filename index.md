

## Posts
|&nbsp;|&nbsp;|
|:---|:---|
{% for post in site.posts %}
|{{ post.date | date: "%-d.%-m.%Y" }}|<a href="{{ post.url }}">{{ post.title }}</a>|
{% endfor %}
