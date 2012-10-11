# API Documentation

Nunjucks borrows a lot of the same concepts from [jinja2's
API](http://jinja.pocoo.org/docs/api/).

## Environment

An `Environment` is a central object which handles templates. It is
configurable, specifying how to load and render templates, and which
extensions are available. It is especially important when dealing with
template inheritance, as its used to fetch base templates. (Read more
about the `Environment` in
[jinja2](http://jinja.pocoo.org/docs/api/#basics)).

The `Environment` constructor takes an optional list of
[loaders](#loaders). You can pass a single loader or an array of
loaders. Loaders specify how to load templates, whether its from the
file system, a database, or some other source.

The type of loader depends on the environment.

#### In node.js

The `FileSystemLoader` is available. This takes a path to load templates from. If you don't pass any loaders, a `FileSystemLoader` is created pointing to the current working directory.

```js
var nunjucks = require('nunjucks');

// Loads templates from the current working directory
var env = new nunjucks.Environment();

// Loads templates from the "views" folder
var env = new nunjucks.Environment(new nunjucks.FileSystemLoader('views'));
```

#### In the Browser

The `HttpLoader` is available. This takes a URL to load templates from (must be the same domain, relative URLs are recommended). If you don't pass any loaders, a `HttpLoader` is created with the `/views` URL.

```
var nunjucks = require('nunjucks');

// Loads templates from /views
var env = new nunjucks.Environment();

// Load templates from /templates
var env = new nunjucks.Environment(new nunjucks.HttpLoader('/templates'))
```

#### Multiple Loaders

You can also specify multiple loaders and it will look through them in order. This is useful if you want to specify several different template paths, or use a custom loader.

```
// The environment will look in templates first,
// then foo/templates, and then try loading
// from your special MyLoader class
var env = new nunjucks.Environment([new nunjucks.FileSystemLoader('templates'),
                                    new nunjucks.FileSystemLoader('foo/templates'),
                                    new MyLoader()]);
```

### Registering custom filters

Call the `addFilter` method on the environment to register a filter. See <a href="/api#Custom-Filters">Custom Filters</a>.

### Loading a template

Call the `getTemplate` method to retrieve a template.

```
var tmpl = env.getTemplate('page.html');
tmpl.render({ foo: 'bar' });
```

### Rendering a template

The `render` method is a shortcut for rendering templates.

```js
env.render('page.html', { foo: 'bar' });

// is equal to

env.getTemplate('page.html').render({ foo: 'bar' });
```

### Registering with Express

Nunjucks comes with special [express](http://expressjs.com/) support. Simple call the `express` method and pass the app and everything will be set up for you.

```js
var app = express();
env.express(app);
```

For more details, see [Express support](/home#Express) on the overview page.

### Customizing Variable and Block Tags

If you want different tokens than `{{` and the rest for variables, blocks, and comments, you can pass a dict as the second argument to the constructor which specifies different tokens:

```js
var env = new nunjucks.Environment(null, {
  blockStart: '<%',
  blockEnd: '%>',
  variableStart: '<$',
  variableEnd: '$>',
  commentStart: '<#',
  commentEnd: '#>'
});
```

Using this environment, templates will look like this:

```
<ul>
<% for item in items %>
  <li><$ item $></li>
<% endfor %>
</ul>
```

## Template

A `Template` is an object that handles the compiling of template
strings and rendering them. The `Environment` method `getTemplate`
returns a `Template` object. You can also create one yourself.

```js
var nunjucks = require('nunjucks');
var tmpl = new nunjucks.Template('Hello {{ username }}');
```

Call the `render` method to render it with a context.

```js
tmpl.render({ username: "James" });

// Displays "Hello James"
```

You cannot use template inheritance if you create the Template by hand. Only templates retrieved from an `Environment` can use inheritance.

## Custom Filters

To install a custom filter, use the `Environment` method `addFilter`.
A filter is simply a function that takes the target object as the
first argument and any arguments passed to the filter as the other
arguments, in order.

```js
var nunjucks = require('nunjucks');
var env = new nunjucks.Environment();

env.addFilter('shorten', function(str, count) {
    return str.slice(0, count || 5);
});
```

This adds a filter `shorten` which returns the first `count`
characters in a string, with `count` defaulting to 5. Here is how it
is used:

```jinja
{# Show the first 5 characters #}
A message for you: {{ message|shorten }}

{# Show the first 20 characters #}
A message for you: {{ message|shorten(20) }}
```

## Custom Tags

Currently you cannot create custom tags. This will be easy to do but I
want to wait until the parser API stabilizes so that it doesn't change
after extensions are created.


## Using Nunjucks in the Browser

Since we're using javascript already, all of your templates are available in the browser! This is extremely helpful because you no longer have to write templates specifically for either the server or the browser.

First, you need to download the js file for the browser. You can get it in the [`browser` folder of nunjucks](https://github.com/jlongster/nunjucks/tree/master/browser).

You'll see 3 different files:

* `nunjucks-dev.js`: To be used in development, this includes the full compiler and loads templates individually when they are requested. Templates are not cached, so changes are automatically available.
* `nunjucks.js`: This is a 20K file that only includes the runtime needed for templates (no compiler). You must precompile your templates.
* `nunjucks-min.js`: To be used in production, this is an 8K file that is simply a minified version of `nunjucks.js`. You must precompile your templates.

You should use `nunjucks-dev.js` in development, and **only `nunjucks-min.js` in production**. The production version is *much* faster.

Once one of these files is loaded in the browser, a global object named `nunjucks` will be available with the same API as if you required it with node.

### Precompiling Templates

When you use `nunjucks-min.js` in production, **you have to precompile your templates**. Precompilation simply generates the javascript for your template once, and you save it and only load the generated code into the page. This is much faster than compiling templates every time the page loads.

Nunjucks comes with a script named `nunjucks-precompile` to perform this task (make sure you have `node_modules/.bin` in your `$PATH`). You pass it a folder and it compiles all of them and dumps javascript to standard out, which you should pipe into a single js file.

```
$ nunjucks-precompile
Usage: nunjucks-precompile <folder>

$ nunjucks-precompile views > templates.js

$ ls views
base.html
item.html
foo.html
```

Now just include `templates.js` in your site (or whatever you named the js file). Your script tags should look like this:

```html
<script src="/js/nunjucks-min.js"></script>
<script src="/js/templates.js"></script>
```

Make sure to include nunjucks before your templates. Your base js path might be different too.

The precompiled templates sets up an `Environment` object for you, which you can access as `nunjucks.env`. `nunjucks` is the global object defined by the main js file.

### Using Templates in the Browser

You don't want to change your javascript to specialize for the production version, which predefines an `Environment` object. Here's how you should make your code work in both environments:

```js
if(!nunjucks.env) {
    // If not precompiled, create an environment with an HTTP
    // loader
    nunjucks.env = new nunjucks.Environment(new nunjucks.HttpLoader('/views'));
}

// Define a shortcut if you want
var env = nunjucks.env;
```

You should change the `"/views"` argument of `HttpLoader` to whatever base URL your templates are served on. You can also define the `Environment` object wherever you like, just make sure to check for `nunjucks.env` if it's defined for precompiled templates.

Now, use templates like you normally would:

```js
var el = $(env.render('item.html', { name: 'james' }));
$('body').append(el);
```

By default, the development version loads the template every time it's rendered. It's very helpful for changes to be automatically viewable in some cases, but you can turn it off by passing `true` when constructing the `HttpLoader` as a second argument:

```js
nunjucks.env = new nunjucks.Environment(
    // Cache templates and never reload them
    new nunjucks.HttpLoader('/views', true);
);
```