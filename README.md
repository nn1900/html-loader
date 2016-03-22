# html loader for webpack

Exports HTML as string. HTML is minimized when the compiler demands.

By default every local `<img src="image.png">` is required (`require("./image.png")`).
You may need to specify loaders for images in your configuration (recommended `file-loader` or `url-loader`).

You can specify which tag-attribute combination should be processed by this loader
via the query parameter `attrs`. Pass an array or a space-separated list of `<tag>:<attribute>` combinations.
(Default: `attrs=img:src`)

To completely disable tag-attribute processing (for instance, if you're handling image loading on the client side) you can pass in `attrs=false`.

You can also convert relative urls to absolute urls as you desire.

## Usage

[Documentation: Using loaders](http://webpack.github.io/docs/using-loaders.html)

## Examples

With this configuration:

``` javascript
{
	module: { loaders: [
		{ test: /\.jpg$/, loader: "file-loader" },
		{ test: /\.png$/, loader: "url-loader?mimetype=image/png" }
	]},
	output: {
		publicPath: "http://cdn.example.com/[hash]/"
	}
}
```

``` html
<!-- fileA.html -->
<img  src="image.jpg"  data-src="image2x.png" >
```

``` javascript
require("html2!./fileA.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg"  data-src="image2x.png" >'

require("html2?attrs=img:data-src!./file.html");
// => '<img  src="image.png"  data-src="data:image/png;base64,..." >'

require("html2?attrs=img:src img:data-src!./file.html");
require("html2?attrs[]=img:src&attrs[]=img:data-src!./file.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg"  data-src="data:image/png;base64,..." >'

require("html2?-attrs!./file.html");
// => '<img  src="image.jpg"  data-src="image2x.png" >'

/// minimized by running `webpack --optimize-minimize`
// => '<img src=http://cdn.example.com/49e...ba9f/a9f...92ca.jpg data-src=data:image/png;base64,...>'

```

## 'Root-relative' urls

For urls that start with a `/`, the default behavior is to not translate them.
If a `root` query parameter is set, however, it will be prepended to the url
and then translated.

With the same configuration above:
``` html
<!-- fileB.html -->
<img src="/image.jpg">
```

``` javascript

require("html2!./fileB.html");
// => '<img  src="/image.jpg">'

require("html2?root=.!./fileB.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg">'

```

## Interpolation

You can use `interpolate` flag to enable interpolation syntax for ES6 template strings, like so:

```
require("html2?interpolate!./file.html");
```

```
<img src="${require(`./images/gallery.png`)}" />
<div>${require('./partials/gallery.html')}</div>
```

## Advanced options

If you need to pass [more advanced options](https://github.com/webpack/html-loader/pull/46), especially those which cannot be stringified, you can also define an `htmlLoader`-property on your `webpack.config.js`:

``` javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html2"
      }
    ]
  }
  htmlLoader: {
  	ignoreCustomFragments: [/\{\{.*?}}/]
  }
};
```

If you need to define two different loader configs, you can also change the config's property name via `html2?config=otherHtmlLoaderConfig`:

```javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html2?config=otherHtmlLoaderConfig"
      }
    ]
  }
  otherHtmlLoaderConfig: {
    ...
  }
};
```

If you need to convert relative urls to absolute urls
(in case you're developing an HTML5 app, and the html file you load contains user uploaded images),
you can match the urls you want to convert via `test` config property,
and via `rootUrl` config property to specify how you want to convert those interested urls:
```javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html2"
      }
    ]
  }
  htmlLoader: {
		test: /^upload\//i,
		rootUrl: 'http://yourserver.com/'
  }
};
```

you can also specify `rootUrl` as a function so that you can provide different root urls as according to different deployment target:
```javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html2"
      }
    ]
  }
  htmlLoader: {
		test: /^upload\//i,
		rootUrl: function() {
			return process.env.TARGET == 'app' ? 'http://yourserver.com' : ''
		}
  }
};
```

## License

MIT (http://www.opensource.org/licenses/mit-license.php)
