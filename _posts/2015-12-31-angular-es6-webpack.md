---
layout: post
title:  "Angular + ES6 + Webpack"
date:   2015-12-31 10:14:18
categories: angular webpack
published: false
---

> tl;dr I've created a Yeoman generator to scaffold apps for AngularJS + ES6 + Webpack. Find it [here](https://www.npmjs.com/package/generator-angular-es6-webpack)

 A few months ago a built a small project with React.js and Webpack, and found Webpack to be an extremely pleasant build tool to use. Webpack is a module bundler; it takes your JavaScript, HTML, CSS/Sass and other assets and packs them together into a single JavaScript file. For a more in depth look at Webpack I recommend reading [What is Webpack](http://webpack.github.io/docs/what-is-webpack.html) and the other associated documentation.

 More recently I was involved in reworking the Grunt.js task based build system for some AngularJS applications, and I came to the realisation that the build pipeline we were using was overcomplicated and difficult to understand. There were tasks that had been setup early on that now were unnecessary and had no effect on the result. I moved one of the builds from Grunt to Gulp.js, but found that it was still complicated and only maybe a little faster?

 Then came the opportunity to start a new project and I pioneered an effort to test AngularJS with Webpack. These thoughts are the results of that effort, and I hope you find them useful if you are exploring building application with AngularJS and Webpack. I have also created a Yeoman generator, [generator-angular-es6-webpack](https://www.npmjs.com/package/generator-angular-es6-webpack) which you can use to generator this project structure.

#Setting up Webpack#

Webpack runs predominately on [loaders](https://webpack.github.io/docs/loaders.html). Loaders process certain types of assets, JavaScript but also HTML, CSS, and images, when they are `require()`'d in the project. Webpack will build our bundle by starting at an entry point, usually the `app.js` file, and mapping through all the dependencies loaded with `require()`.

Here is a simple Webpack config file:

{% highlight javascript %}
var webpack = require('webpack');

module.exports = {
  entry: './app/app.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    preLoaders: [
      { test: /\.js$/, loader: 'eslint-loader', exclude: /node_modules/ }
    ],
    loaders: [
      { test: /\.js$/, loader: 'ng-annotate!babel', exclude: /node_modules/ },
      { test: /.html$/, loader: 'ngtemplate!html' },
      { test: /\.scss$/, loaders: ['style', 'css',
          'autoprefixer-loader?browsers=last 2 versions', 'sass'], },
      { test: /\.png$/, loader: 'file-loader' }
    ]
  }
};
{% endhighlight %}

The config defines an entry point file, here `app.js`, that Webpack will start the dependencies tree from. It also defines a set of loaders to process each asset type. Each loader definition has a regex test to match against filename, and then a loader, pipeline of loaders separated by `!`, or an array of loaders.

Pipelines and arrays of loaders are processed from right to left in Webpack, which can be a bit tricky when first getting started. In this example JavaScript files will first be preLoaded through ESLint to lint for errors, they will then be passed through the Babel.js loader to transpile ES6 code to ES5, the resulting code will be passed through the ng-annotate loader to make the AngularJS code minification safe.
