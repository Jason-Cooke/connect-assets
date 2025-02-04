# connect-assets

[![Build Status](https://travis-ci.org/js-kyle/connect-assets.png)](https://travis-ci.org/js-kyle/connect-assets)
![Node version](https://img.shields.io/node/v/connect-assets.png)

Transparent file compilation and dependency management for Node’s [connect](https://github.com/senchalabs/connect) framework in the spirit of the Rails asset pipeline.

## What can it do?

connect-assets can:

1. Serve `.js.coffee` ([CoffeeScript](http://coffeescript.org)) files as compiled `.js`
2. Concatenate `.js.coffee` and `.js` together.
3. Serve `.css.styl` ([Stylus](http://learnboost.github.com/stylus/)) as compiled `.css`
4. Serve `.css.less` ([Less](http://lesscss.org/)) as compiled `.css`
5. Serve `.css.sass` or `.css.scss` ([SASS](http://sass-lang.com)) as compiled `.css`
6. Serve `.jst.jade` ([Jade templates](https://github.com/visionmedia/jade)) as compiled JavaScript functions (be sure to include the Jade runtime — see below).
7. Serve `.jst.ejs` as compiled JavaScript functions.
8. Preprocess `style.css.ejs` and `script.js.ejs` with [EJS](http://embeddedjs.com/) — just append `.ejs` to any file.
9. Serve files with a cache-control token and use a far-future expires header for maximum efficiency.
10. Avoid redundant git diffs by storing compiled `.js` and `.css` files in memory rather than writing them to the disk when in development.

## How do I use it?

First, install it in your project's directory:

```shell
npm install connect-assets
```

Also install any specific compilers you'll need, e.g.:

```shell
npm install coffee-script
npm install stylus
npm install less
npm install node-sass
npm install jade
npm install ejs
```

Then add this line to your app's configuration:

```javascript
app.use(require("connect-assets")());
```

Finally, create an `assets` directory in your project and throw all assets compiled into JavaScript into `/assets/js` and all assets compiled into CSS into `/assets/css`.

### Markup functions

connect-assets provides five global functions named `js`, `jsInline`, `css`, `cssInline` and `assetPath`. Use them in your views. They return the HTML markup needed to include the most recent version of your assets (or, the path to the asset), taking advantage of caching when available. For instance, in a [Jade template](http://jade-lang.com/), the code

```
!= css("normalize")
!= js("jquery")
```

(where `!=` is Jade's syntax for running JS and displaying its output) results in the markup

```html
<link rel="stylesheet" href="/css/normalize-[hash].css" />
<script src="/js/jquery-[hash].js"></script>
```

You can pass a Hash of special attributes to helper method `css` or `js`:

```
!= css("normalize", { 'data-turbolinks-track': true } })
!= js("jquery", { async: true })
```

Results in:

```html
<link rel="stylesheet" href="/css/normalize-[hash].css" data-turbolinks-track />
<script src="/js/jquery-[hash].js" async></script>
```

The inline variants `jsInline` and `cssInline` write the contents straight into the tags, instead of linking. For example,

```
!= cssInline("normalize")
!= jsInline("jquery")
```

(where `!=` is Jade's syntax for running JS and displaying its output) results in the markup

```html
<style>[contents]</style>
<script>[contents]</script>
```

You can also reference image paths via the `assetPath` helper. First, you must specify the
path to your images via the `paths` option e.g:
```javascript
...

var assets = require('connect-assets');

app.use(assets({
  paths: [
    'assets/css',
    'assets/js',
    'assets/img'
  ]
}));
```
You can then use the `assetPath` helper in your Jade like so:
```
img(src="#{assetPath('image-name.png')}")
```

Would result in:
```html
<img src="/assets/img/image-name-[hash].png">
```

### Sprockets-style concatenation

You can indicate dependencies in your `.js.coffee` and `.js` files using the Sprockets-style syntax.

In CoffeeScript:

```coffeescript
#= require dependency
```

In JavaScript:

```javascript
//= require dependency
```

When you do so, and point the `js` function at that file, two things can happen:

1. By default, you'll get multiple `<script>` tags out, in an order that gives you all of your dependencies.
2. If you passed the `build: true` option to connect-assets (enabled by default when `NODE_ENV=production`), you'll just get a single tag, wich will point to a JavaScript file that encompasses the target's entire dependency graph—compiled, concatenated, and minified (with [UglifyJS](https://github.com/mishoo/UglifyJS)).

If you want to bring in a whole folder of scripts, use `//= require_tree dir` instead of `//= require file`.

You can also indicate dependencies in your `.css` files using the Sprockets-style syntax.

```css
/*= require reset.css */

body { margin: 0; }
```

See [Mincer](https://github.com/nodeca/mincer) for more information.

## Options

If you like, you can pass any of these options to the first parameter of the function returned by `require("connect-assets")`:

Option        | Default Value                   | Description
--------------|---------------------------------|-------------------------------
paths         | ["assets/js", "assets/css"]     | The directories that assets will be read from, in order of preference.
helperContext | global                          | The object that helper functions (css, js, assetPath) will be attached to.
servePath     | "assets"                        | The virtual path in which assets will be served over HTTP. If hosting assets locally, supply a local path (say, "assets"). If hosting assets remotely on a CDN, supply a URL: "http://myassets.example.com/assets".
precompile    | ["\*.\*"]                       | An array of assets to precompile while the server is initializing. Patterns should match the filename only, not including the directory.
build         | dev: false; prod: true          | Should assets be saved to disk (true), or just served from memory (false)?
buildDir      | "builtAssets"                   | The directory to save (and load) compiled assets to/from.
compile       | true                            | Should assets be compiled if they don’t already exist in the `buildDir`?
bundle        | dev: false; prod: true          | Should assets be bundled into a single tag (when possible)?
compress      | dev: false; prod: true          | Should assets be minified? If enabled, requires `uglify-js` and `csswring`.
gzip          | false                           | Should assets have gzipped copies in `buildDir`?
fingerprinting| dev: false; prod: true          | Should fingerprints be appended to asset filenames?
sourceMaps    | dev: true; prod: false          | Should source maps be served?

## Custom Configuration of Mincer

This package depends on [mincer](https://github.com/nodeca/mincer), which is quite configurable by design. Many options from mincer are not exposed through connect-assets in the name of simplicity.

As asset compliation happens immediately after connect-assets is initialized, any changes that affect the way mincer compiles assets should be made during initialization. A custom initialization function can be passed to connect-assets as a second argument to the function returned by `require("connect-assets")`:

```javascript
app.use(require("connect-assets")(options, function (instance) {
  // Custom configuration of the mincer environment can be placed here
  instance.environment.registerHelper(/* ... */);
}));
```

## CLI

connect-assets includes a command-line utility, `connect-assets`, which can be used to precompile assets on your filesystem (which you can then upload to your CDN of choice). From your application directory, you can execute it with `./node_modules/.bin/connect-assets [options]`.
```
Usage: connect-assets [-h] [-v] [-gz] [-ap] [-i [DIRECTORY [DIRECTORY ...]]]
                      [-c [FILE [FILE ...]]] [-o DIRECTORY]

Optional arguments:
  -h, --help            Show this help message and exit.
  -v, --version         Show program's version number and exit.
  -i [DIRECTORY [DIRECTORY ...]], --include [DIRECTORY [DIRECTORY ...]]
                        Adds the directory to a list of directories that
                        assets will be read from, in order of preference.
                        Defaults to 'assets/js' and 'assets/css'.
  -c [FILE [FILE ...]], --compile [FILE [FILE ...]]
                        Adds the file (or pattern) to a list of files to
                        compile. Defaults to all files with extensions. Only
                        include the left most extension (ex. main.css).
  -o DIRECTORY, --output DIRECTORY
                        Specifies the output directory to write compiled
                        assets to. Defaults to 'builtAssets'.
  -s PATH, --servePath PATH
                        The virtual path in which assets will be served
                        over HTTP. If hosting assets locally, supply a
                        local path (say, "assets"). If hosting assets
                        remotely on a CDN, supply a URL.
  -gz, --gzip
                        Enables gzip file generation, which is disabled by
                        default.
  -ap, --autoprefixer   Enables autoprefixer during compilation.
  -sm, --sourceMaps     Enables source map generation for all files.
  -emc, --embedMappingComments
                        Embed source map url into compiled files.
  -nsmp, --noSourceMapProtection
                        Do not add XSSI protection header to source map files.
                        https://github.com/adunkman/connect-assets/issues/345#issuecomment-235246691
```
### CLI examples
**Basic use case:**
Compile contents of public/javascripts folder, saving to a the cdn directory.
`connect-assets -i public/javascripts -o cdnassets`

**Advanced use case (nested directories):**
When compiling files which use Sprockets style concatenation e.g. `//= require dependency`, the path to the dependency must also be passed using the `--include` flag.
Consider this project structure:
```
Simple App
│   README.md
│   app.js
└─── public
│   │   robots.txt
│   └─── javascripts
│       │   bundle.js
│       │   sw.js
|       |   client.js
│       └───  app
|               └───  users
│                   |  users.controller.js
│                   |  users.routes.js
└─── test
    │   users.spec.js
```
Contents of bundle.js:
```
//= require users/users.controller.js
//= require users/users.routes.js
```
In the above scenario `connect-assets -i public/javascripts -o cdnassets` will fail to compile `bundle.js` as connect-assets will fail to find the file on the provided path.
To remove errors, ensure that the paths (the same paths as what are defined in your connect-assets options).
For example, `connect-assets -i public/javascripts -i public/javascripts/app -o cdnassets` will successfully pre-compile `bundle.js`.
    
## Serving Assets from a CDN

The CLI utility precompiles assets supplied into their production-ready form, ready for
upload to a CDN or static file server. The generated `manifest.json` is all
that is required on your application server if connect-assets is properly
configured. Once assets have been precompiled and uploaded to CDN (perhaps as part of your build process), you can pass the Mincer environment your manifest file like so:

```
const assetManifest = require('./manifest.json');

app.use(require("connect-assets")(options, function (instance) {
  instance.manifest = assetManifest;
}));
```
Your CDN url will also need to be passed to the `servePath` option of connect-assets.

## Credits

Follows in the footsteps of sstephenson's [Sprockets](https://github.com/sstephenson/sprockets), through the [Mincer](https://github.com/nodeca/mincer) project.

Take a look at the [contributors](https://github.com/adunkman/connect-assets/contributors) who make this project possible.
