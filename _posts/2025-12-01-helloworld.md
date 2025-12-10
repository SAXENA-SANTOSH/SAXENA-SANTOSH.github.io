---
title: "Hello World — Jekyll & Liquid tags reference"
layout: post
date: 2025-12-01 09:00:00 +0000
categories: [reference, jekyll]
tags: [liquid, jekyll, reference]
excerpt: "A compact, copyable reference of common Liquid tags/filters and Jekyll helpers for writing posts and templates."
---

This post is a living reference of commonly-used Liquid tags, filters and Jekyll helpers with short explanations and copyable examples. Examples are shown verbatim using `{% raw %}` so you can copy them into your templates or posts.

<!-- TOC -->

- Front matter
- Output / URL helpers
- Control flow (if / case / unless)
- Loops (for / cycle / continue / break)
- Variables (assign / capture / default)
- Includes & renders
- Filters (date, replace, truncate, where, sort, map, join, uniq, markdownify, strip_html)
- Useful tags (raw, comment, highlight, paginate, include_cached)
- Collections & data
- Examples & recipes

## Front matter (post/page metadata)

YAML front matter configures a page/post. Minimal example:

{% raw %}
---
title: "Hello World"
layout: post
date: 2025-12-01 09:00:00 +0000
categories: [blog]
tags: [example, hello]
excerpt: "Short summary shown in listings"
image: /assets/img/hero.jpg
published: true
---
{% endraw %}

## Output / URL helpers

- `relative_url` and `absolute_url` to build links that respect `baseurl` and `url`.

Example:

{% raw %}
<a href="{{ '/about/' | relative_url }}">About</a>
<a href="{{ '/assets/img/hero.jpg' | absolute_url }}">Full image URL</a>
{% endraw %}

- `post_url` and `post_path` (useful for permalinks in posts):

{% raw %}
{% post_url 2025-12-01-hello-world %}
{% endraw %}

## Control flow

### if / elsif / else

Use conditionals for rendering logic.

{% raw %}
{% if page.image %}
	<img src="{{ page.image | relative_url }}" alt="{{ page.title }}">
{% elsif page.excerpt %}
	<p>{{ page.excerpt }}</p>
{% else %}
	<p>No image or excerpt provided.</p>
{% endif %}
{% endraw %}

### unless

Opposite of `if`.

{% raw %}
{% unless page.published == false %}
	<!-- show content only when published is not false -->
{% endunless %}
{% endraw %}

### case / when

Switch-style logic.

{% raw %}
{% case page.layout %}
	{% when "post" %}  <p>Post layout</p>
	{% when "page" %}  <p>Page layout</p>
	{% else %}           <p>Other</p>
{% endcase %}
{% endraw %}

## Loops

### for

Loop over arrays or collections.

{% raw %}
{% for post in site.posts limit:5 %}
	<article>
		<h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
		<time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%b %-d, %Y" }}</time>
	</article>
{% endfor %}
{% endraw %}

Access loop helpers with `forloop`:

{% raw %}
Index: {{ forloop.index }} (1-based)
Is last: {{ forloop.last }}
{% endraw %}

### cycle

Cycle values (useful for zebra row classes).

{% raw %}
{% for item in site.data.items %}
	<div class="row {% cycle 'even' 'odd' %}">{{ item.name }}</div>
{% endfor %}
{% endraw %}

### break / continue

You can use `break` and `continue` inside `for` loops:

{% raw %}
{% for p in site.posts %}
	{% if p.tags contains 'skip' %}
		{% continue %}
	{% endif %}
	{% if forloop.index == 5 %}
		{% break %}
	{% endif %}
	{{ p.title }}
{% endfor %}
{% endraw %}

## Variables and capture

### assign

Assign a simple variable.

{% raw %}
{% assign author = "Santosh" %}
Hello, {{ author }}
{% endraw %}

### capture

Capture a block of rendered content into a variable.

{% raw %}
{% capture greeting %}
	<strong>Hi, {{ page.author | default: "Author" }}</strong>
{% endcapture %}
{{ greeting }}
{% endraw %}

### default filter

Provide a fallback value in templates.

{% raw %}
{{ page.author | default: "Guest" }}
{% endraw %}

## Includes & render

Use `_includes/` for reusable fragments. Prefer `render` (available in modern Jekyll) because it creates a new variable scope.

Create `_includes/alert.html`:

{% raw %}
<div class="alert alert-{{ include.type | default: 'info' }}">
	{{ include.message }}
</div>
{% endraw %}

Include it with `render`:

{% raw %}
{% render "alert.html", type: "warning", message: "This is a warning" %}
{% endraw %}

Or with the classic `include` (older style, shares scope):

{% raw %}
{% include alert.html type="info" message="Heads up" %}
{% endraw %}

## Filters (common)

Filters transform strings, arrays, numbers and dates. Chain them with `|`.

- `date` / `date_to_xmlschema`
- `truncate`, `truncatewords`
- `strip_html`, `strip_newlines`
- `replace`, `replace_first`, `remove`, `remove_first`
- `split`, `join`, `map`, `uniq`, `sort`, `where`, `where_exp`
- `markdownify` (render markdown to HTML)

Examples:

{% raw %}
{{ page.date | date: "%b %-d, %Y" }}
{{ post.content | strip_html | truncate: 140 }}
{{ site.posts | where: "category", "news" | sort: "date" | reverse }}
{{ site.data.authors | map: "name" | join: ", " }}
{{ some_markdown_variable | markdownify }}
{% endraw %}

### where / where_exp

Filter collection by attribute or expression.

{% raw %}
{% assign tutorials = site.posts | where: "tags", "tutorial" %}
{% assign ruby_posts = site.posts | where_exp: "p", "p.content contains 'Ruby'" %}
{% endraw %}

## Useful tags

### raw

Wrap blocks you don't want Liquid to process (very useful when showing Liquid examples).

{% raw %}
{{ this_would_normally_be_interpreted }}
{% endraw %}

### comment

Add comments that won't be output.

{% raw %}
{% comment %}This comment is stripped from output{% endcomment %}
{% endraw %}

### highlight

Syntax-highlight a block of code in older Jekyll versions using `highlight`:

{% raw %}
{% highlight html %}
<p>Hello world</p>
{% endhighlight %}
{% endraw %}

### paginate (if using pagination plugin / site generator)

Example for paginating posts in an index layout (requires pagination support):

{% raw %}
{% for post in paginator.posts %}
	{{ post.title }}
{% endfor %}

{% if paginator.total_pages > 1 %}
	{% if paginator.previous_page %}<a href="{{ paginator.previous_page_path }}">Prev</a>{% endif %}
	Page {{ paginator.page }} of {{ paginator.total_pages }}
	{% if paginator.next_page %}<a href="{{ paginator.next_page_path }}">Next</a>{% endif %}
{% endif %}
{% endraw %}

### include_cached

If your Jekyll has `include_cached` (speeds up repeated includes) you can use it similarly to `include`/`render`. Not all versions have this tag.

## Collections & Data

Collections live in `_collection_name/` and are exposed as `site.collection_name`.

Data files sit in `_data/` and are accessible via `site.data.filename`.

Example using data:

{% raw %}
{% for author in site.data.authors %}
	<h4>{{ author.name }}</h4>
	<p>{{ author.bio }}</p>
{% endfor %}
{% endraw %}

## Small recipes

### Tag list for a post

{% raw %}
{% if page.tags %}
	<ul class="tags">
	{% for tag in page.tags %}
		<li><a href="{{ '/tags/' | append: tag | relative_url }}">{{ tag }}</a></li>
	{% endfor %}
	</ul>
{% endif %}
{% endraw %}

### Responsive image example (basic)

{% raw %}
<figure>
	<img src="{{ '/assets/img/hero.jpg' | relative_url }}" alt="Hero" />
	<figcaption>Caption</figcaption>
</figure>
{% endraw %}

### Render markdown inside a variable

{% raw %}
{% capture md %}**Bold text** and _italic_ in a variable{% endcapture %}
{{ md | markdownify }}
{% endraw %}

## Notes & Gotchas

- When showing Liquid examples inside a page that Jekyll builds, wrap them with `{% raw %}`/`{% endraw %}` to avoid interpretation.
- Some tags and plugins (like pagination, include_cached) depend on your Jekyll version or GitHub Pages support — test locally.
- Filters such as `where_exp`, `group_by_exp` require newer Liquid/Jekyll versions.

## Where to go from here

This file is a compact reference. If you'd like, I can:

- Add an interactive copy button for each example in this post.
- Expand this into a dedicated reference page at `_pages/reference.md` with searchable examples.

If you want me to commit those extras now, tell me which you'd like (copy buttons, interactive search, or a full-page reference) and I'll implement it and preview locally.

