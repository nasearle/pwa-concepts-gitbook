# Using sw-precache and sw-toolbox




## Contents




<a href="#intro"><strong>Introduction</strong></a> 

<a href="#routes"><strong>Creating routes with sw-toolbox</strong></a> 

<a href="#gulp"><strong>Using sw-precache and sw-toolbox in a gulp build file</strong></a> 

<a href="#cmdline"><strong>Running sw-precache from the command line</strong></a> 

<a href="#further"><strong>Further reading</strong></a>

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_sw-precache_and_sw-toolbox.html">sw-precache and sw-toolbox</a>

<a id="intro">


## Introduction




In this text we'll cover <code>sw-precache</code> and <code>sw-toolbox</code>, two Node packages created by Google to automate the creation of service workers, and to make the creation of custom caching routes easier. We'll explore how to use <code>sw-precache</code> from the command line, how to build routes using <code>sw-toolbox,</code> and how to integrate both tools into a gulp-based workflow.

<a id="routes">


## Creating routes with sw-toolbox




<code>sw-toolbox</code> simplifies the process of intercepting network requests in the service worker and performing some caching strategy with the request/response.

Before you can use <code>sw-toolbox</code>, you must first install the package in your project from the command line:
```
npm install sw-toolbox
```

To use <code>sw-toolbox</code> you define  <em>routes</em>  and include them in your service worker. Routes behave like <code>fetch</code> event listeners, but are a more convenient way of creating custom handlers for specific requests.

Routes look like this:
```
toolbox.router.get(urlPattern, handler, options)
```

A route intercepts requests that match the specified URL pattern and HTTP request method, and responds according to the rules defined in the request handler. The HTTP request method is called on <code>toolbox.router</code> (in the example above it's <code>get</code>) and can be any of the methods defined <a href="https://googlechrome.github.io/sw-toolbox/api.html#main">here</a>. The <code>options</code> parameter lets us define a cache to use for that route, as well as a network timeout if the handler is the built-in <code>toolbox.networkFirst</code>. See the <a href="https://googlechrome.github.io/sw-toolbox/api.html#main">Tutorial: API</a> for more details.

Routes can be added directly to an existing service worker, or written in a separate JavaScript file and imported into the service worker using the <code>importScripts</code> method. Before the routes can be used, the <code>sw-toolbox</code> Node script itself must also be imported into the service worker from the <strong>node_modules</strong> folder. For example, you could include the following snippet at the bottom of your service worker to import both the <code>sw-toolbox</code> package and a JavaScript file containing your custom routes:

importScripts("./node_modules/sw-toolbox/sw-toolbox.js","./js/toolbox-script.js");

<div class="note">
Note: Be sure to include the sw-toolbox package as the first argument in importScripts, so that your custom routes' dependencies exist in the service worker before being called.
</div>

### Built-in Handlers

<code>sw-toolbox</code> has five built-in handlers to cover the most common caching strategies (see the <a href="https://googlechrome.github.io/sw-toolbox/api.html#main">Tutorial: API</a> for the full list and the <a href="#strategies">Caching strategies table</a> below for a quick reference). For more information about caching strategies see the <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/">Offline Cookbook</a>.

Let's look at an example:
```
toolbox.router.get('/my-app/index.html', global.toolbox.networkFirst, {networkTimeoutSeconds: 5});
```

This intercepts all <code>GET</code> requests for <strong>/my-app/index.html</strong> and handles the request according to the built-in "network first" strategy. In this approach the request is first sent to the network, and if that succeeds the request/response pair is added to the cache. If it fails, it tries to get the response from the cache. We have set the <code>networkTimeoutSeconds</code> option to <code>5</code> so that the app fetches the response from the cache if the network doesn't respond within 5 seconds.

To define "wildcards" (URL patterns for matching more than one file), or if you need to match a cross-origin request, sw-toolbox has two options: Express style routing and regular expression routing.

#### For more information

* <a href="https://googlechrome.github.io/sw-toolbox/usage.html#main">sw-toolbox Tutorial: Usage</a>
* <a href="https://googlechrome.github.io/sw-toolbox/api.html#main">sw-toolbox Tutorial: API</a>

### Express-style Routing

If you're familiar with <a href="http://expressjs.com/en/guide/routing.html">Express.js</a>, you can write URL patterns using a syntax similar to its routing syntax.

To use Express-style URL patterns, pass the pattern into the route as a string. <code>sw-toolbox</code> then converts the URL to a regular expression via the <a href="https://github.com/pillarjs/path-to-regexp">path-to-regexp</a> library.

For example:
```
toolbox.router.get('img/**/*.{png,jpg}', global.toolbox.cacheFirst);
```

This intercepts all  <code>GET</code> requests for any <code>png</code> or <code>jpg</code> file under the <strong>img</strong> folder, regardless of depth. It handles the request according to the "cache first" strategy, first looking in the cache for the response. If that fails, the request is sent to the network and, if that succeeds, the response is added to the cache.

Here is another example:
```
toolbox.router.get('/.*fly$/', global.toolbox.cacheFirst);
```

This matches any content that ends with fly (like butterfly or dragonfly) using the cache first strategy:

To match a request from another domain using Express-style routing, we must define the <code>origin</code> property in the <code>options</code> object. The value could be either a string (which is checked for an exact match) or a RegExp object. In both cases, it's matched against the full origin of the URL (for example, <strong>https://<span></span>example.com</strong>).

For example:
```
toolbox.router.get('/(.*)', global.toolbox.cacheFirst, {
  origin: /\.googleapis\.com$/
});
```

This matches all files (<code>'/(.*)'</code>) with an origin that ends with ".googleapis.com", and uses the "cache first" strategy.

### Regular Expression Routing

You can also use <a href="https://regex101.com/">regular expressions</a> to define the URL pattern in the route by passing a <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions">RegExp</a> object as the first parameter. This RegExp is matched against the full request URL (request and path) to determine if the route applies to the request. This matching makes for easier cross-origin routing, since the origin and the path are matched without having to specify an <code>origin</code> like we did with Express style routes.

This route handles all GET requests that end with <strong>index.html</strong>:
```
toolbox.router.get(/index.html$/, function(request) {
  return new Response('Handled a request for ' + request.url);
});
```

This route handles all POST requests that begin with <strong>https://<span></span>api.flickr.com/</strong>: 
```
toolbox.router.post(/^https://api.flickr.com\//, global.toolbox.networkFirst);
```

### Cache Control

<code>sw-toolbox</code> also gives us the ability to control the cache and its characteristics. The Express route below handles requests ending with googleapis.com using the cache first strategy. In addition to handling the origin we customize the cache itself. 

* We give it a name ("googleapis")
* We give it a maximum size of 10 items (indicated by the <code>maxEntries</code> parameter)
* We set the content to expire in 86400 seconds (24 hours)
```
toolbox.router.get('/(.*)', global.toolbox.cacheFirst, {
  cache: {
    name: 'googleapis',
    maxEntries: 10,
    maxAgeSeconds: 86400
  },
  origin: /\.googleapis\.com$/
});
```

<a id="strategies">

### Caching strategies with sw-toolbox

It's important to consider all of the caching strategies and find the right balance between speed and data freshness for each of your data sources. Use the following table to determine which caching strategy is most appropriate for the dynamic resources that populate your app shell.  

<strong>Table of Common Caching Strategies</strong>

<table markdown="1">
<tr><td colspan="1" rowspan="1">
<p><strong>Strategy</strong></p>
</td><td colspan="1" rowspan="1">
<p><strong>The service worker ...</strong></p>
</td><td colspan="1" rowspan="1">
<p><strong>Best use of this strategy ....</strong></p>
</td><td colspan="1" rowspan="1">
<p><strong>Corresponding </p>
<p>sw-toolbox handler </strong></p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>Cache first,</p>
<p>Network fallback</p>
</td><td colspan="1" rowspan="1">
<p>Loads the local (cached) HTML and JavaScript first, if possible, bypassing the network. If cached content is not available, then the service worker returns a response from the network instead. </p>
</td><td colspan="1" rowspan="1">
<p>When dealing with remote resources that are very unlikely to change, such as static images. </p>
</td><td colspan="1" rowspan="1">
<p><code>toolbox.cacheFirst</code></p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>Network first, Cache fallback</p>
</td><td colspan="1" rowspan="1">
<p>Checks the network first for a response and, if successful, returns current data to the page. If the network request fails, then the service worker returns the cached entry instead. </p>
</td><td colspan="1" rowspan="1">
<p>When data must be as fresh as possible, such as for a real-time API response, but you still want to display something as a fallback when the network is unavailable.</p>
</td><td colspan="1" rowspan="1">
<p><code>toolbox.networkFirst</code></p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>Cache/network race</p>
</td><td colspan="1" rowspan="1">
<p>Fires the same request to the network and the cache simultaneously. In most cases, the cached data loads first and is returned directly to the page. Meanwhile, the network response updates the previously cached entry. The cache updates keep the cached data relatively fresh. The updates occur in the background and do not block rendering of the cached content. </p>
</td><td colspan="1" rowspan="1">
<p>When content is updated frequently, such as for articles, social media timelines, and game leaderboards. It can also be useful when chasing performance on devices with slow disk access where getting resources from the network might be quicker than pulling data from cache.</p>
</td><td colspan="1" rowspan="1">
<p><code>toolbox.fastest</code></p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>Network only</p>
</td><td colspan="1" rowspan="1">
<p>Only checks the network. There is no going to the cache for data. If the network fails, then the request fails. </p>
</td><td colspan="1" rowspan="1">
<p>When only fresh data can be displayed on your site. </p>
</td><td colspan="1" rowspan="1">
<p><code>toolbox.networkOnly</code></p>
</td>
</tr>
<tr><td colspan="1" rowspan="1">
<p>Cache only</p>
</td><td colspan="1" rowspan="1">
<p>The data is cached during the install event so you can depend on the data being there.</p>
</td><td colspan="1" rowspan="1">
<p>When displaying static data on your site.</p>
</td><td colspan="1" rowspan="1">
<p><code>toolbox.cacheOnly</code></p>
</td>
</tr></table>


The example below demonstrates some <code>sw-toolbox</code> strategies to cache different parts of an application.
```
(function(global) {
  'use strict';

  // Example 1
  global.toolbox.router.get('/(.*)', global.toolbox.cacheFirst, {
    cache: {
      name: 'googleapis',
      maxEntries: 20,
    },
    origin: /\.googleapis\.com$/
  });

  // Example 2
  global.toolbox.router.get(/\.(?:png|gif|jpg)$/, global.toolbox.cacheFirst, {
    cache: {
      name: imagesCacheName,
      maxEntries: 50
    }
  });

  // SPECIAL CASES:
  // Example 3
  self.addEventListener('fetch', function(event) {
    if (event.request.headers.get('accept').includes('video/mp4')) {
      // respond with a network only request for the requested object
      event.respondWith(global.toolbox.networkOnly(event.request));
    }
    // you can add additional synchronous checks based on event.request.
  });

  // Example 4
  global.toolbox.router.get('(.+)', global.toolbox.networkOnly, {
    origin: /\.(?:youtube|vimeo)\.com$/
  });

  // Example 5
  global.toolbox.router.get('/*', global.toolbox.cacheFirst);
})(self);
```

Example 1 uses a cache first strategy to fetch content from the <strong>googleapis.com</strong> domain. It will store up to 20 matches in the googleapis cache. 

Example 2 uses a cache first strategy to fetch all PNG and JPG images (those files that end with "png" or "jpg") from the <code>images-cache</code> cache. If it can't find the items in the cache, it fetches them from the network and adds them to the <code>images-cache</code> cache. When more than 50 items are stored in the cache, the oldest items are removed. 

Example 3 works with local videos in MP4 and a network only strategy. We don't want large files to bloat the cache so we will only play the videos when we are online. 

Example 4 matches all files from Youtube and Vimeo and uses the network only strategy. When working with external video sources we not only have to worry about cache size but also about potential copyright issues. 

Example 5 presents our default route. If the request did not match any prior routes it will match this one and run with a cache first strategy. 

<a id="gulp">


## Using sw-precache and sw-toolbox in a gulp build file




<code>sw-precache</code> is a module for generating a service worker that precaches resources. It integrates with your build process. <code>sw-precache</code> gives you fine control over the behavior of the generated service worker. At the time of creation we can specify files to precache, scripts to import, and many other options that determine how the service worker behaves (see the <a href="https://github.com/GoogleChrome/sw-precache">sw-precache Github page</a> for more information).

### Integrating sw-precache into a gulp build system

To use <code>sw-precache</code> in gulp, we first import the plugin at the top of the gulp file.
```
var swPrecache = require('sw-precache');
```


We then create a gulp task and call <code>write</code> on <code>swPrecache</code>. The write method looks like this:
```
swPrecache.write(filePath, options, callback)
```

<code>filePath</code> is the location of the file to write the service worker to. <code>options</code> is an object that defines the behavior of the generated service worker (see the <a href="https://github.com/GoogleChrome/sw-precache#options-parameter">documentation on Github</a> for the full list of options). The callback is always executed. This is for gulp to know when an async operation has completed. If there is an error, it is passed to the callback. If no error is found, null is passed to the callback.

Let's look at an example:
```
gulp.task('generate-service-worker', function(callback) {
  swPrecache.write('app/service-worker.js'), {
    //1
    staticFileGlobs: [
       'app/index.html',
       'app/js/main.js',
       'app/css/main.css',
       'app/img/**/*.{svg,png,jpg,gif}'
     ],
    // 2
    importScripts: [
       'app/node_modules/sw-toolbox/sw-toolbox.js',
       'app/js/toolbox-script.js'
     ],
    // 3
    stripPrefix: 'app/'
  }, callback);
});
```

We call the gulp task <code>'generate-service-worker'</code> and pass a callback to the function to make it asynchronous.

<code>swPrecache.write</code> generates a service worker with the following options:

* The resources in <code>staticFileGlobs</code> are precached, meaning the generated service worker will contain an <code>install</code> event handler that caches the resources.
* The scripts in <code>importScripts</code> are included in the generated service worker inside an <code>importScripts</code> method. In the example we are including the <code>sw-toolbox</code> module and a script containing our routes.
* The <code>app/</code> prefix is removed from all file paths in <code>staticFileGlobs</code> so that the paths in the generated service worker are relative.

<div class="note">
<strong>Note:</strong> Incorporating <code>sw-toolbox</code> routes into your build is as simple as including the <code>sw-toolbox</code> module and a script containing your routes in the <code>importScripts</code> option of <code>swPrecache.write</code>.
</div>

<a id="cmdline">


## Running sw-precache from the command line




You can use <code>sw-precache</code> from the command line when you want to test the result of using it, but don't want to have to change your build system for every version of the experiment. 

Sensible defaults are assumed for options that are not provided. For example, if you are inside the top-level directory that contains your site's contents, and you'd like to generate a <strong>service-worker.js</strong> file that will automatically precache all of the local files, you can simply run:
```
sw-precache
```

Alternatively, if you'd like to only precache <code>.html</code> files that live within <strong>dist/</strong>, which is a subdirectory of the current directory, you could run:
```
sw-precache --root=dist --static-file-globs='dist/**/*.html'
```

<div class="note">
<strong>Note:</strong> Be sure to use quotes around parameter values that have special meanings to your shell (such as the * characters in the sample command line above, for example).
</div>

Finally, there's support for passing complex configurations using <code>--config <file></code>. Any of the options from the file can be overridden through a command-line flag. We recommend using an external JavaScript file to define configurations using <a href="https://nodejs.org/api/modules.html#modules_module_exports">module.exports</a>. For example, assume there's a <strong>path/to/sw-precache-config.js</strong> file that contains:
```
module.exports = {
  staticFileGlobs: [
    'app/css/**.css',
    'app/**.html',
    'app/images/**.*',
    'app/js/**.js'
  ],
  stripPrefix: 'app/',
  runtimeCaching: [{
    urlPattern: /this\\.is\\.a\\.regex/,
    handler: 'networkFirst'
  }]
};
```

We can pass the file to the command-line interface, also setting the verbose option:
```
sw-precache --config=path/to/sw-precache-config.js --verbose
```

This provides the most flexibility, such as providing a regular expression for the <code>runtimeCaching.urlPattern</code> option.

<code>sw-precache</code> also supports passing in a JSON file for <code>--config</code>, though this provides less flexibility:
```
{
  "staticFileGlobs": [
    "app/css/**.css",
    "app/**.html",
    "app/images/**.*",
    "app/js/**.js"
  ],
  "stripPrefix": "app/",
  "runtimeCaching": [{
    "urlPattern": "/express/style/path/(.*)",
    "handler": "networkFirst"
  }]
}
```

<a id="further">


## Further reading




#### Documentation

* <a href="https://github.com/GoogleChrome/sw-precache">sw-precache - Github</a>
* <a href="https://www.npmjs.com/package/sw-precache">sw-precache - npm</a>
* <a href="https://github.com/GoogleChrome/sw-toolbox">sw-toolbox - Github</a>
* <a href="https://googlechrome.github.io/sw-toolbox/usage.html#main">sw-toolbox Usage Tutorial</a>
* <a href="https://googlechrome.github.io/sw-toolbox/api.html#main">sw-toolbox API Tutorial</a>

#### URL Patterns

* <a href="http://regexr.com/">RegExr</a>
* <a href="https://expressjs.com/en/guide/routing.html">Express Routing</a>


