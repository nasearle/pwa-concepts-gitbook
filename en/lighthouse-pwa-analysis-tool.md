# Lighthouse PWA Analysis Tool




## Contents




<a href="#introduction"><strong>Introduction</strong></a> 

<a href="#extension"><strong>Running Lighthouse as a Chrome extension</strong></a>

<a href="#commandline"><strong>Running Lighthouse from the command line</strong></a>

Codelab: <a href="https://google-developer-training.gitbooks.io/progressive-web-apps-ilt-codelabs/content/docs/lab_auditing_with_lighthouse.html">Auditing with Lighthouse</a>

<a id="introduction" />


## Introduction




How do I tell if all of my Progressive Web App (PWA) features are in order? <a href="https://github.com/GoogleChrome/lighthouse">Lighthouse</a> is an open-source tool from Google that audits a web app for PWA features. It provides a set of metrics to help guide you in building a PWA with a full application-like experience for your users. 

Lighthouse tests if your app:

* Can load in offline or flaky network conditions
* Is relatively fast
* Is served from a secure origin
* Uses certain accessibility best practices

Lighthouse is available as a Chrome extension for Chrome 52 (and later) and a command line tool. 

<a id="extension" />


## Running Lighthouse as a Chrome extension




Download the Lighthouse Chrome extension from the <a href="http://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk">Chrome Web Store</a>. 

When installed it places an <img src="../img/91e97511ef44e440.png" style="width:20px;height:20px;" alt="Lighthouse Icon ">  icon in your taskbar. 

Run Lighthouse on your application by selecting the icon and choosing <strong>Generate report</strong> (with your app open in the browser page).

![Lighthouse extension showing generate report button](../img/92c3177801055abb.png)

Lighthouse generates an HTML page with the results. An example page is shown below. 

![Lighthouse report](../img/76f48671607bf2b2.png)

<div class="note">
<strong>Note: </strong>You can test it out on an example PWA, <a href="https://www.airhorner.com/">airhorner.com</a>.
</div>

<a id="commandline" />


## Running Lighthouse from the command line




If you want to run Lighthouse from the command line (for example, to integrate it with a build process) it is available as a <a href="https://nodejs.org/en/">Node</a> module. 

You can download Node from <a href="https://nodejs.org/en/">nodejs.org</a> (select the version that best suits your environment and operating system). 

<div class="note">
<strong>Note:</strong> You need the --harmony <a href="http://stackoverflow.com/questions/13351965/what-does-node-harmony-do">flag</a> with Node v5+ or Node v4.
</div>

To install Lighthouse's Node module from the command line, use the following command:

    npm install -g lighthouse

This installs the tool globally. You can then run Lighthouse from the command line (where <a href="https://airhorner.com/">https://airhorner.com/</a> is your app):

    lighthouse https://airhorner.com/


You can check Lighthouse flags and options with the following command:

    lighthouse --help


