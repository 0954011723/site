title: Plugins
prev: scripts
next: troubleshooting
---
## Overview

Hexo has powerful plugin system. Most of built-in plugins are extensions of Hexo. This makes you easy to hook up your custom code easily.

There're 2 ways to extend Hexo: **Script** and **Plugin**. If your code is simple and doesn't have any dependencies, you can choose **Script**. If your code is complicated and have dependencies, or you want to publish it on NPM registry, you should use **Plugin**.

### Script

To use script, put JavaScript files in `scripts` folder and it'll be loaded once Hexo is initialized.

### Plugin

To use plugin, create a new folder. The folder name must `hexo-` prefixed so it could be loaded by Hexo. The folder must contained 2 files at least. One is your code and the other is `package.json`.

``` plain
.
├── index.js
└── package.json
```

You should at least describe `name`, `version`, `main` and `dependencies` in `package.json`. For example:

``` json
{
  "name": "hexo-my-plugin",
  "version": "0.0.1",
  "main": "index",
  "dependencies": {
  }
}
```

Hexo has 9 categories of plugins:

1. Generator
2. Renderer
3. Helper
4. Deployer
5. Processor
6. Tag
7. Console
8. Migrator
9. Filter

## Generator

Generator generates content based on processed source files.

### Syntax

``` js
extend.generator.register(function(locals, render, callback){
  // ...
});
```

Argument | Description
--- | ---
`locals` | [Site variables](variables.html)
`render` | Render function
`callback` | Callback function

### Render

To render your content with theme, you have to use render function.

``` js
render(path, layout, locals)
```

Argument | Description
--- | ---
`path` | URL without root URL
`layout` | Theme layout to use. This value can be an array or a string. When it's an array, Hexo'll use the first matched layout.
`locals` | Page variables

If you don't need to render your content with theme. You can bind the content on the routes directly. For example:

``` js
route.set('feed.xml', content);
```

### Example

``` js
hexo.extend.generator.register(function(locals, render, callback){
  render('archive.html', ['archive', 'index'], locals);
  callback();
});
```

## Renderer

If you want to use a new markup language in Hexo, you can implement it with renderer plugin.

### Syntax

``` js
hexo.extend.renderer.register(name, output, fn, [sync]);
```

Paramter | Description
--- | ---
`name` | Input file extension (lowercase, not included `.`)
`output` | Output file extension (lowercase, not included `.`)
`fn` | Render function
`sync` | Sync mode (`false` by default)

`fn` is invoked with 3 arguments:

Argument | Description
--- | ---
`data` | `data` contains 2 properties: `path` (File path) and `text` (File content). `path` doesn't always exist in sync mode.
`options` | Options or local variables.
`callback` | Callback function. Only used in async mode.

### Example

**Sync mode:**

``` js
var ejs = require('ejs');

hexo.extend.renderer.register('ejs', 'html', function(data, options){
  options.filename = data.path;
  return ejs.render(data.text, options);
}, true);
```

**Async mode:**

``` js
var stylus = require('stylus');

hexo.extend.renderer.register('styl', 'css', function(data, callback){
  stylus(data.text).set('filename', data.path).render(callback);
});
```

## Helper

Helper is used in theme to help you insert specified content quickly. If you write something in your theme many times. You should consider to put it in the helper. (Don't repeat yourself, right?)

### Syntax

You can get all local variables from `this`.

``` js
hexo.extend.helper.register(name, fn);
```

Paramter | Description
--- | ---
`name` | Helper name
`fn` | Helper function

### Example

``` js
hexo.extend.helper.register('js', function(path){
  return '<script type="text/javascript" src="' + path + '"></script>';
});
```

Input:

``` js
<%- js('script.js') %>
```

Output:

``` html
<script type="text/javascript" src="script.js"></script>
```

## Deployer

Deployer plugin helps you deploy your site on web without complicated commands.

### Syntax

``` js
hexo.extend.deployer.register(name, fn);
```

Parameter | Description
--- | ---
`name` | Deployer name
`fn` | Deployer function

`fn` is invoked with 2 arguments:

Argument | Description
--- | ---
`args` | Deployer options and arguments.
`callback` | Callback function

## Processor

Processor processes raw data and stores them in database. If you want to process some specified files in `source` folder. You can implement it with the processor plugin.

{% note info Hidden files won't be processed %}
Hexo won't process hidden files.
{% endnote %}

### Syntax

``` js
hexo.extend.processor.register([rule], fn);
```

Paramter | Description
--- | ---
`rule` | File path matching rule. This value can be a [Backbone-style route](http://backbonejs.org/#Router), a regular expression or a function.
`fn` | Processor function

`fn` is invoked with 2 arguments:

Argument | Description
--- | ---
`file` | File data (see below)
`callback` | Callback function

### File data

Property | Description
--- | ---
`source` | File source
`path` | File relative path
`type` | File event. The value can be `create`, `update` or `delete`.
`read` | Reads the file
`readSync` | Reads the file synchronizedly
`stat` | Gets the file status
`statSync` | Gets the file status synchronizedly
`render` | Renders the file
`renderSync` | Renders the file synchronizedly

**Reads the file:**

``` js
file.read([options], callback);
file.readSync([options]);
```

Option | Description | Default
--- | --- | ---
`cache` | Enables cache | false
`encoding` | File encoding | utf8

**Gets the file status:**

``` js
file.stat(callback);
file.statSync();
```

**Renders the file:**

``` js
file.render([options], callback);
file.renderSync([options]);
```

## Tag

Tag helps you insert specified content in your post quickly.

### Syntax

``` js
hexo.extend.tag.register(name, fn, [ends]);
```

Parameter | Description
--- | ---
`name` | Tag name
`fn` | Tag function
`options` | Options

`fn` is invoked with 2 arguments:

Argument | Description
--- | ---
`args` | Arguments
`content` | Content (Only available when the tag has end tag)
`options` | Options

### Options

Option | Description | Default
--- | --- | ---
`ends` | Whether a tag must have an end tag | false
`escape` | Escape contents in a tag | true

### Example

**Without end tag:** Inserts a Youtube video.

``` js
hexo.extend.tag.register('youtube', function(args, content, options){
  var id = args[0];
  return '<div class="video-container"><iframe width="560" height="315" src="http://www.youtube.com/embed/' + id + '" frameborder="0" allowfullscreen></iframe></div>';
});
```

**With end tag:** Inserts a pull quote.

``` js
hexo.extend.tag.register('pullquote', function(args, content, options){
  var className = args.length ? ' ' + args.join(' ') : '';
  return '<blockquote class="pullquote' + className + '">' + content + '</blockquote>';
}, true);
```

## Console

Console is the interface between Hexo and you.

### Syntax

``` js
hexo.extend.console.register(name, desc, [options], fn);
```

Parameter | Description
--- | ---
`name` | Console name (lowercase)
`desc` | Console description
`options` | Options
`fn` | Console function. Invoked with a [Optimist] argument.

### Options

Option | Description
--- | ---
`init` | Display in a non-Hexo folder
`debug` | Display in debug mode
`alias` | Console alias

### Example

``` bash
$ hexo config
```

``` js
hexo.extend.console.register('config', 'Display configuration', function(args){
  console.log(hexo.config);
});
```

## Migrator

Migrator helps you migrate from other blog system to Hexo.

### Syntax

``` js
hexo.extend.migrator.register(name, fn);
```

Parameter | Description
--- | ---
`name` | Migrator name
`fn` | Migrator function. Invoked with a [Minimist] argument.

## Filter

Filters are similar with tags, but more powerful and customizable. You can process any attributes such as titles and contents in posts. It's recommended to run asynchronous tasks in filters.

There're two types of filters: **pre-filter** process uncompiled data, while **post-filter** process compiled data.

{% iframe plugins/filter-flow.html 100% 40 %}

### Syntax

``` js
hexo.extend.filter.register([type], fn);
```

Parameter | Description
--- | ---
`type` | Filter type. The value can be `pre` or `post` (Default).
`fn` | Filter function. Invoked with a argument: post data.

### Example

Replaces `#username` in post content to a Twitter link.

``` js
hexo.extend.filter.register('post', function(data, callback){
  data.content = data.content.replace(/#(\d+)/, '<a href="http://twitter.com/$1">#$1</a>');

  callback(null, data);
});
```

[Minimist]: https://github.com/substack/minimist