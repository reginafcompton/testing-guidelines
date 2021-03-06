# Intro to JavaScript Testing

* [The right style](#the-right-style)
* [Test your syntax with jshint](#jshint)
* [Test your logic with jasmine](#jasmine)
* [Friendly reminders](#friendly-reminders)

## The right style

JavaScript, as a language itself, does not have too many opinions. Functional, reliable, error-free JS comes in a variety of colors and shapes: in fact, the number of ways to [declare a function](https://www.bryanbraun.com/2014/11/27/every-possible-way-to-define-a-javascript-function/) reaches double digits. The flexibility of JS can create headaches, inspire pure rage, and ruin an afternoon.

Narrow your options! As you write code and tests-for-your-code, check your JS against a style guide. Consistent style makes code easier to test and debug, while also minimizing cross-browser errors – a considerable pain point in JS.

At DataMade, we recommend [our fork of the Node.js style guide](https://github.com/datamade/node-style-guide), a simple overview of best JS practices identified by [Felix Geisendörfer](http://felixge.de/). The [Crockford conventions](http://javascript.crockford.com/code.html) nicely complement the Node.js style guide: in it, [Douglas Crockford](https://en.wikipedia.org/wiki/Douglas_Crockford) gives a no-nonsense discussion of how to write "publication quality" JavaScript.

## jshint

An impeccable style guide, however, does not prevent human error. Every coder has at least one horror story about an hours-long hunt for a missing semicolon that caused the (sometimes silent) crash of an entire program.

[`jshint`](http://jshint.com/about/) can be the hero in these stories. `jshint` analyzes JavaScript line-by-line and reports typos and common bug-causing mistakes.

Easily integrate `jshint` into your workflow using one of three strategies:

* Spotcheck code! Copy-and-paste select lines of potentially error-raising JS in the [`jshint` online console](http://jshint.com/).
* Enable `jshint` as a constant, nagging reminder of JS best practices: add a JSHint plugin to your preferred code editor. At DataMade, many of us start on and continue using Sublime. For those Sublime users, install [JSHint Gutter](https://github.com/victorporof/Sublime-JSHint) through the Sublime package manager, and activate it by right clicking and selecting `JSHint --> Lint Code`. The results may be enlightening, but also exhausting: fix what needs fixing, and then clear the annotations with `CMD + esc`.
* Install the `jshint` [command line interface](http://jshint.com/docs/cli/) and [the standard DataMade linting rules](https://github.com/datamade/node-style-guide/blob/master/.jshintrc), and integrate it with the regular running of your test suite. Run `npm install -g jshint` to install JSHint globally. Then, in terminal, tell JSHint to lint your files.

The `jshint` CLI has several options. Most simply, the `jshint` command accepts a specific file as a parameter:

``` bash
# Inspect this JS file!
jshint my_unique_app/static/js/custom.js

# Discouraging output...
my_unique_app/static/js/custom.js: line 14, col 51, Missing semicolon.
my_unique_app/static/js/custom.js: line 18, col 48, Missing semicolon.
my_unique_app/static/js/custom.js: line 52, col 34, Missing semicolon.

3 errors
```

Alternatively, JSHint can recursively discover all JS files in a directory and return a report: `jshint .` The results, in this case, may not be useful, since JSHint lints third-party libraries, too. For greater precision, add a configuration file in the root of your app, called `.jshintignore`. Here, tell JSHint to ignore, let's say, the directory where you store external libraries.

```
my_unique_app/static/js/lib/*
```

You can configure the rules JSHint uses to lint your code via a [`.jshintrc` file](http://jshint.com/docs/). This file contains a JSON object that activates JSHint options. DataMade has [a standard `.jshintrc` template](https://github.com/datamade/node-style-guide/blob/master/.jshintrc). Use it to initiate testing with JSHint, and [customize it](http://jshint.com/docs/options/) to fit the needs of your project.

If you know a discrete portion of your code is not `jshint` compliant, but you want to leave your config in tact for the rest, you can tell `jshint` to ignore a line or code block using the `ignore` directive, like so:

```javascript
// Ignore a line.
console.log('please leave me be!'); // jshint ignore:line

// Ignore a code block.
/* jshint ignore:start */
function rulebreaker(i) {
    alert(i);
}
/* jshint ignore:end */
```

Note that this maneuver should _not_ be used just to get the tests to pass, but rather when there is not a better alternative. For example, JSHint complains when you leave a `console.log` in your code, but there are legitimate use cases for logging to the console in production, as in Councilmatic, where [we log the OCD ID of the entity being viewed](https://github.com/datamade/django-councilmatic/blob/5cc8ae50e8b0cf305afae563dd8deb29731204e7/councilmatic_core/templates/councilmatic_core/person.html#L232).

The JSHint CLI allows for automated testing with Travis. Include in your `.travis.yml` file instructions for installing `jshint` and directive(s) for running the linter:

```
...

install:
- pip install --upgrade pip
- pip install --upgrade -r requirements.txt
- npm install -g jshint

...

script:
- pytest tests
- jshint my_unique_app/static/js/custom.js

...
```

## jasmine

The right style and perfect syntax make for clean, readable, debuggable JavaScipt. Yet, testing JS functionality requires something distinct. For this, we recommend [Jasmine](https://jasmine.github.io/2.0/introduction.html), a testing framework for behavior-driven development.

### Python integration

Jasmine [plays nicely with Django and Flask apps](https://jasmine.github.io/setup/python.html). In the virtualenv of your app, install Jasmine and initialize a project:

```
# Install
pip install jasmine

# Initialize a project
jasmine-install
```

`jasmine-install` creates a `spec` repo, where all testing specs and the configuration file (i.e., `jasmine.yml`) live. Before writing and running tests, tell Jasmine where to find relevant files by specifying, most importantly, the `src_files` and `src_dir` attributes:

```
...

src_files:
  - js/*

...

src_dir: 'my_app/static'

...
```

Then, in terminal, run:

```bash
jasmine
```

Visit `http://127.0.0.1:8888`, and view your test results.

Of course, without any tests, you should see relatively sparse output, something along these lines: "finished in 0.008s No specs found". Let's change that by writing a test! Open a JS file, and create a simple function – but not within `$(document).ready`. Why? `$(document).ready` [hides the functions inside a closure](http://bittersweetryan.github.io/jasmine-presentation/#slide-17).

```javascript
function makeTheTruth() {
    return true;
};
```

Then, make a spec file:

```bash
touch spec/javascripts/custom_spec.js
```

Open the newly created spec file, and add a test:

```javascript
describe("Test custom.js", function() {
  it("returns true", function () {
      var result = makeTheTruth();
      expect(result).toBe(true);
  });
});
```

Reload `http://127.0.0.1:8888`, and the browser should show the number of tests and the number of failures: "finished in 0.007s 1 spec, 0 failures".

### Resources to consider for advanced integrated testing

At DataMade, we are actively trying to improve our testing protocols. We hope to consider more robust ways of Jasmine integration with our python apps, as suggested by these sources:

* [django-jasmine](https://github.com/jakeharding/django-jasmine)
* [django.js for Jasmine views](http://djangojs.readthedocs.io/en/latest/test.html)

