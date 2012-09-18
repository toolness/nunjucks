## How Nunjucks is Different from Jinja2

There are a few differences due to different semantics between
javascript and Python or missing features. 

**Missing features that will be implemented (in order of priority):**

* Autoescaping
* Whitespace control: `{%-` and `-%}`
* Installing custom tags
* Macros
* The special `self` variable
* inline conditionals: `for if bar else baz`
* `for` does not support `if not` and `else`
* The `loop.cycle` method within loops
* The form `for k, v in item` is more constrained 
  * Unpacking doesn't really exist, it only works on dicts
  * Some of the special loop variables don't exist:
      * `loop.revindex`
      * `loop.revindex0`
      * `loop.last`
      * `loop.length`
  * Note that these exist when looping over arrays with `for item in items`

**Missing features that probably won't be implemented:**

* Scoping is not as fine-grained, special modifiers like `with context` does not exist
* Tests: `if i is divisibleby(3)`
* Named block end-tags: `{% endblock content %}`
* Sandboxed mode
* Line statements: `# for item in seq`
* Using `block` inside of `for` loops does not work
* The API is different, especially for writing custom tags. See `API`.

**In addition, the following filters don't exist,** but they will be gradually implemented:

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