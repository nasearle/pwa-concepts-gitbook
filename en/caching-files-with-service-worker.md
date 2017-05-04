# Caching Files with Service Worker




## Contents




<a href="#cacheinsw"><strong>Using the Cache API in the service worker</strong></a>        

<a href="#cacheapi"><strong>Using the Cache API</strong></a>        

<a href="#moreresources"><strong>Further reading</strong></a> 

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_caching_files_with_service_worker.html">Caching Files with Service Worker</a>

<a id="cacheinsw" />


## Using the Cache API in the service worker




The Service Worker API comes with a <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache">Cache interface</a>, that lets you create stores of responses keyed by request. While this interface was intended for service workers it is actually exposed on the window, and can be accessed from anywhere in your scripts. The entry point is <code>caches</code>.

You are responsible for implementing how your script (service worker) handles updates to the cache. All updates to items in the cache must be explicitly requested; items will not expire and must be deleted. 

<a id="whentostore" />

### Storing resources

In this section, we outline a few common patterns for caching resources:  <em>on service worker install</em> ,  <em>on user interaction</em> , and  <em>on network response</em> . There are a few patterns we don't cover here. See the <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/">Offline Cookbook</a> for a more complete list.

#### On install - caching the application shell

We can cache the HTML, CSS, JS, and any static files that make up the application shell in the <code>install</code> event of the service worker:

```
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open(cacheName).then(function(cache) {
      return cache.addAll(
        [
          '/css/bootstrap.css',
          '/css/main.css',
          '/js/bootstrap.min.js',
          '/js/jquery.min.js',
          '/offline.html'
        ]
      );
    })
  );
});
```

This event listener triggers when the service worker is first installed.

<div class="note">
<strong>Note: </strong>It is important to note that while this event is happening, any previous version of your service worker is still running and serving pages, so the things you do here must not disrupt that. For instance, this is not a good place to delete old caches, because the previous service worker may still be using them at this point.
</div>

<a href="https://developer.mozilla.org/en-US/docs/Web/API/ExtendableEvent/waitUntil"><code>event.waitUntil</code></a> extends the lifetime of the <code>install</code> event until the passed promise resolves successfully. If the promise rejects, the installation is considered a failure and this service worker is abandoned (if an older version is running, it stays active). 

<code>cache.addAll</code> will reject if any of the resources fail to cache. This means the service worker will only install if all of the resources in <code>cache.addAll</code> have been cached.

#### On user interaction

If the whole site can't be taken offline, you can let the user select the content they want available offline (for example, a video, article, or photo gallery).

One method is to give the user a "Read later" or "Save for offline" button. When it's clicked, fetch what you need from the network and put it in the cache:

```
document.querySelector('.cache-article').addEventListener('click', function(event) {
  event.preventDefault();
  var id = this.dataset.articleId;
  caches.open('mysite-article-' + id).then(function(cache) {
    fetch('/get-article-urls?id=' + id).then(function(response) {
      // /get-article-urls returns a JSON-encoded array of
      // resource URLs that a given article depends on
      return response.json();
    }).then(function(urls) {
      cache.addAll(urls);
    });
  });
});
```

In the above example, when the user clicks an element with the <code>cache-article</code> class, we are getting the article ID, fetching the article with that ID, and adding the article to the cache.

<div class="note">
<strong>Note:</strong> The Cache API is available on the window object, meaning you don't need to involve the service worker to add things to the cache.
</div>

#### On network response

If a request doesn't match anything in the cache, get it from the network, send it to the page and add it to the cache at the same time.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('mysite-dynamic').then(function(cache) {
      return cache.match(event.request).then(function (response) {
        return response || fetch(event.request).then(function(response) {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    })
  );
});
```

This approach works best for resources that frequently update, such as a user's inbox or article contents. This is also useful for non-essential content such as avatars, but care is needed. If you do this for a range of URLs, be careful not to bloat the storage of your origin — if the user needs to reclaim disk space you don't want to be the prime candidate. Make sure you get rid of items in the cache you don't need any more.

<div class="note">
<strong>Note:</strong> To allow for efficient memory usage, you can only read a response/request's body once. In the code above, <code>.clone()</code> is used to create a copy of the response that can be read separately. See <a href="https://jakearchibald.com/2014/reading-responses/">What happens when you read a response?</a> for more information.
</div>

<a id="servefiles" />

### Serving files from the cache

To serve content from the cache and make your app available offline you need to intercept network requests and respond with files stored in the cache. There are several approaches to this: 

* cache only
* network only 
* cache falling back to network 
* network falling back to cache
* cache then network

There are a few approaches we don't cover here. See Jake Archibald's <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/">Offline Cookbook</a> for a full list.

#### Cache only

You don't often need to handle this case specifically. <a href="#cachefallback">Cache falling back to network</a> is more often the appropriate approach.

This approach is good for any static assets that are part of your app's main code (part of that "version" of your app). You should have cached these in the install event, so you can depend on them being there.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(caches.match(event.request));
});
```

If a match isn't found in the cache, the response will look like a connection error.

#### Network only

This is the correct approach for things that can't be performed offline, such as analytics pings and non-GET requests. Again, you don't often need to handle this case specifically and the <a href="#cachefallback">cache falling back to network</a> approach will often be more appropriate.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(fetch(event.request));
});
```

Alternatively, simply don't call <code>event.respondWith</code>, which will result in default browser behaviour.

<a id="cachefallback" />

#### Cache falling back to the network

If you're making your app offline-first, this is how you'll handle the majority of requests. Other patterns will be exceptions based on the incoming request.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      return response || fetch(event.request);
    })
  );
});
```

This gives you the "Cache only" behavior for things in the cache and the "Network only" behaviour for anything not cached (which includes all non-GET requests, as they cannot be cached).

#### Network falling back to the cache

This is a good approach for resources that update frequently, and are not part of the "version" of the site (for example, articles, avatars, social media timelines, game leader boards). Handling network requests this way means the online users get the most up-to-date content, and offline users get an older cached version.

However, this method has flaws. If the user has an intermittent or slow connection they'll have to wait for the network to fail before they get content from the cache. This can take an extremely long time and is a frustrating user experience. See the next approach, <a href="#cachethen">Cache then network</a>, for a better solution.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request).catch(function() {
      return caches.match(event.request);
    })
  );
});
```

Here we first send the request to the network using <code>fetch()</code>, and only if it fails do we look for a response in the cache. 

<a id="cachethen" />

#### Cache then network

This is also a good approach for resources that update frequently. This approach will get content on screen as fast as possible, but still display up-to-date content once it arrives.

This requires the page to make two requests: one to the cache, and one to the network. The idea is to show the cached data first, then update the page when/if the network data arrives.

Here is the code in the page:

```
var networkDataReceived = false;

startSpinner();

// fetch fresh data
var networkUpdate = fetch('/data.json').then(function(response) {
  return response.json();
}).then(function(data) {
  networkDataReceived = true;
  updatePage(data);
});

// fetch cached data
caches.match('/data.json').then(function(response) {
  if (!response) throw Error("No data");
  return response.json();
}).then(function(data) {
  // don't overwrite newer network data
  if (!networkDataReceived) {
    updatePage(data);
  }
}).catch(function() {
  // we didn't get cached data, the network is our last hope:
  return networkUpdate;
}).catch(showErrorMessage).then(stopSpinner());
```

We are sending a request to the network and the cache. The cache will most likely respond first and, if the network data has not already been received, we update the page with the data in the response. When the network responds we update the page again with the latest information.

Here is the code in the service worker:

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('mysite-dynamic').then(function(cache) {
      return fetch(event.request).then(function(response) {
        cache.put(event.request, response.clone());
        return response;
      });
    })
  );
});
```

This caches the network responses as they are fetched.

Sometimes you can replace the current data when new data arrives (for example, game leaderboard), but be careful not to hide or replace something the user may be interacting with. For example, if you load a page of blog posts from the cache and then add new posts to the top of the page as they are fetched from the network, you might consider adjusting the scroll position so the user is uninterrupted. This can be a good solution if your app layout is fairly linear. 

<a id="generic-fallback" />

#### Generic fallback

If you fail to serve something from the cache and/or network you may want to provide a generic fallback. This technique is ideal for secondary imagery such as avatars, failed POST requests, "Unavailable while offline" page.

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // Try the cache
    caches.match(event.request).then(function(response) {
      // Fall back to network
      return response || fetch(event.request);
    }).catch(function() {
      // If both fail, show a generic fallback:
      return caches.match('/offline.html');
      // However, in reality you'd have many different
      // fallbacks, depending on URL & headers.
      // Eg, a fallback silhouette image for avatars.
    })
  );
});
```

The item you fallback to is likely to be an install dependency.

You can also provide different fallbacks based on the network error:

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    // Try the cache
    caches.match(event.request).then(function(response) {
      if (response) {
        return response;
      }
      return fetch(event.request).then(function(response) {
        if (response.status === 404) {
          return caches.match('pages/404.html');
        }
        return response
      });
    }).catch(function() {
      // If both fail, show a generic fallback:
      return caches.match('/offline.html');
    })
  );
});
```

Network response errors do not throw an error in the <code>fetch</code> promise. Instead, <code>fetch</code> returns the response object containing the error code of the network error. This means we handle network errors in a <code>.then</code> instead of a <code>.catch</code>.

<a id="remove" />

### Removing outdated caches

Once a new service worker has installed and a previous version isn't being used, the new one activates, and you get an <code>activate</code> event. Because the old version is out of the way, it's a good time to delete unused caches.

```
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.filter(function(cacheName) {
          // Return true if you want to remove this cache,
          // but remember that caches are shared across
          // the whole origin
        }).map(function(cacheName) {
          return caches.delete(cacheName);
        })
      );
    })
  );
});
```

During activation, other events such as <code>fetch</code> are put into a queue, so a long activation could potentially block page loads. Keep your activation as lean as possible, only using it for things you couldn't do while the old version was active.

<a id="cacheapi" />


## Using the Cache API




Here we cover the Cache API properties and methods.

<a id="checksupport" />

### Checking for support

We can check if the browser supports the Cache API like this:

```
if ('caches' in window) {
  // has support
}
```

<a id="createcache" />

### Creating the cache

An origin can have multiple named Cache objects. To create a cache or open a connection to an existing cache we use the <a href="https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/open"><code>caches.open</code></a> method. 

```
caches.open(cacheName)
```

This returns a promise that resolves to the cache object. <code>caches.open</code> accepts a string that will be the name of the cache.

<a id="workwithdata" />

### Working with data

The Cache API comes with several methods that let us create and manipulate data in the cache. These can be grouped into methods that either create, match, or delete data.

#### Create data

There are three methods we can use to add data to the cache. These are <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache/add"><code>add</code></a>, <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache/addAll"><code>addAll</code></a>, and <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache/put"><code>put</code></a>. In practice, we will call these methods on the cache object returned from <code>caches.open()</code>. For example:

```
caches.open('example-cache').then(function(cache) {
        cache.add('/example-file.html');
});
```

<code>Caches.open</code> returns the <code>example-cache</code> Cache object, which is passed to the callback in <code>.then</code>. We call the <code>add</code> method on this object to add the file to that cache.

<code>cache.add(request)</code> - The add method takes a URL, retrieves it, and adds the resulting response object to the given cache. The key for that object will be the request, so we can retrieve this response object again later by this request.

<code>cache.addAll(requests)</code> - This method is the same as add except it takes an array of URLs and adds them to the cache. If any of the files fail to be added to the cache, the whole operation will fail and none of the files will be added.

<code>cache.put(request, response)</code> - This method takes both the request and response object and adds them to the cache. This lets you manually insert the response object. Often, you will just want to <code>fetch()</code> one or more requests and then add the result straight to your cache. In such cases you are better off just using <code>cache.add</code> or <code>cache.addAll</code>, as they are shorthand functions for one or more of these operations:

```
fetch(url).then(function (response) {
  return cache.put(url, response);
})
```

#### Match data

There are a couple of methods to search for specific content in the cache: <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache/match"><code>match</code></a> and <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache/matchAll"><code>matchAll</code></a>. These can be called on the <code>caches</code> object to search through all of the existing caches, or on a specific cache returned from <code>caches.open()</code>.

<code>caches.match(request, options)</code> -  This method returns a Promise that resolves to the response object associated with the first matching request in the cache or caches. It returns <code>undefined</code> if no match is found. The first parameter is the request, and the second is an optional list of options to refine the search. Here are the options as defined by MDN:

* <code>ignoreSearch</code>: A Boolean that specifies whether to ignore the query string in the URL.  For example, if set to <code>true</code> the <code>?value=bar</code> part of <code>http://foo.com/?value=bar</code> would be ignored when performing a match. It defaults to <code>false</code>.
* <code>ignoreMethod</code>: A Boolean that, when set to <code>true</code>, prevents matching operations from validating the Request HTTP method (normally only GET and HEAD are allowed.) It defaults to false.
* <code>ignoreVary</code>: A Boolean that when set to <code>true</code> tells the matching operation not to perform VARY header matching — that is, if the URL matches you will get a match regardless of whether the Response object has a VARY header. It defaults to <code>false</code>.
* <code>cacheName</code>: A DOMString that represents a specific cache to search within. Note that this option is ignored by <code>Cache.match()</code>.

<code>caches.matchAll(request, options)</code> -  This method is the same as <code>.match</code> except that it returns all of the matching responses from the cache instead of just the first. For example, if your app has cached some images contained in an image folder, we could return all images and perform some operation on them like this:

```
caches.open('example-cache').then(function(cache) {
  cache.matchAll('/images/').then(function(response) {
    response.forEach(function(element, index, array) {
      cache.delete(element);
    });
  });
})
```

#### Delete data

We can delete items in the cache with <code>cache.delete(request, options)</code>. This method finds the item in the cache matching the request, deletes it, and returns a Promise that resolves to <code>true</code>. If it doesn't find the item, it resolves to false. It also has the same optional options parameter available to it as the match method.

#### Retrieve keys

Finally, we can get a list of cache keys using <code>cache.keys(request, options)</code>. This returns a Promise that resolves to an array of cache keys. These will be returned in the same order they were inserted into the cache. Both parameters are optional. If nothing is passed, <code>cache.keys</code> returns all of the requests in the cache. If a request is passed, it returns all of the matching requests from the cache. The options are the same as those in the previous methods.

The keys method can also be called on the caches entry point to return the keys for the caches themselves. This lets you purge outdated caches in one go.

<a id="moreresources" />


## Further reading




#### Learn about the Cache API

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache">Cache</a> - MDN
* <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/">The Offline Cookbook</a>

#### Learn about using service workers

* <a href="https://developers.google.com/web/fundamentals/getting-started/primers/service-workers">Using Service Workers</a>


