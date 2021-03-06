# Introduction to Service Worker




## Contents




<a href="#whatisserviceworker"><strong>What is a service worker?</strong></a> 

<a href="#whatcantheydo"><strong>What can service workers do?</strong></a> 

<a href="#lifecycle"><strong>Service worker lifecycle</strong></a> 

<a href="#events"><strong>Service worker events</strong></a>

<a href="#resources"><strong>Further reading</strong></a>

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_scripting_the_service_worker.html">Scripting the Service Worker</a>

<a id="whatisserviceworker" />


## What is a service worker?




A <a href="https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API">service worker</a> is a type of <a href="https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API">web worker</a>. It's essentially a JavaScript file that runs separately from the main browser thread, intercepting network requests, caching or retrieving resources from the cache, and delivering push messages. 

Because workers run separately from the main thread, service workers are independent of the application they are associated with. This has several consequences:

* Because the service worker is not blocking (it's designed to be fully asynchronous) synchronous XHR and <code>localStorage</code> cannot be used in a service worker.
* The service worker can receive push messages from a server when the app is not active. This lets your app show push notifications to the user, even when it is not open in the browser.

 <div class="note">
<strong>Note:</strong> Whether notifications are received when the browser itself is not running depends on how the browser is integrated with the OS. For instance on desktop OS's, Chrome and Firefox only receive notifications when the browser is running. However, Android is designed to wake up any browser when a push message is received and will always receive push messages regardless of browser state. See the <a href="https://web-push-book.gauntface.com/chapter-07/01-faq/#why-doesnt-push-work-when-the-browser-is-closed">FAQ</a> in Matt Gaunt's <a href="https://web-push-book.gauntface.com/">Web Push Book</a> for more information.
</div>

* The service worker can't access the DOM directly. To communicate with the page, the service worker uses the <a href="https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage"><code>postMessage()</code></a> method to send data and a "message" event listener to receive data.

Things to note about a service worker:

* Service worker is a programmable network proxy that lets you control how network requests from your page are handled.
* Service workers only run over HTTPS. Because service workers can intercept network requests and modify responses, "man-in-the-middle" attacks could be very bad.

<div class="note">
<strong>Note: </strong>Services like <a href="https://letsencrypt.org/">Letsencrypt</a> let you procure SSL certificates for free to install on your server. 
</div>

* The service worker becomes idle when not in use and restarts when it's next needed. You cannot rely on a global state persisting between events. If there is information that you need to persist and reuse across restarts, you can use <a href="https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API">IndexedDB</a> databases.
* Service workers make extensive use of promises, so if you're new to promises, then you should stop reading this and check out <a href="https://developers.google.com/web/fundamentals/getting-started/primers/promises">Promises, an introduction</a>.

<a id="whatcantheydo" />


## What can service workers do?




Service workers enable applications to control network requests, cache those requests to improve performance, and provide offline access to cached content.  

Service workers depend on two APIs to make an app work offline: <a href="https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API">Fetch</a> (a standard way to retrieve content from the network) and <a href="https://developer.mozilla.org/en-US/docs/Web/API/Cache">Cache</a> (a persistent content storage for application data). This cache is persistent and independent from the browser cache or network status.

### Improve performance of your application/site

Caching resources will make content load faster under most network conditions. See <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_caching_files_with_service_worker.html">Caching files with the service worker</a> and <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/">The Offline Cookbook</a> for a full list of caching strategies.

### Make your app "offline-first"

Using the Fetch API inside a service worker, we can intercept network requests and then modify the response with content other than the requested resource. We can use this technique to serve resources from the cache when the user is offline. See <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_caching_files_with_service_worker.html">Caching files with the service worker</a> to get hands-on experience with this technique.

### Act as the base for advanced features

Service workers provide the starting point for features that make web applications work like native apps. Some of these features are: 

* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API">Notifications API</a>: A way to display and interact with notifications using the operating system's native notification system.
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Push_API">Push API: </a> An API that enables your app to subscribe to a push service and receive push messages. Push messages are delivered to a service worker, which can use the information in the message to update the local state or display a notification to the user. Because service workers run independently of the main app, they can receive and display notifications even when the browser is not running. 
* <a href="https://developers.google.com/web/updates/2015/12/background-sync">Background Sync API</a>: Lets you defer actions until the user has stable connectivity. This is useful to ensure that whatever the user wants to send is actually sent. This API also allows servers to push periodic updates to the app so the app can update when it's next online
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Channel_Messaging_API">Channel Messaging API</a>: Lets web workers and service workers communicate with each other and with the host application. Examples of this API include new content notification and updates that require user interaction.

<a id="lifecycle" />


## Service worker lifecycle




A service worker goes through three steps in its lifecycle:

* Registration
* Installation
* Activation

### Registration and scope

To <strong>install</strong> a service worker, you need to <strong>register</strong> it in your main JavaScript code. Registration tells the browser where your service worker is located, and to start installing it in the background. Let's look at an example:

#### main.js
```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js')
  .then(function(registration) {
    console.log('Registration successful, scope is:', registration.scope);
  })
  .catch(function(error) {
    console.log('Service worker registration failed, error:', error);
  });
}
```

This code starts by checking for browser support by examining <code>navigator.serviceWorker</code>. The service worker is then registered with <code>navigator.serviceWorker.register</code>, which returns a promise that resolves when the service worker has been successfully registered. The <code>scope</code> of the service worker is then logged with <code>registration.scope</code>. 

The <code>scope</code> of the service worker determines which files the service worker controls, in other words, from which path the service worker will intercept requests. The default scope is the location of the service worker file, and extends to all directories below. So if <strong>service-worker.js</strong> is located in the root directory, the service worker will control requests from all files at this domain.

You can also set an arbitrary scope by passing in an additional parameter when registering. For example:

#### main.js
```
navigator.serviceWorker.register('/service-worker.js', {
  scope: '/app/'
});
```

In this case we are setting the scope of the service worker to <code>/app/</code>, which means the service worker will control requests from pages like <code>/app/</code>, <code>/app/lower/</code> and <code>/app/lower/lower</code>, but not from pages like <code>/app</code> or <code>/</code>, which are higher. 

If the service worker is already installed, <code>navigator.serviceWorker.register</code> returns the registration object of the currently active service worker.

### Installation

Once the the browser registers a service worker, <strong>installation</strong> can be attempted. This occurs if the service worker is considered to be new by the browser, either because the site currently doesn't have a registered service worker, or because there is a byte difference between the new service worker and the previously installed one. 

A service worker installation triggers an <code>install</code> event in the installing service worker. We can include an <code>install</code> event listener in the service worker to perform some task when the service worker installs. For instance, during the install, service workers can precache parts of a web app so that it loads instantly the next time a user opens it (see <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#on-install-as-dependency">caching the application shell</a>). So, after that first load, you're going to benefit from instant repeat loads and your time to interactivity is going to be even better in those cases. An example of an installation event listener looks like this: 

#### service-worker.js
```
// Listen for install event, set callback
self.addEventListener('install', function(event) {
    // Perform some task
});
```

<a id="activation" />

### Activation

Once a service worker has successfully installed, it transitions into the <strong>activation</strong> stage. If there are any open pages controlled by the previous service worker, the new service worker enters a <code>waiting</code> state. The new service worker only activates when there are no longer any pages loaded that are still using the old service worker. This ensures that only one version of the service worker is running at any given time. 

<div class="note">
<strong>Note: </strong>Simply refreshing the page is not sufficient to transfer control to a new service worker, because the new page will be requested before the the current page is unloaded, and there won't be a time when the old service worker is not in use.
</div>

When the new service worker activates, an <code>activate</code> event is triggered in the activating service worker. This event listener is a good place to clean up outdated caches (see the <a href="https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#on-activate">Offline Cookbook</a> for an example).

#### service-worker.js
```
self.addEventListener('activate', function(event) {
  // Perform some task
});
```

Once activated, the service worker controls all pages that load within its scope, and starts listening for events from those pages. However, pages in your app that were loaded before the service worker activation will not be under service worker control. The new service worker will only take over when you close and reopen your app, or if the service worker calls <a href="https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim"><code>clients.claim()</code></a>. Until then, requests from this page will not be intercepted by the new service worker. This is intentional as a way to ensure consistency in your site.

<a id="events" />


## Service worker events




Service workers are event driven. Both the installation and activation processes trigger corresponding <code>install</code> and <code>activate</code> events to which the service workers can respond. There are also <code>message</code> events, where the service worker can receive information from other scripts, and functional events such as <code>fetch</code>, <code>push</code>, and <code>sync</code>. 

To examine service workers, navigate to the Service Worker section in your browser's developer tools. The process is different in each browser that supports service workers. For information about using your browser's developer tools to check the status of service workers, see <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/tools_for_pwa_developers.html">Tools for PWA Developers</a>.

<a id="resources" />


## Further reading




* A more detailed introduction to <a href="https://developers.google.com/web/fundamentals/instant-and-offline/service-worker/lifecycle">The Service Worker Lifecycle</a>
* More on <a href="https://developers.google.com/web/fundamentals/instant-and-offline/service-worker/registration">Service Worker Registration</a>
* <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_scripting_the_service_worker.html">Create your own service worker</a> (lab)
* <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_offline_quickstart.html">Take a blog site offline</a> (lab)
* <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_caching_files_with_service_worker.html">Cache files with Service Worker</a> (lab)


