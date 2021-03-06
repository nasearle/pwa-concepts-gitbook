# Working with the Fetch API




## Contents




<a href="#whatisfetch"><strong>What is fetch?</strong></a> 

<a href="#makerequest"><strong>Making a request</strong></a> 

<a href="#readresponse"><strong>Reading the response object</strong></a> 

<a href="#makecustomrequest"><strong>Making custom requests</strong></a>

<a href="#cors"><strong>Cross-origin requests</strong></a> 

<a href="#furtherreading"><strong>Further reading</strong></a>

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_fetch_api.html">Fetch API</a>

<a id="whatisfetch" />


## What is fetch?




The <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API">Fetch API</a> is a simple interface for fetching resources. Fetch makes it easier to make web requests and handle responses than with the older <a href="https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest">XMLHttpRequest</a>, which often requires additional logic (for example, for handling redirects). 

<div class="note">
<strong>Note: </strong>Fetch supports the <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS">Cross Origin Resource Sharing (CORS)</a>. Testing generally requires running a local server. Note that although fetch does not require HTTPS, service workers do and so using fetch in a service worker requires HTTPS. Local servers are exempt from this.
</div>

You can check for browser support of fetch in the window interface. For example:

#### main.js
```
if (!('fetch' in window)) {
  console.log('Fetch API not found, try including the polyfill');
  return;
}
// We can safely use fetch from now on
```

There is a <a href="https://github.com/github/fetch">polyfill</a> for <a href="http://caniuse.com/#feat=fetch">browsers that are not currently supported</a> (but see the readme for important caveats.). 

The <a href="https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch">fetch() method</a> takes the path to a resource as input. The method returns a <a href="http://www.html5rocks.com/en/tutorials/es6/promises/">promise</a> that resolves to the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Response">Response</a> of that request. 

<a id="makerequest" />


## Making a request




Let's look at a simple example of fetching a JSON file:

#### main.js
```
fetch('examples/example.json')
.then(function(response) {
  // Do stuff with the response
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

We pass the path for the resource we want to retrieve as a parameter to fetch. In this case this is <strong>examples/example.json</strong>. The fetch call returns a promise that resolves to a response object. 

When the promise resolves, the response is passed to <code>.then</code>. This is where the response could be used. If the request does not complete, <code>.catch</code> takes over and is passed the corresponding error.

Response objects represent the response to a request. They contain the requested resource and useful properties and methods. For example, <code>response.ok</code>, <code>response.status</code>, and <code>response.statusText</code> can all be used to evaluate the status of the response. 

Evaluating the success of responses is particularly important when using fetch because bad responses (like 404s) still resolve. The only time a fetch promise will reject is if the request was unable to complete. The previous code segment would only fall back to .<code>catch</code> if there was no network connection, but not if the response was bad (like a 404). If the previous code were updated to validate responses it would look like:

#### main.js
```
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Do stuff with the response
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

Now if the response object's <code>ok</code> property is false (indicating a non 200-299 response), the function throws an error containing <code>response.statusText</code> that triggers the <code>.catch</code> block. This prevents bad responses from propagating down the fetch chain.

<a id="readresponse" />


## Reading the response object




Responses must be read in order to access the body of the response. Response objects have <a href="https://developer.mozilla.org/en-US/docs/Web/API/Response">methods</a> for doing this. For example, <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body/json">Response.json()</a> reads the response and returns a promise that resolves to JSON. Adding this step to the current example updates the code to:

#### main.js
```
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Read the response as json.
  return response.json();
})
.then(function(responseAsJson) { 
  // Do stuff with the JSON
  console.log(responseAsJson);
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

This code will be cleaner and easier to understand if it's abstracted into functions:

#### main.js
```
function logResult(result) {
  console.log(result);
}

function logError(error) {
  console.log('Looks like there was a problem: \n', error);
}

function validateResponse(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  return response;
}

function readResponseAsJSON(response) {
  return response.json();
}

function fetchJSON(pathToResource) {
  fetch(pathToResource) // 1
  .then(validateResponse) // 2
  .then(readResponseAsJSON) // 3
  .then(logResult) // 4
  .catch(logError);
}

fetchJSON('examples/example.json');
```

(This is <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-concepts/content/docs/working_with_promises.html#chaining">promise chaining</a>.)

 To summarize what's happening:

Step 1. Fetch is called on a resource, <strong>examples/example.json</strong>. Fetch returns a promise that will resolve to a response object. When the promise resolves, the response object is passed to <code>validateResponse</code>.

Step 2. <code>validateResponse</code> checks if the response is valid (is it a 200-299?). If it isn't, an error is thrown, skipping the rest of the <code>then</code> blocks and triggering the <code>catch</code> block. This is particularly important. Without this check bad responses are passed down the chain and could break later code that may rely on receiving a valid response. If the response is valid, it is passed to <code>readResponseAsJSON</code>.

<div class="note">
<strong>Note:</strong> You can also handle any network status code using the <code>status</code> property of the <code>response</code> object. This lets you respond with custom pages for different errors or handle other responses that are not <code>ok</code> (i.e., not 200-299), but still usable (e.g., status codes in the 300 range). See <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-concepts/content/docs/caching-files-with-service-worker.html#generic-fallback">Caching files with the service worker</a> for an example of a custom response to a 404.
</div>

Step 3. <code>readResponseAsJSON</code> reads the body of the response using the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body/json">Response.json()</a> method. This method returns a promise that resolves to JSON. Once this promise resolves, the JSON data is passed to <code>logResult</code>. (Can you think of what would happen if the promise from <code>response.json()</code> rejects?)

Step 4. Finally, the JSON data from the original request to <strong>examples/example.json</strong> is logged by <code>logResult</code>. 

#### For more information

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Response">Response interface</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body/json">Response.json()</a>
* <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-concepts/content/docs/working_with_promises.html#chaining">Promise chaining</a>

### Example: fetching images

Let's look at an example of fetching an image and appending it to a web page. 

#### main.js
```
function readResponseAsBlob(response) {
  return response.blob();
}

function showImage(responseAsBlob) {
  // Assuming the DOM has a div with id 'container'
  var container = document.getElementById('container');
  var imgElem = document.createElement('img');
  container.appendChild(imgElem);
  var imgUrl = URL.createObjectURL(responseAsBlob);
  imgElem.src = imgUrl;
}

function fetchImage(pathToResource) {
  fetch(pathToResource)
  .then(validateResponse)
  .then(readResponseAsBlob)
  .then(showImage)
  .catch(logError);
}

fetchImage('examples/kitten.jpg');
```

In this example an image (<strong>examples/kitten.jpg)</strong> is fetched. As in the previous example, the response is validated with <code>validateResponse</code>. The response is then read as a <a href="https://developer.mozilla.org/en-US/docs/Web/API/Blob">Blob</a> (instead of as JSON), and an image element is created and appended to the page, and the image's <code>src</code> attribute is set to a data URL representing the Blob.

<div class="note">
<strong>Note:</strong> The <a href="https://developer.mozilla.org/en-US/docs/Web/API/URL">URL object's</a> <a href="https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL">createObjectURL() method</a> is used to generate a data URL representing the Blob. This is important to note as you cannot set an image's source directly to a Blob. The Blob must first be converted into a data URL.
</div>

#### For more information

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Blob">Blobs</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body/blob">Response.blob()</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/URL">URL object</a>

### Example: fetching text

Let's look at another example, this time fetching some text and inserting it into the page. 

#### main.js
```
function readResponseAsText(response) {
  return response.text();
}

function showText(responseAsText) {
  // Assuming the DOM has a div with id 'message'
  var message = document.getElementById('message');
  message.textContent = responseAsText;
}

function fetchText(pathToResource) {
  fetch(pathToResource)
  .then(validateResponse)
  .then(readResponseAsText)
  .then(showText)
  .catch(logError);
}

fetchText('examples/words.txt');
```

In this example a text file is being fetched, <strong>examples/words.txt</strong>. Like the previous two exercises, the response is validated with <code>validateResponse</code>. Then the response is read as text, and appended to the page.

<div class="note">
<strong>Note:</strong> It may be tempting to fetch HTML and append that using the <code>innerHTML</code> attribute, but be careful -- this can expose your site to <a href="https://www.google.com/about/appsecurity/learning/xss/">cross site scripting attacks</a>!
</div>

#### For more information

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body/text">Response.text()</a>

<div class="note">
<strong>Note:</strong> For completeness, the methods we have used are actually methods of <a href="https://developer.mozilla.org/en-US/docs/Web/API/Body">Body</a>, a Fetch API <a href="https://developer.mozilla.org/en-US/docs/Glossary/mixin">mixin</a> that is implemented in the Response object.  
</div>

<a id="makecustomrequest" />


## Making custom requests




<code>fetch()</code> can also receive a second optional parameter, <code>init</code>, that allows you to create custom settings for the request, such as the <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods">request method</a>, cache mode, credentials, <a href="https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch">and more</a>.

<a id="head" />

### Example: HEAD requests

By default fetch uses the GET method, which retrieves a specific resource, but other request HTTP methods can also be used.

HEAD requests are just like GET requests except the body of the response is empty. You can use this kind of request when all you want the file's metadata, and you want or need the file's data to be transported.

To call an API with a HEAD request, set the method in the <code>init</code> parameter. For example:

#### main.js
```
fetch('examples/words.txt', {
  method: 'HEAD'
})
```

This will make a HEAD request for <strong>examples/words.txt</strong>. 

You could use a HEAD request to check the size of a resource. For example:

#### main.js
```
function checkSize(response) {
  var size = response.headers.get('content-length');
  // Do stuff based on response size
}

function headRequest(pathToResource) {
  fetch(pathToResource, {
    method: 'HEAD'
  })
  .then(validateResponse)
  .then(checkSize)
  // ...
  .catch(logError);
}

headRequest('examples/words.txt');
```

Here the HEAD method is used to request the size (in bytes) of a resource (represented in the <strong>content-length</strong> header) without actually loading the resource itself. In practice this could be used to determine if the full resource should be requested (or even how to request it).

### Example: POST requests

Fetch can also send data to an API with POST requests. The following code sends a "title" and "message" (as a string) to <strong>someurl/comment</strong>:

#### main.js
```
fetch('someurl/comment', {
  method: 'POST',
  body: 'title=hello&message=world'
})
```

<div class="note">
<strong>Note: </strong>In production, remember to always encrypt any sensitive user data.
</div>

The method is again specified with the <code>init</code> parameter. This is also where the body of the request is set, which represents the data to be sent (in this case the title and message).

The body data could also be extracted from a form using the <a href="https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData">FormData</a> interface. For example, the above code could be updated to:

#### main.js
```
// Assuming an HTML <form> with id of 'myForm'
fetch('someurl/comment', {
  method: 'POST',
  body: new FormData(document.getElementById('myForm')
})
```

### Custom headers

The <code>init</code> parameter can be used with the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Headers">Headers</a> interface to perform various actions on HTTP request and response headers, including retrieving, setting, adding, and removing them. An example of reading response headers was shown in a <a href="#head">previous section</a>. The following code demonstrates how a custom <a href="https://developer.mozilla.org/en-US/docs/Web/API/Headers">Headers</a> object can be created and used with a fetch request:

#### main.js
```
var myHeaders = new Headers({
  'Content-Type': 'text/plain',
  'X-Custom-Header': 'hello world'
});

fetch('/someurl', {
  headers: myHeaders
});
```

Here we are creating a Headers object where the <code>Content-Type</code> header has the value of <code>text/plain</code> and a custom <code>X-Custom-Header</code> header has the value of <code>hello world</code>. 

<div class="note">
<strong>Note: </strong>Only some headers, like <code>Content-Type</code> can be modified. Others, like <code>Content-Length</code> and <code>Origin</code> are <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#Guard">guarded</a>, and cannot be modified (for security reasons).
</div>

Custom headers on <a href="#cors">cross-origin</a> requests must be supported by the server from which the resource is requested. The server in this example would need to be configured to accept the <code>X-Custom-Header</code> header in order for the fetch to succeed. When a custom header is set, the browser performs a <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Preflighted_requests">preflight</a> check. This means that the browser first sends an <code>OPTIONS</code> request to the server to determine what HTTP methods and headers are allowed by the server. If the server is configured to accept the method and headers of the original request, then it is sent. Otherwise, an error is thrown. 

#### For more information

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Headers">Headers</a>
* <a href="https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request">Preflight checks</a>

<a id="cors" />


## Cross-origin requests




Fetch (and XMLHttpRequest) follow the <a href="https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy">same-origin policy</a>. This means that browsers restrict <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS">cross-origin</a> HTTP requests from within scripts.  A cross-origin request occurs when one domain (for example <strong>http://<span></span>foo.com/</strong>) requests a resource from a separate domain (for example <strong>http://<span></span>bar.com/</strong>). This code shows a simple example of a cross-origin request:

#### main.js
```
// From http://foo.com/
fetch('http://bar.com/data.json') 
.then(function(response) {
  // Do something with response
});
```

<div class="note">
<strong>Note:</strong> Cross-origin request restrictions are often a point of confusion. Many resources like images, stylesheets, and scripts are fetched cross-origin. However, these are exceptions to the same-origin policy. Cross-origin requests are still restricted <em>from within scripts</em>.
</div>

There have been attempts to work around the same-origin policy (such as <a href="http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about">JSONP</a>). The <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS">Cross Origin Resource Sharing</a> (CORS) mechanism has enabled a standardized means of retrieving cross-origin resources. The CORS mechanism lets you specify in a request that you want to retrieve a cross-origin resource (in fetch this is enabled by default). The browser adds an <code>Origin</code> header to the request, and then requests the appropriate resource. The browser only returns the response if the server returns an <code>Access-Control-Allow-Origin</code> header specifying that the origin has permission to request the resource. In practice, servers that expect a variety of parties to request their resources (such as 3rd party APIs) set a wildcard value for the <code>Access-Control-Allow-Origin</code> header, allowing anyone to access that resource.

If the server you are requesting from doesn't support CORS, you should get an error in the console indicating that the cross-origin request is blocked due to the CORS <code>Access-Control-Allow-Origin</code> header being missing. 

You can use <a href="https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch"><code>no-cors</code></a> mode to request opaque resources. <a href="https://fetch.spec.whatwg.org/#concept-filtered-response-opaque">Opaque responses</a> can't be accessed with JavaScript but the response can still be served or cached by a service worker. Using <code>no-cors</code> mode with fetch is relatively simple. To update the above example with <code>no-cors</code>, we pass in the <code>init</code> object with <code>mode</code> set to <code>no-cors</code>:

#### main.js
```
  // From http://foo.com/
fetch('http://bar.com/data.json', {
  mode: 'no-cors' // 'cors' by default
}) 
.then(function(response) {
  // Do something with response
});
```

#### For more information

* <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS">Cross Origin Resource Sharing</a>

<a id="furtherreading" />


## Further reading




* <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_fetch_api.html">Fetch API Codelab</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API">Learn more about the Fetch API</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch">Learn more about Using Fetch</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch">Learn more about GlobalFetch.fetch()</a>
* <a href="https://developers.google.com/web/updates/2015/03/introduction-to-fetch">Get an Introduction to Fetch</a>
* <a href="https://davidwalsh.name/fetch">David Welsh's blog on fetch</a>
* <a href="https://jakearchibald.com/2015/thats-so-fetch/">Jake Archibald's blog on fetch</a>


