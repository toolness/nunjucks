
<div class="tagline">Nunjucks is a <span class="highlight">full featured templating engine</span> for javascript.</div>

It is a
direct port of the Python-powered [jinja2](http://jinja.pocoo.org/)
templating engine and aims to be feature-complete with jinja2. It's targeted at server-side apps but can also be used in the browser.

It was born out of frustration with other javascript templating
engines. The most popular ones either are ugly and don't abstract
enough (EJS) or have too different of a syntax (Jade). The only other project like this is
[jinjs](https://github.com/ravelsoft/node-jinjs), which has
been abandoned.

## Features

* Beautiful templates with filters, looping, and all that you'd expect
* It's very fast, as fast as jinja2
* [Template inheritance](/templating#Template-Inheritance) allows you to easily share templates across
  your site, without having to touch code
* jinja2-style parsing of code `{{ foo('bar', 1) }}`
* [Lots](/templating) of builtin tags, filters, and operators
* Complete [browser-side support](/api#Using-Nunjucks-in-the-Browser) with precompiled templates and a small 8K runtime
* [Extensive](https://github.com/jlongster/nunjucks/blob/master/tests/test.js) and [thorough](https://github.com/jlongster/nunjucks/blob/master/tests/filters.js) tests

View [jinja2's homepage](http://jinja.pocoo.org/) for a full list of
features. Like that page says, "Jinja is beautiful":

```jinja
{% extends "layout.html" %}
{% block body %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```

With [robust browser-side support](/api#Using-Nunjucks-in-the-Browser) you can now render any of your templates server-side or client-side. It doesn't matter anymore, so stop worrying about how your site is divided!

## Installation

`npm install nunjucks`

## Usage

First, require nunjucks:

```js
var nunjucks = require('nunjucks');
```

You can create a template from a string and render it:

```js
var tmpl = new nunjucks.Template('Hello {{ username }}');
console.log(tmpl.render({ username: "james" }));
```

You can use an environment which allows you to fetch files from the
file system. See the [`Environment`](#environment) class for more details.

```js
var env = new nunjucks.Environment();
var tmpl = env.getTemplate('test.html');
console.log(tmpl.render({ username: "james" }));
```

## Express

You can also tell nunjucks to install itself in your express app.
Assuming your templates are in the `templates` folder:

```js
var nunjucks = require('nunjucks');
var express = require('express');

var env = new nunjucks.Environment(new nunjucks.FileSystemLoader('views'));
env.express(app);
```

The `FileSystemLoader` takes a path, so change it to wherever your templates live.

## Community

If you have any questions, or want to file a bug, please do so as a [github issue on the project](https://github.com/jlongster/nunjucks/issues).

## FAQ

### Why another templating system?

I've been unhappy with any of the existing ones for javascript.
Mustache is great, but it's not really built to be the main templating
system of a large app. It lacks sophisticated features such as
template inheritance, and the need is obviously there as seen in this
[github issue](https://github.com/mustache/spec/issues/38).

EJS is really ugly and forces you to put too much logic in templates.
Jade is cool but forces a completely difference whitespace-based
syntax on you, which isn't my style. jinjs is a jinja2 port, but is
buggy and abandoned. Most other systems that are similar to jinja2
either lack the specific jinja2 features I want or are hacky and buggy.

jinja2 is a proven, successful templating system. I think javascript
needs a robust, feature-complete port of it.

### Can I use Nunjucks in client-side javascript?

Yes, just include the `nunjucks-dev.js` file in the `browser` folder.

In production, you should use `browser/nunjucks.js` instead (or even better, the minified version with `-min`), and precompile your templates. This file is much smaller and only includes the runtime necessary without the full compiler.

To precompile your templates, use the `precompile.js` script in the `bin` directory.

### Can I share templates between Python/jinja2 and javascript/nunjucks?

No, that is not the purpose of this project. There will be subtle
differences as documented in the "How is nunjucks different than
jinja2" section, and some very obscure features may never be
implemented. Additionally, custom filters and tags are not easily
available to multiple languages, as they are written specifically in
Python or javascript.

### How hard is it to convert jinja2 templates to nunjucks?

It should be very easy. The differences are very small, but they are
there. Depending on what features you use, you may need to make small
tweaks. The biggest hurdle will be porting any custom filters or tags
to javascript.

Read the [How Nunjucks is Different from Jinja2](/differences)
page for more information on how nunjucks differs from jinja2.

### What are the differences between this and jinja2, if any?

There are a few subtle differences that mostly stem from different semantics between Python and javascript. There are also several small features that I still need to implement.

There is a page which gives a [detailed description of all differences](/differences).

## Status

All of the features that will be in v0.1 have been ported
over. I am
currently testing the codebase and letting it solidify before I make
the first release. The focus is now documentation, benchmarks, and
tests.

Features needed for v0.1:

* Thorough-ish documentation, especially the API and client-side stuff
* Display pretty errors
** Add a global error handler and attempt to inject nice errors
* Better express integration
* Benchmarks (it's fast, how to prove it?)

Features needed for v0.2:

* Macros, a few other missing jinja2 constructs
* Even more thorough documentation
* Autoescaping
