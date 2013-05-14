# browser-bundle-deps

walk the dependency graph of a browser bundle entry file and return it as a json stream. Supports both AMD and CommonJS (node-style) modules.

# example

``` js
var bdeps = require('browser-module-deps');
var JSONStream = require('JSONStream');

var stringify = JSONStream.stringify();
stringify.pipe(process.stdout);

var file = __dirname + '/files/main.js';
mdeps(file).pipe(stringify);
```

output:

```
$ node example/deps.js
[
{"id":"/data/dev/browser-bundle-deps/test/files/main.js","source":"define([\"./baz\"], function(foo) {\n    console.log('main: ' + foo(5));\n});\n","format":"amd","deps":{"./baz":"/data/dev/browser-bundle-deps/test/files/baz.js"},"entry":true}
,
{"id":"/data/dev/browser-bundle-deps/test/files/baz.js","source":"define([\"./bar\"], function(bar) {\n    require([\"./foo\"], function() {\n        return function (n) {\n            return bar(n) * foo(n);\n        };\n    });\n});\n","format":"amd","deps":{"./foo":"/data/dev/browser-bundle-deps/test/files/foo.js","./bar":"/data/dev/browser-bundle-deps/test/files/bar.js"}}
,
{"id":"/data/dev/browser-bundle-deps/test/files/bar.js","source":"module.exports = function (n) {\n    return n * 100;\n};\n","format":"commonJS","deps":{}}
,
{"id":"/data/dev/browser-bundle-deps/test/files/foo.js","source":"var bar = require('./bar');\n\nmodule.exports = function (n) {\n    return n * 111 + bar(n);\n};\n","format":"commonJS","deps":{"./bar":"/data/dev/browser-bundle-deps/test/files/bar.js"}}
]
```

# usage

```
usage: browser-bundle-deps [files]

  generate json output from each entry file

```

# methods

``` js
var bdeps = require('browser-bundle-deps')
```

## bdeps(files, opts={})

Return a readable stream of javascript objects from an array of filenames
`files`.

Optionally pass in some `opts`:

* opts.transform - a string or array of string transforms (see below)

* opts.transformKey - an array path of strings showing where to look in the
package.json for source transformations. If falsy, don't look at the
package.json at all.

* opts.resolve - custom resolve function using the
`opts.resolve(id, parent, cb)` signature that
[browser-resolve](https://github.com/shtylman/node-browser-resolve) has

* opts.filter - a function (id) to skip resolution of some module `id` strings.
If defined, `opts.filter(id)` should return truthy for all the ids to include
and falsey for all the ids to skip.

* opts.packageFilter - transform the parsed package.json contents before using
the values. `opts.packageFilter(pkg)` should return the new `pkg` object to use.

# transforms

browser-bundle-deps can be configured to run source transformations on files before
parsing them for `require()` calls. These transforms are useful if you want to
compile a language like [coffeescript](http://coffeescript.org/) on the fly or
if you want to load static assets into your bundle by parsing the AST for
`fs.readFileSync()` calls.

If the transform is a function, it should take the `file` name as an argument
and return a through stream that will be written file contents and should output
the new transformed file contents.

If the transform is a string, it is treated as a module name that will resolve
to a module that is expected to follow this format:

``` js
var through = require('through');
module.exports = function (file) { return through() };
```

You don't necessarily need to use the
[through](https://github.com/dominictarr/through) module to create a
readable/writable filter stream for transforming file contents, but this is an
easy way to do it.

When you call `bdeps()` with an `opts.transform`, the transformations you
specify will not be run for any files in node_modules/. This is because modules
you include should be self-contained and not need to worry about guarding
themselves against transformations that may happen upstream.

Modules can apply their own transformations by setting a transformation pipeline
in their package.json at the `opts.transformKey` path. These transformations
only apply to the files directly in the module itself, not to the module's
dependants nor to its dependencies.

# install

With [npm](http://npmjs.org), to get the module do:

```
npm install browser-bundle-deps
```

and to get the `browser-bundle-deps` command do:

```
npm install -g browser-bundle-deps
```

# license

MIT
