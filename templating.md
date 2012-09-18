# Templating Documentation

This is a basic overview of the features available in nunjucks. You should refer to [jinja's
documentation](http://jinja.pocoo.org/docs/templates/) for more in-depth documentation, but keep in mind [how nunjucks is different from
jinja2](#how-nunjucks-is-different-from-jinja2).

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

Nunjucks comes with several [builtin filters](#Builtin-Filters), and you can [add your own](#) as well.

## Tags

Tags are special blocks that perform operations on sections of the template. Nunjucks comes with several builtin.

Just like filters, you will be able to add your own (this functionality is not available yet).

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

`if` tests a condition and lets you selectively display content. It behaves exactly as javascript's `if` behaves.

```
{% if variable %}
  It is true
{% endif %}
```

If variable is defined and evaluates to true, "It is true" will be displayed. Otherwise, nothing will be.

You can specify alternate conditions with `elif` and `else`:

```
{% if hungry %}
  I am hungry
{% elif tired %}
  I am tired
{% else %}
  I am good!
{% endif %}
```

### Set

`set` lets you modify the template context.

```
{{ username }}
{% set username = "joe" %}
{{ username }}
```

If `username` was initially "james', this would print "james joe".

You can introduce new variables, and also set multiple at once:

```
{% set x, y, z = 5 %}
```

### Extends

`extends` is used to specify template inheritance. The specified template is used as a base template.

```
{% extends "base.html" %}
```

See [jinja's documentation on template inheritance](http://jinja.pocoo.org/docs/templates/#template-inheritance) for more information.

### Block

`block` defines a section on the template and identifies it with a name. This is used by template inheritance. Base templates can specify blocks and child templates can override them with new content.

```
{% block css %}
<link rel="stylesheet" href="app.css" />
{% endblock }
```

You can even define blocks within looping:

```
{% for item in items %}
{% block item %}{{ item }}{% endblock %}
{% endfor %}
```

Child templates can override the `item` block and change how it is displayed.

See [jinja's documentation on template inheritance](http://jinja.pocoo.org/docs/templates/#template-inheritance) for more information.

### Include

`include` pulls in other templates in place. It's useful when you need to share smaller chunks across several templates that already inherit other templates.

```
{% include "item.html" %}
```

You can even include templates in the middle of loops:

```
{% for item in items %}
{% include "item.html" %}
{% endfor %}
```

This is especially useful for cutting up templates into pieces so that the browser-side environment can render the small chunks when it needs to change the page.

## Comments

You can write comments using `{#` and `#}`. Comments are completely stripped out when rendering.

```
{# Loop through all the users #}
{% for user in users %}...{% endfor %}
```

## Expressions

You can use many types of literal expressions that you are used to in javascript.

* Strings: `"How are you?"`, `'How are you?'`
* Numbers: `40`, `30.123`
* Arrays: `[1, 2, "array"]`
* Dicts: `{ one: 1, two: 2 }`
* Boolean: `true`, `false`

### Math

Like jinja, nunjucks allows you to operate on values (though this should be rarely used). The following operators are available:

* Addition: `+`
* Subtraction: `-`
* Division: `/`
* Division and integer truncation: `//`
* Division remainder: `%`
* Multiplication: `*`
* Power: `**`

You can use them like this:

```
{{ 2 + 3 }}       (outputs 5)
{{ 10/5 }}        (outputs 2)
{{ numItems*2 }}
```

Read more in [jinja's math documentation](http://jinja.pocoo.org/docs/templates/#math).

### Comparisons

* `==`
* `!=`
* `>`
* `>=`
* `<`
* `<=`

Examples:

```
{% if numUsers < 5 %}...{% endif %}
{% if i == 0 %}...{% endif %}
```

### Logic

* `and`
* `or`
* `not`
* Use parentheses to group expressions

Examples:

```
{% if users and showUsers %}...{% endif %}
{% if i == 0 and not hideFirst %}...{% endif %}
{% if (x < 5 or y < 5) and foo %}...{% endif %}
```

## Builtin Filters

Nunjucks has ported most of jinja's filers, so [go look in its docs](http://jinja.pocoo.org/docs/templates/#list-of-builtin-filters) for filters.

These filters are currently **not** implemented:

* `dictsort`
* `filesizeformat`
* `forceescape`
* `format`
* `pprint`
* `safe`
* `striptags`
* `sum`
* `truncate`
* `urlize`
* `wordwrap`
* `xmlattr`
