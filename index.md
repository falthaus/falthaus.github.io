

## Posts

<table align="left">
		<tr><td>&nbsp;</td><td>&nbsp;</td></tr>
	{% for post in site.posts %}
		<tr><td align="left">{{ post.date | date: "%-d.%-m.%Y" }}</td><td align="left"><a href="{{ post.url }}">{{ post.title }}</a></td></tr>
	{% endfor %}
</table>