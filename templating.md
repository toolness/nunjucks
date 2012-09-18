# Templating Documentation

For now, head over to [jinja's
documentation](http://jinja.pocoo.org/docs/templates/) for templates.
Nunjucks supports most of jinja's features, and those docs are
very good. Nunjucks will get better docs over time.

Please read ["How Nunjucks is Different from
Jinja2"](#how-nunjucks-is-different-from-jinja2) to see what features
are missing. Nunjucks is being quickly developed and will implement
missing features over time.

## Variables

A variable looks up a value from the template context. If you wanted to simply display a variable, you would do:

```
{{ username }}
```

This looks up `username` from the context and displays it. Variable names can have dots in them which lookup properties, just like javascript. You can also use the square bracket syntax.

```
{{ foo.bar }}
{{ foo["bar"] }}
```

These two forms to the exact same thing, just like javascript.

## Filters

Filters are essentially functions that can be applied to variables. They are called with a pipe operator (`|`) and can take arguments.

```
{{ foo | title }}
{{ foo | join(",") }}
{{ foo | replace("foo", "bar") | capitalize }}
```

The third example shows how you can chain filters. It would display "Bar", by first replacing "foo" with "bar" and then capitalizing it.

Nunjucks comes with several (builtin filters)[#], and you can (add your own)[#] as well.

## Tags

Tags are special blocks that perform operations on sections of the template. Nunjucks comes with several builtin:

### For

`for` iterates over arrays and dictionaries.

```
<h1>Posts</h1>
<ul>
{% for item in items %}
  <li>{{ item.title }}</li>
{% endfor %}
</ul>
```

The above example lists all the posts using the `title` attribute of each item in the `items` array as the display value.

You can also iterate over dictionaries:

```
{% for ingredient, amount in food %}
  Use {{ amount }} of {{ ingredient }}
{% endfor %}
```

Food is a dictionary with ingredients as keys and amounts as values.

Inside loops, you have access to a few special variables:

* `loop.index`: the current iteration of the loop (1 indexed)
* `loop.index0`: the current iteration of the loop (0 indexed)
* `loop.revindex`: number of iterations until the end (1 indexed)
* `loop.revindex0`: number of iterations until the end (0 based)
* `loop.first`: boolean indicating the first iteration
* `loop.last`: boolean indicating the last iteration
* `loop.length`: total number of items

`loop.revindex*`, `loop.last`, and `loop.length` are currently unavailable when iterating over dictionaries.

### If

### Set

### Extends

### Block

### Include

Just like filters, you will be able to add your own (this functionality is not available yet).

## Expressions

You can also embed literal objects within filters and tags. 

### Math
### Comparisons
### Logic

## Comments