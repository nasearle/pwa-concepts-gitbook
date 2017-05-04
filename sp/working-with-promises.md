# Working with Promises




## Contents




[<strong>Introduction</strong>](#intro)<strong>        </strong>

[<strong>Why use promises?</strong>](#why)<strong>        </strong>

[<strong>Promise terminology</strong>](#terms)<strong>        </strong>

[<strong>How to use promises</strong>](#howto)<strong>        </strong>

[<strong>Further reading</strong>](#further)

Codelab:  [Promises](https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_promises.html)

<a id="intro" />


## Introduction




[Promises](https://developers.google.com/web/fundamentals/getting-started/primers/promises) offer a better way to handle asynchronous code in JavaScript. Promises have been around for a while in the form of libraries, such as:

*  [Q](https://github.com/kriskowal/q)
*  [when](https://github.com/cujojs/when)
*  [WinJS](http://msdn.microsoft.com/en-us/library/windows/apps/br211867.aspx)
*  [RSVP.js](https://github.com/tildeio/rsvp.js)

The promise libraries listed above and promises that are part of the ES2015 JavaScript specification (also referred to as ES6) are all   [Promises/A+](https://github.com/promises-aplus/promises-spec) compatible. 

See  [Can I Use](http://caniuse.com/#feat=promises) for an up-to-date list of browsers that support promises.

<a id="why" />


## Why use promises?




Asynchronous APIs are common in JavaScript to access the network or disk, to communicate with web workers and service workers, and even when using a timer. Most of these APIs use callback functions or events to communicate when a request is ready or has failed. While these techniques worked well in the days of simple web pages, they don't scale well to complete web applications.

### The old way: using events

Using events to report asynchronous results has some major drawbacks:

* It fragments your code into many pieces scattered among event handlers.
* It's possible to get into race conditions between defining the handlers and receiving the events.
* It often requires creating a class or using globals just to maintain state.

These make error handling difficult. For an example, look at any XMLHttpRequest code.

### The old way: using callbacks

Another solution is to use callbacks, typically with anonymous functions. An example might look like the following:

<code></code>`
function isUserTooYoung(id, callback) {
  openDatabase(function(db) {
    getCollection(db, 'users', function(col) {
      find(col, {'id': id}, function(result) {
        result.filter(function(user) {
          callback(user.age < cutoffAge);
        });
      });
    });
  });
}
<code></code>`

The callback approach has two problems: 

* The more callbacks that you use in a callback chain, the harder it is to read and analyze its behavior. 
* Error handling becomes problematic. For example, what happens if a function receives an illegal value and/or throws an exception?

### Using promises

Promises provide a standardized way to manage asynchronous operations and handle errors. The above example becomes much simpler using promises:

<code></code>`
function isUserTooYoung(id) {
  return openDatabase() // returns a promise
  .then(function(db) {return getCollection(db, 'users');})
  .then(function(col) {return find(col, {'id': id});})
  .then(function(user) {return user.age < cutoffAge;});
}
<code></code>`

Think of a promise as an object that waits for an asynchronous action to finish, then calls a second function. You can schedule that second function by calling <code>.then()</code> and passing in the function. When the asynchronous function finishes, it gives its result to the promise and the promise gives that to the next function (as a parameter).

Notice that there are several calls to <code>.then()</code> in a row. Each call to <code>.then()</code> waits for the previous promise, runs the next function, then converts the result to a promise if needed. This lets you painlessly chain synchronous and asynchronous calls. It simplifies your code so much that most new web specifications return promises from their asynchronous methods.

<a id="terms" />


## Promise terminology




When working with promises you may hear terminology commonly associated with callbacks or other asynchronous code.

In the following example, we convert the asynchronous task of setting an image <code>src</code> attribute into a promise.

<code></code>`
function loadImage(url) {
  // wrap image loading in a promise
  return new Promise(function(resolve, reject) {
    // A new promise is "pending"
    var image = new Image();
    image.src = url;
    image.onload = function() {
      // Resolving a promise changes its state to "fulfilled"
      // unless you resolve it with a rejected promise
      resolve(image);
    };
    image.onerror = function() {
      // Rejecting a promise changes its state to "rejected"
      reject(new Error('Could not load image at ' + url));
    };
  });
}
<code></code>`

A promise is in one of these states:

* Pending - The promise's outcome hasn't yet been determined, because the asynchronous operation that will produce its result hasn't completed yet.
* Fulfilled - The operation resolved and the promise has a value.
* Rejected - The operation failed and the promise will never be fulfilled. A failed promise has a reason indicating why it failed.

You may also hear the term  *settled* : it represents a promise that has been acted upon, and is either fulfilled or rejected.

<a id="howto" />


## How to use promises




### Writing a simple promise

Here's a typical pattern for creating a promise:

<code></code>`
var promise = new Promise(function(resolve, reject) {
  // do a thing, possibly async, then...

  if (/* everything turned out fine */) {
    resolve("Stuff worked!");
  }
  else {
    reject(Error("It broke"));
  }
});
<code></code>`

The promise constructor takes one argument—a callback with two parameters: resolve and reject. Do something within the callback, perhaps async, then call resolve if everything worked, or otherwise call reject.

Like <code>throw</code> in plain old JavaScript, it's customary, but not required, to reject with an <code>Error</code> object. The benefit of <code>Error</code> objects is that they capture a stack trace, making debugging tools more helpful.

Here's one way to use that promise:

<code></code>`
promise.then(function(result) {
  console.log("Success!", result); // "Stuff worked!"
}, function(err) {
  console.log("Failed!", err); // Error: "It broke"
});
<code></code>`

The <code>then()</code> method takes two arguments, a callback for a success case, and another for the failure case. Both are optional, so you can add a callback for the success or failure case only.

A more common practice is to use <code>.then()</code> for success cases and <code>.catch()</code> for errors.

<code></code>`
promise.then(function(result) {
  console.log("Success!", result);
}).catch(function(error) {
  console.log("Failed!", error);
})
<code></code>`

There's nothing special about <code>catch()</code>, it's equivalent to <code>then(undefined, func)</code>, but it's more readable. <strong>Note that the two code examples above do not behave the same way</strong>. The latter example is equivalent to:

<code></code>`
promise.then(function(response) {
  console.log("Success!", response);
}).then(undefined, function(error) {
  console.log("Failed!", error);
})
<code></code>`

The difference is subtle, but extremely useful. Promise rejections skip forward to the next <code>then()</code> with a rejection callback (or <code>catch()</code>, since they're equivalent). With <code>then(func1, func2)</code>, <code>func1</code> or <code>func2</code> will be called, never both. But with <code>then(func1).catch(func2)</code>, both will be called if <code>func1</code> rejects, as they're separate steps in the chain. 

### Promise chains: then and catch

We can attach additional functions to a promise using <code>then()</code> and <code>catch()</code> to create a <strong>promise chain</strong>. In a promise chain, the output of one function serves as input for the next.

#### Then

The <code>then()</code> method schedules a function to be called when the previous promise is fulfilled. When the promise is fulfilled, <code>.then()</code> extracts the promise's value (the value the promise resolves to), executes the callback function, and wraps the returned value in a new promise.

Think of <code>then()</code> as the <code>try</code> portion of a <code>try</code>/<code>catch</code> block.

Remember our earlier example that calls several actions in a row:

<code></code>`
function isUserTooYoung(id) {
  return openDatabase() // returns a promise
  .then(function(db) {return getCollection(db, 'users');})
  .then(function(col) {return find(col, {'id': id});})
  .then(function(user) {return user.age < cutoffAge;});
}
<code></code>`

Calling <code>.then()</code> gets the returned value from the previous promise. It returns a promise that can be passed to follow-on functions, or a value that can be acted upon or returned and used as a parameter in follow-on functions. You can chain any number of actions using <code>.then().</code>

#### Catch

Promises also provide a mechanism to simplify error handling. When a promise rejects (or throws an exception), it jumps to the first <code>.catch()</code> call following the error and passes control to its function. 

Think of the part of the chain preceding <code>catch</code> as being wrapped in an implicit <code>try { }</code> block.

In the following example, we load an image using <code>loadImage()</code> and apply a series of conversions using <code>then()</code>. If at any point we get an error (if either the original promise or any of the subsequent steps rejects) we jump to the <code>catch()</code> statement. 

Only the last <code>then()</code> statement will attach the image to the DOM. Until then, we <code>return</code> the same image so that the image will be passed to the next <code>then()</code>.

<code></code>`
function processImage(imageName, domNode) {
  // returns an image for the next step. The function called in
  // the return statement must also return the image.
  // The same is true in each step below.
  return loadImage(imageName)
  .then(function(image) {
    // returns an image for the next step.
    return scaleToFit(150, 225, image);
  })
  .then(function(image) {
    // returns the image for the next step.
    return watermark('Google Chrome', image);
  })
  .then(function(image) {
    // Attach the image to the DOM after all processing has been completed.
    // This step does not need to return in the function or here in the
    // .then() because we are not passing anything on
    showImage(image);
  })
  .catch(function(error) {
    console.log('We had a problem in running processImage', error);
  });
}
<code></code>`

You can use multiple catches in a promise chain to "recover" from errors in a promise chain. For example, the following code continues on with a fallback image if <code>processImage</code> or <code>scaleToFit</code> rejects:

<code></code>`
function processImage(imageName, domNode) {
  return loadImage(imageName)
  .then(function(image) {
    return scaleToFit(150, 225, image);
  })
  .catch(function(error) {
    console.log('Error in loadImage() or scaleToFit()', error);
    console.log('Using fallback image');
    return fallbackImage();
  })
  .then(function(image) {
    return watermark('Google Chrome', image);
  })
  .then(function(image) {
    showImage(image);
  })
  .catch(function(error) {
    console.log('We had a problem with watermark() or showImage()', error);
  });
}
<code></code>`

<div class="note">
<strong>Note:</strong> The promise chain continues executing after a <code>catch()</code> until it reaches the last <code>then()</code> or <code>catch()</code> in the chain.
</div>

### Synchronous operations 

Not all promise-related functions have to return a promise. If the functions in a promise chain are synchronous, they don't need to return a promise.

The <code>scaleToFit</code> function is part of the image processing chain and doesn't return a promise: 

<code></code>`
function scaleToFit(width, height, image) {
  image.width = width;
  image.height = height;
  console.log('Scaling image to ' + width + ' x ' + height);
  return image;
}
<code></code>`

However, this function does need to return the image passed into it so that it can be passed to the next function in the chain. 

### Promise.all

Often we want to take action only after a collection of asynchronous operations have completed successfully.  [<strong>Promise.all</strong>](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) returns a promise that resolves if all of the promises passed into it resolve. If any of the passed-in promises reject, then <code>Promise.all</code> rejects with the reason of the first promise that rejected. This is very useful for ensuring that a group of asynchronous actions complete before proceeding to another step. 

In the example below, <code>promise1</code> and <code>promise2</code> return promises. We want both of them to load before proceeding. Both promises are passing into <code>Promise.all</code>. If either request rejects, then <code>Promise.all</code> rejects with the value of the rejected promise. If both requests fulfill, <code>Promise.all</code> resolves with the values of both promises (as a list).

<code></code>`
var promise1 = getJSON('/users.json');
var promise2 = getJSON('/articles.json');

Promise.all([promise1, promise2]) // Array of promises to complete
.then(function(results) {
  console.log('all data has loaded');
})
.catch(function(error) {
  console.log('one or more requests have failed: ' + error);
});
<code></code>`

<div class="note">
<strong>Note: </strong>Even if an input promise rejects, causing <code>Promise.all</code> to reject, the remaining input promises still settle. In other words, the remaining promises still execute, they simply are not returned by <code>Promise.all</code>.
</div>

<a id="race" />

### Promise.race

Another promise method that you may see referenced is  [Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race). <code>Promise.race</code> takes a list of promises and settles as soon as the first promise in the list settles. If the first promise resolves, <code>Promise.race</code> resolves with the corresponding value, if the first promise rejects, <code>Promise.race</code> rejects with the corresponding reason. The following code shows example usage of <code>Promise.race</code>:  

<code></code>`
Promise.race([promise1, promise2])
.then(function(value) {
  console.log(value);
})
.catch(function(reason) {
  console.log(reason);
});
<code></code>`

If one of the promises resolves first, the <code>then</code> block executes and logs the value of the resolved promise. If one of the promises rejects first, the <code>catch</code> block executes and logs the reason for the promise rejection.

It may still be tempting, however, to use <code>Promise.race</code> to race promises, as the name suggests. Consider the following example:

<code></code>`
var promise1 = new Promise(function(resolve, reject) {
  // something that fails
});

var promise2 = new Promise(function(resolve, reject) {
  // something that succeeds
});

Promise.race([promise1, promise2])
.then(function(value) {
  // Use whatever returns fastest
})
.catch(function(reason) {
  console.log(reason);
});
<code></code>`

At first glance it looks like this code races two promises—one that rejects, and another that resolves—and uses the first one to return. However, <code>Promise.race</code> rejects immediately if one of the supplied promises rejects, even if another supplied promise resolves later. So if <code>promise1</code> rejects before <code>promise2</code> resolves, <code>promise.All</code> will reject even though <code>promise2</code> supplies a valid value. <code>Promise.race</code> by itself can't be used to reliably return the first promise that resolves.

Another pattern that may be appealing is the following:

<code></code>`
var promise1 = new Promise(function(resolve, reject) {
  // get a resource from the Cache
});

var promise2 = new Promise(function(resolve, reject) {
  // Fetch a resource from the network
});

Promise.race([promise1, promise2])
.then(function(resource) {
  // Use the fastest returned resource
})
.catch(function(reason) {
  console.log(reason);
});
<code></code>`

This example appears to race the cache against the network, using the fastest returned resource. However, both the  [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage) and  [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) can resolve with "bad" responses ( [fetch](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch) resolves even for 404s, and  [caches.match](https://developer.mozilla.org/en-US/docs/Web/API/CacheStorage/match) resolves with falsey values if a resource is not available). In this example, if a resource is not available in the cache (which typically responds faster than the network), <code>Promise.race</code> resolves with the falsey value from the cache, and ignores the network request (which may resolve). See the  [Cache & network race](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#cache-and-network-race) section in the  [Offline Cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/) for an example of a race function that works as expected.

<a id="further" />


## Further reading




*  [JavaScript Promises: an Introduction](https://developers.google.com/web/fundamentals/getting-started/primers/promises)
*  [Promise - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)


