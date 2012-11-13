# Templating Documentation

This is a basic overview of the features available in nunjucks. You should refer to [jinja's
documentation](http://jinja.pocoo.org/docs/templates/) for more in-depth documentation, but keep in mind [how nunjucks is different from
jinja2](/differences).

## Variables

A variable looks up a value from the template context. If you wanted to simply display a variable, you would do:

```jinja
{{ username }}
```

This looks up `username` from the context and displays it. Variable names can have dots in them which lookup properties, just like javascript. You can also use the square bracket syntax.

```jinja
{{ foo.bar }}
{{ foo["bar"] }}
```

These two forms to the exact same thing, just like javascript.

If a value is `undefined` or `null`, nothing is displayed. The same behavior occurs when referencing undefined or null objects. The following all output nothing if `foo` is undefined: `{{ foo }}`, `{{ foo.bar }}`, `{{ foo.bar.baz }}`.

## Filters

Filters are essentially functions that can be applied to variables. They are called with a pipe operator (`|`) and can take arguments.

```jinja
{{ foo | title }}
{{ foo | join(",") }}
{{ foo | replace("foo", "bar") | capitalize }}
```

The third example shows how you can chain filters. It would display "Bar", by first replacing "foo" with "bar" and then capitalizing it.

Nunjucks comes with several [builtin filters](/templating#Builtin-Filters), and you can [add your own](/api#Registering-custom-filters) as well.

## Template Inheritance

Template inheritance should work exactly like it does in jinja2. [jinja has great documentation on inheritance](http://jinja.pocoo.org/docs/templates/#template-inheritance) that you can read.

## Tags

Tags are special blocks that perform operations on sections of the template. Nunjucks comes with several builtin.

Just like filters, you will be able to add your own (this functionality is not available yet).

### For

`for` iterates over arrays and dictionaries.

```jinja
<h1>Posts</h1>
<ul>
{% for item in items %}
  <li>{{ item.title }}</li>
{% endfor %}
</ul>
```

The above example lists all the posts using the `title` attribute of each item in the `items` array as the display value.

You can also iterate over dictionaries:

```jinja
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

```jinja
{% if variable %}
  It is true
{% endif %}
```

If variable is defined and evaluates to true, "It is true" will be displayed. Otherwise, nothing will be.

You can specify alternate conditions with `elif` and `else`:

```jinja
{% if hungry %}
  I am hungry
{% elif tired %}
  I am tired
{% else %}
  I am good!
{% endif %}
```

### Macro

`macro` allows you to define reusable chunks of content. It is similar to a function in a programming language. Here's an example:

```jinja
{% macro field(name, value='', type='text') %}
<div class="field">
  <input type="{{ type }}" name="{{ name }}"
         value="{{ value | escape }}" />
</div>
{% endmacro %}
```
Now `field` is available to be called like a normal function:

```jinja
{{ field('user') }}
{{ field('pass', type='password') }}
```

Keyword/default arguments are available. See [keyword arguments](/templating#Keyword-Arguments) for a more detailed explanation.

You can [import](/templating#Import) macros from other templates, allowing you to reuse them freely across your project.

### Set

`set` lets you create/modify a variable.

```jinja
{{ username }}
{% set username = "joe" %}
{{ username }}
```

If `username` was initially "james', this would print "james joe".

You can introduce new variables, and also set multiple at once:

```jinja
{% set x, y, z = 5 %}
```

If `set` is used at the top-level, it changes the value of the global template context. If used inside scoped blocks, like `for`, `include`, and others, it only modifies the current scope.

### Extends

`extends` is used to specify template inheritance. The specified template is used as a base template.

```jinja
{% extends "base.html" %}
```

See [jinja's documentation on template inheritance](http://jinja.pocoo.org/docs/templates/#template-inheritance) for more information.

### Block

`block` defines a section on the template and identifies it with a name. This is used by template inheritance. Base templates can specify blocks and child templates can override them with new content.

```jinja
{% block css %}
<link rel="stylesheet" href="app.css" />
{% endblock }
```

You can even define blocks within looping:

```jinja
{% for item in items %}
{% block item %}{{ item }}{% endblock %}
{% endfor %}
```

Child templates can override the `item` block and change how it is displayed.

See [jinja's documentation on template inheritance](http://jinja.pocoo.org/docs/templates/#template-inheritance) for more information.

### Include

`include` pulls in other templates in place. It's useful when you need to share smaller chunks across several templates that already inherit other templates.

```jinja
{% include "item.html" %}
```

You can even include templates in the middle of loops:

```jinja
{% for item in items %}
{% include "item.html" %}
{% endfor %}
```

This is especially useful for cutting up templates into pieces so that the browser-side environment can render the small chunks when it needs to change the page.

### Import

`import` loads a different template and allows you to access its exported values. Macros and top-level assignments (done with [`set`](#set)) are exported from templates, allowing you to access them in a different template.

Imported templates are processed without the current context, so they do not have access to any of the current template variables.

Let's start with a template called `forms.html` that has the following in it:

```jinja
{% macro field(name, value='', type='text') %}
<div class="field">
  <input type="{{ type }}" name="{{ name }}"
         value="{{ value | escape }}" />
</div>
{% endmacro %}

{% macro label(text) %}
<div>
  <label>{{ text }}</label>
</div>
{% endmacro %}
```

We can import this template and bind all of its exported values to a variable so that we can use it:

```jinja
{% import "forms.html" as forms %}

{{ forms.label('Username') }}
{{ forms.input('user') }}
{{ forms.label('Password') }}
{{ forms.input('pass', type='password') }}
```

You can also import specific values from a template into the current namespace with `from import`:

```jinja
{% from "forms.html" import input, label as description %}

{{ description('Username') }}
{{ input('user') }}
{{ description('Password') }}
{{ input('pass', type='password') }}
```

## Keyword Arguments

jinja2 uses Python's keyword arguments support to allow keyword arguments in functions, filters, and macros. Nunjucks supports keyword arguments as well by introduction a new calling convention.

Keyword arguments look like this:

```jinja
{{ foo(1, 2, bar=3, baz=4) }}
```

`bar` and `baz` are keyword arguments. Nunjucks converts them into a hash and passes it as the last argument. It's equivalent to this call in javascript:

```js
foo(1, 2, { bar: 3, baz: 4})
```

Since this is a standard calling convention, it works for all functions and filters if they are written to expect them. [Read more](/api#Keyword-Arguments) about this in the API section.

Macros allow you to also use keyword arguments in the definition, which allows you to specify default values. Nunjucks automatically maps the keyword arguments to the ones defined with the macro.

```
{% macro foo(x, y, z=5, w=6) %}
{{ x }}, {{ y }}, {{ z }}, {{ w}}
{% endmacro %}

{{ foo(1, 2) }}        -> 1, 2, 5, 6
{{ foo(1, 2, w=10) }}  -> 1, 2, 5, 10
```

You can mix positional and keyword arguments with macros. For example, you can specify a positional argument as a keyword argument:

```jinja
{{ foo(20, y=21) }}     -> 20, 21, 5, 6
```

You can also simply pass a positional argument in place of a keyword argument:

```jinja
{{ foo(5, 6, 7, 8) }}   -> 5, 6, 7, 8
```

In this way, you can "skip" positional arguments:

```jinja
{{ foo(8, z=7) }}      -> 8, , 7, 6
```

## Comments

You can write comments using `{#` and `#}`. Comments are completely stripped out when rendering.

```jinja
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

```jinja
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

```jinja
{% if numUsers < 5 %}...{% endif %}
{% if i == 0 %}...{% endif %}
```

### Logic

* `and`
* `or`
* `not`
* Use parentheses to group expressions

Examples:

```jinja
{% if users and showUsers %}...{% endif %}
{% if i == 0 and not hideFirst %}...{% endif %}
{% if (x < 5 or y < 5) and foo %}...{% endif %}
```

### Function Calls

If you have passed a javascript method to your template, you can call it like normal.

```jinja
{{ foo(1, 2, 3) }}
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
