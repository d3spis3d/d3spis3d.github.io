---
layout: post
title:  "Angular + ES6 + Webpack"
date:   2016-01-06 10:14:18
categories: angular webpack
published: true
---

> tl;dr I've created a Yeoman generator to scaffold apps for AngularJS + ES6 + Webpack. Find it [here](https://www.npmjs.com/package/generator-angular-es6-webpack)

 Recently I have been undertaking a project, using AngularJS and Webpack. At first the AngularJS framework does not seem entirely suitable for use with Webpack, but with a different approach than I would typically take to AngularJS application structure, they play nicely. These thoughts are the results of that effort, and I hope you find them useful if you are exploring building application with AngularJS and Webpack. I have also created a Yeoman generator, [generator-angular-es6-webpack](https://www.npmjs.com/package/generator-angular-es6-webpack) which you can use to generate this project structure.

#Setting up Webpack#

Webpack runs predominately on [loaders](https://webpack.github.io/docs/loaders.html). Loaders process certain types of assets, not only JavaScript but also HTML, CSS, and images, when they are `require()`'d in the project. Webpack will build our bundle by starting at an entry point, usually the `app.js` file, and mapping through all the dependencies loaded with `require()`.

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

Pipelines and arrays of loaders are processed from right to left in Webpack, which can be a bit tricky when first getting started. In this example JavaScript files will first be preLoaded through ESLint to lint for errors, they will then be loaded through the Babel.js loader to transpile ES6 code to ES5, the resulting code will be passed through the ng-annotate loader to make the AngularJS code minification safe.

#Struturing the AngularJS app#

{% highlight javascript %}
project
  |_ app
     |_ components
     |_ services
     |_ directives
     |_ images
     |_ app.js
  |_ test
{% endhighlight %}

Above is the project layout that I recommend for AngularJS applications with Webpack. The components folder will holder the controllers, templates, and stylesheets for each section of the application, and each section of the application will have it's own routes file. Services and directives are separated from the main AngularJS module and will be loaded as dependencies to the main module. In app.js we setup the main AngularJS module, import the routes files for each component of the application and import the services and directives modules.

#Our first component#

Lets start with a sample component, a home page with a welcome message.

{% highlight javascript %}
// app/components/home/home-controller.js
export default /*@ngInject*/ function() {
  this.welcomeMessage = 'Welcome to our sample application';
}
{% endhighlight %}

{% highlight html %}
<!-- app/components/home/home.html -->
<header>
  <h1>Angular + ES6 + Webpack Application</h1>
</header>
<section>
  <p>
    { { homeCtrl.welcomeMessage } }
  </p>
</section>
{% endhighlight %}

{% highlight css %}
/* app/components/home/home.scss */
header {
  background-color: gray;
}
section {
    color: red;
}
{% endhighlight %}

{% highlight javascript %}
// app/components/home/home.routes.js
import mainCtrl from 'components/home/home-controller';
import 'components/home/home.html';
import 'components/home/home.scss';

export default /*@ngInject*/ function($stateProvider) {
  $stateProvider.state('home', {
      url: '/',
      views: {
        '@': {
          controller: mainCtrl,
          controllerAs: 'mainCtrl',
          templateUrl: '/components/home/home.html'
        }
      }
    });
}
{% endhighlight %}

The controller and routes config functions are defined as the default export of ES6 modules. The routes config import the controller and associated template, and stylesheet. These exports/imports will be converted to CommonJS modules and `require()` calls by Babel.js.

#Services#

As mentioned previously, the services in our application will be separated out into another Angular module. This allows the sharing of common services between applications, as we can import only the services we need from a common repository and add them to our services module.

Let's create a basic service to fetch weather data, and setup the services module.

{% highlight javascript %}
// app/services/weather-service.js
export default /*@ngInject*/ function($resource) {
  return $resource('/api/weather/:location', {}, {
    get: {
      method: 'GET'
    }
  });
}
{% endhighlight %}

{% highlight javascript %}
// app/services/services.js
import angular from 'angular';

import weather from 'services/weather-service';

angular.module('services', [])
.service('Weather', weather);
{% endhighlight %}

#Main module#

Now that we have a component and the services module we can look at setting up the main application module. In the main app module we will import all component routes files, services and directives modules (I will leave the directives module up to the reader as it is very similar to the services module), and set up any other application wide configuration that we need, http interceptors etc.

{% highlight javascript %}
// app/app.js
import 'jquery';
import angular from 'angular';
import 'angular-resource';
import 'angular-ui-router';

import 'services/services';
import 'directives/directives';

import homeRoutes from 'componets/home/home.routes';

angular.module('ngES6WebpackApp',
  [ngResource, 'ui.router', 'services', 'directives'])
.config(homeRoutes);
{% endhighlight %}

We can now also go back and update our home component to use our weather service.

{% highlight javascript %}
// app/components/home/home-controller.js
export default /*@ngInject*/ function(Weather) {
  this.welcomeMessage = 'Welcome to our sample application';
  Weather.get({location: 'Melbourne'}).$promise.then((weatherData) => {
      this.weather = weatherData;
  });
}
{% endhighlight %}

{% highlight html %}
<!-- app/components/home/home.html -->
<header>
  <h1>Angular + ES6 + Webpack Application</h1>
</header>
<section>
  <p>
    { { homeCtrl.welcomeMessage } }
  </p>
</section>
<aside>
  <p>
    { { homeCtrl.weather.temperture} } - { { homeCtrl.weather.forecast } }
  </p>
</aside>
{% endhighlight %}

#Conclusion#

I hope this post has given you some ideas on how to structure AngularJS apps for bundling with Webpack. As I mentioned above, I have created a Yeoman generator for this application structure, [generator-angular-es6-webpack](https://www.npmjs.com/package/generator-angular-es6-webpack). If you're interested in using these tools try it out. It will scaffold additional features not covered here, like a full Webpack configuration with development server, Gulp tasks, and unit tests with Karma.

tags: angularjs, webpack, es6, es2015 , babeljs
