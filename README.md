# JavaScript SDK Design Guide 
[![All Contributors](https://img.shields.io/badge/all_contributors-3-orange.svg?style=flat-square)](#contributors-)

## Introduction

This guide provides an introduction to develop a [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) SDK.

> The best one sentence to describe an SDK is: _"The SDK is the connection bridging the gap between users and the (browser) machine."_


By using this guide, the SDK will be able to run in browsers, desktop, mobile web and various other platforms capable of running JavaScript. 

The target audience of this writeup excludes **non-browser environments** such as hardware, embedded and Node.js.

> It is possible to suggest improvements by [editing](https://github.com/hueitan/JavaScript-sdk-design/edit/master/README.md), or dropping suggestions on the [issue list](https://github.com/hueitan/JavaScript-sdk-design/issues). I owe you a beer :beers:

<p align="right">
<i>READ IT</i>
<a href="http://sdk-design.js.org" target="_blank">ONLINE</a> /
<a href="https://gitprint.com/huei90/javascript-sdk-design/blob/master/README.md" target="_blank">PDF</a>
<i>OR</i>
<a href="http://ctt.ec/GyjRN" title="Click to Tweet">Tweet It</a> <br/>
</p>

<!--<hr/>-->
<!--<p align="center">-->
<!--<a href="http://ctt.ec/GyjRN" title="Click to Tweet">Click to Tweet</a>-->
<!--</p>-->
<hr/>

## Content

* [What is an SDK](#what-is-an-sdk)
* [Design Philosophy](#design-philosophy)
* [Scope](#scope)
* [Include the SDK](#include-the-sdk)
  * [Asynchronous Syntax](#asynchronous-syntax)
  * [Traditional Syntax](#traditional-syntax)
  * [Comparison](#comparison)
  * [Problem of Asynchronous](#problem-of-asynchronous)
  * [Others](#others)
* [SDK Versioning](#sdk-versioning)
* [Changelog Document](#changelog-document)
* [Namespace](#namespace)
* [Storage Mechanism](#storage-mechanism)
  * [Cookie](#cookie)
    * [Check Cookie Writable](#check-cookie-writable)
    * [Check Third-Party Cookie Writable](#check-third-party-cookie-writable)
    * [Write Read Remove Cookie Code](#writereadremove-cookie-code)
  * [Session](#session)
  * [LocalStorage](#localstorage)
    * [Check LocalStorage Writable](#check-localstorage-writable)
  * [Session Storage](#session-storage)
    * [Check SessionStorage Writable](#check-sessionstorage-writable)
* [Event](#event)
  * [Document Ready](#document-ready)
  * [Message Event](#message-event)
  * [Orientation Change](#orientation-change)
  * [Disable Scroll](#disable-scroll)
* [Request](#request)
  * [Image Beacon](#image-beacon)
  * [Single Post](#single-post)
  * [Multiple Post](#multiple-post)
  * [Iframe](#iframe)
  * [Script jsonp](#script-jsonp)
  * [Navigator.sendBeacon()](#navigatorsendbeacon)
  * [XMLHttpRequest](#xmlhttprequest)
  * [Fragment Identifier](#fragment-identifier)
  * [Maximum Number of Connection](#maximum-number-of-connection)
* [Component of URI](#component-of-uri)
  * [Parsing URI](#parsing-uri)
* [Debugging](#debugging)
  * [Simulating Multiple Domains](#simulating-multiple-domains)
  * [Developer Tools](#developer-tools)
  * [Console Logs](#console-logs)
  * [Debugging Proxy](#debugging-proxy)
  * [BrowserSync](#browsersync)
  * [Debugging Node.js Apps](#debugging-nodejs-apps)
* [Tips and Tricks](#tips-and-tricks)
  * [Piggyback](#piggyback)
  * [Page Visibility API](#page-visibility-api)
  * [Document Referrer](#document-referrer)
  * [Console Logs Polyfill](#console-logs-polyfill)
  * [EncodeURI or EncodeURIComponent](#encodeuri-or-encodeuricomponent)
  * [YOU MIGHT NOT NEED JQUERY](#you-might-not-need-jquery)
  * [You Don't Need jQuery](#you-dont-need-jquery)
  * [Load Script with Callback](#load-script-with-callback)
  * [Once Function](#once-function)
  * [Pixel Ratio Density](#pixel-ratio-density)
  * [Get Style Value](#get-style-value)
  * [Check if Element in Viewport](#check-if-element-in-viewport)
  * [Check if Element is Visible](#check-if-element-is-visible)
  * [Get Viewport Size](#get-viewport-size)
  * [User Tracking](#user-tracking)
  * [Opt Out](#opt-out)
* [WTF](#wtf)
  * [Misspelling Of Referrer](#misspelling-of-referrer)
* [Template](#template)
* [Book to Read](#booknice-to-read)
* [Contributors](#contributors-)

## What is an SDK

This question is pretty ubiquitous, but here it is again.

"Short for **software development kit**, a programming package that enables a programmer to develop applications for a specific platform. Typically an SDK includes one or more APIs, programming tools, and documentation." - _[webopedia](http://www.webopedia.com/TERM/S/SDK.html)_

## Design Philosophy

Depending on the purpose of SDK's service and usage — common shared traits are, but not limited to be **native**, **short**, **fast**, **clean**, **readable** and **testable**.

The widely adopted good practice, is to write SDK with vanilla JavaScript. 
Languages compiling to JavaScript such as LiveScript, CoffeeScript, TypeScript and others are **not** recommended.

It is also recommended **not** to use libraries suchs as jQuery in SDK development. 
The exception is of course when it is really important. There are also other jQuery-like libraries, zepto.js _etc_ to choose from, for the DOM manipulation purposes.

In event of HTTP [ajax request](#request) requirements — there are native equivalent such as `window.fetch`. It is light-weight, supported in ever growing platforms. 

Backward compatibility is paramount. Every new  SDK version released should be enabled with  support of previous older versions. Likewise, current version should be designed to support future SDK versions. This is referred to as Forward compatibility.

Moreover, a good **Documentation**, well commented code, a healthy unit test coverage, as well as end-to-end (user) scenario are key to the success of SDK.

## Scope

*Based on the book [Third-Party JavaScript](http://thirdpartyjs.com)*

Three use cases worth considering while designing a JavaScript SDK:

1. [Embedded widgets](./SCOPE.md#embedded-widgets) - Small interactive applications embedded on the publisher's web page (Disqus, Google Maps, Facebook Widget)
2. [Analytics and metrics](./SCOPE.md#analytics-and-metrics) - For gathering intelligence about visitors and how they interact with the publisher's website (GA, Flurry, Mixpanel)
3. [Web service API wrappers](./SCOPE.md#web-service-api-wrappers) - For developing client-side applications that communicate with external web services. (Facebook Graph API)

> [Suggest a case](https://github.com/hueitan/JavaScript-sdk-design/edit/master/README.md) in which the use of an SDK in JavaScript environment is deemed important.

## Include the SDK

To include the SDK in a user-facing environment, It is a good practice to use **Asynchronous Syntax** to load the scripts.

This helps to optimize the user experience on the website that are using the SDK.
This approach reduces chances of the SDK library interfering with the hosting website.

### Asynchronous Syntax

```html
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://<DOMAIN>.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();
</script>
```

The `async` syntax is used when targetting modern browsers.

```html
<script async src="http://<DOMAIN>.com/sdk.js"></script>
```

### Traditional Syntax

```html
<script type="text/javascript" src="http://<DOMAIN>.com/sdk.js"></script>
```

### Comparison

Here's the simple graph to show the difference between Asynchronous and Traditional Syntax.

Asynchronous:

``` 
 |----A-----|
    |-----B-----------|
        |-------C------|
```

Synchronous:

```
  |----A-----||-----B-----------||-------C------|
```

Asynchronous and deferred JavaScript execution explained

<img src="http://peter.sh/wp-content/uploads/2010/09/execution2.jpg"/>



> _https://developers.google.com/speed/docs/insights/BlockingJS_ <br/>
> It is good practice to avoid, or minimize, the use of blocking JavaScript, especially external scripts that must be fetched before they can be executed. Scripts that are necessary to render page content can be inlined to avoid extra network requests, however the inlined content needs to be small and must execute quickly (non-blocking fashion) to deliver good performance. Scripts that are not critical to initial render should be made asynchronous or deferred until after the first render.

### Problem of Asynchronous

When using an Asynchronous approach, It is ill-advised to execute SDK initialization functions before all libraries are loaded, parsed and executed in the hosting page.

Consider the following snippet as a visual example to the previous statement:

```javascript
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://<DOMAIN>.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```

The end result of such initialization will lead to bugs. 
The `SDKName()` function, undefined at this point, executes before it becomes available in the environment's global variable. The script is not loaded yet.

To make it work, some tricks are necessary to make sure the script executes successfully. The event will (need to) be stored in the `SDKName.q` queue array. The SDK should be able to handle and execute the `SDKName.q` event and initialize the `SDKName` namespace.

The following snippet depicts the statement in previous paragraph.

```javascript
<script>
  (function () {
    // add a queue event here
    SDKName = SDKName || function () {
      (SDKName.q = SDKName.q || []).push(arguments);
    };
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://<DOMAIN>.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```

**Or** using `[].push`

```js
<script>
  (function () {
    // add a queue event here
    SDKName = window.SDKName || (window.SDKName = []);
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://<DOMAIN>.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName.push(['some arguments']);
</script>
```

### Others

There are other different ways to include a script

#### Import in ES2015

```javascript
  import "your-sdk";
```

#### Modular include a Script

There is full source code — and this awesome tutorial _"[Loading JavaScript Modules](https://libraryinstitute.wordpress.com/2010/12/01/loading-javascript-modules/)"_ may help for in-depth understanding of concepts discussed above.

```js
module('sdk.js',['sdk-track.js', 'sdk-beacon.js'],function(track, beacon) {
  // sdk definitions, split into local and global/exported definitions
  // local definitions
  // exports
});

// you should contain this "module" method
(function () {

  var modules = {}; // private record of module data

  // modules are functions with additional information
  function module(name,imports,mod) {

    // record module information
    window.console.log('found module '+name);
    modules[name] = {name:name, imports: imports, mod: mod};

    // trigger loading of import dependencies
    for (var imp in imports) loadModule(imports[imp]);

    // check whether this was the last module to be loaded
    // in a given dependency group
    loadedModule(name);
  }

  // function loadModule
  // function loadedModule

  window.module = module;
})();
```

## SDK Versioning

It is not a good practice to use one of the following versioning styles: 

  - `brand-v<timestamp>.js` 
  - `brand-v<datetime>.js` 
  - `brand-v1-v2.js`,

The reason is that it becomes confusing to track the lastest version. 
Therefore, previous styling does not help developers who use the SDK.

It is however a good practice to use [Semantic Versioning](http://semver.org), also known as **SemVer**, when versioning SDKs.
It has three main parts, each corresponding to importance of a release: "MAJOR.MINOR.PATCH".
Version in `v1.0.0` `v1.5.0` `v2.0.0` is easier to trace and track in changelog documentation, for instance.

Depending on service design, some of the ways SDK can be distributed (or tracked) by version are the following:

 - Using Query String path — `http://<DOMAIN>.com/sdk.js?v=1.0.0`
 - Using the Folder Naming — `http://<DOMAIN>.com/v1.0.0/sdk.js`
 - Using hostname (subdomain) — `http://v1.<DOMAIN>.com/sdk.js`

Depending on Use Case, there are other environment dependent forms that are commonly advised to use:  

 - In `stable` version `http://<DOMAIN>.com/sdk-stable.js`  
 - In `unstable` version `http://<DOMAIN>.com/sdk-unstable.js` 
 - In `alpha` version `http://<DOMAIN>.com/sdk-alpha.js` 
 - In `latest` version `http://<DOMAIN>.com/sdk-latest.js` 
 - In `experimental` version `http://<DOMAIN>.com/sdk-experimental.js`

> Reading suggestion: _*[Why use SemVer?](http://blog.npmjs.org/post/162134793605/why-use-semver)*_ on `npm` blog.

## Changelog Document

It's hard to notice when an SDK has updates (or is upgraded) when no announcement has been issued.
It's good practice to write a Changelog to document major, minor and even bug-fix changes.
Tracking changes in SDK APIs deliver good developer experience. - _[Keep a Changelog](http://keepachangelog.com) ([Github Repo](https://github.com/olivierlacan/keep-a-changelog))_

Each version should have:

```
[Added] for new features.
[Changed] for changes in existing functionality.
[Deprecated] for soon-to-be removed features.
[Removed] for now removed features.
[Fixed] for any bug fixes.
[Security] in case of vulnerabilities.
```

In addition, [commit-message-emoji](https://github.com/dannyfritz/commit-message-emoji) uses an emoji to explain the commit's changes itself. Find the best suitable format or [changelog generator tool](https://github.com/topics/changelog) for your project.

## Namespace

To avoid collision with other libraries, it is better to define no more than one global SDK namespace.
The naming should also avoid using the commonly used words and catch-phrases as namespaces.

As a quick example, SDK playground can well use `(function () { ... })()` or ES6 Blocks `{ ... }` to wrap all sources.

This is an increasingly common practice found in many popular JavaScript libraries such as (jQuery, Node.js, etc.). This technique creates a closure around the entire contents of the file which, perhaps most importantly, creates a private namespace and thereby helps avoid potential name clashes between different JavaScript modules and libraries. [#](http://www.toptal.com/javascript/interview-questions)

To avoid **namespace collision**

From Google Analytics, define the namespace by changing the value `ga`

```js
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
```

From [OpenX experience](http://docs.openx.com/ad_server/adtags_namespace.html), support a parameter to request the namespace.

```html
<script src="http://your_domain/sdk?namespace=yourcompany"></script>
```

## Storage Mechanism

### Cookie

The domain scope of using cookies is quite complex while involving the `subdomain` and `path`.

For `path=/`,
there is a cookie part `first=value1` in domain `http://github.com`,
and another cookie `second=value2` in domain `http://sub.github.com`

|               | http://github.com | http://sub.github.com |
|:-------------:|:-----------------:|:---------------------:|
| first=value1  |         ✓         |           ✓           |
| second=value2 |         ✘         |           ✓           |

There is a cookie `first=value1` in domain `http://github.com`,
cookie `second=value2` in domain path `http://github.com/path1`
and cookie `third=value3` in domain `http://sub.github.com`,


|               | http://github.com | http://github.com/path1 | http://sub.github.com |
|:-------------:|:-----------------:|:-----------------------:|:---------------------:|
| first=value1  |         ✓         |            ✓            |           ✓           |
| second=value2 |         ✘         |            ✓            |           ✘           |
| third=value3  |         ✘         |            ✘            |           ✓           |

#### Check Cookie Writable

Given a domain (Default as current hostname), check whether the cookie is writable.

```js
var checkCookieWritable = function(domain) {
    try {
        // Create cookie
        document.cookie = 'cookietest=1' + (domain ? '; domain=' + domain : '');
        var ret = document.cookie.indexOf('cookietest=') != -1;
        // Delete cookie
        document.cookie = 'cookietest=1; expires=Thu, 01-Jan-1970 00:00:01 GMT' + (domain ? '; domain=' + domain : '');
        return ret;
    } catch (e) {
        return false;
    }
};
```

#### Check Third-Party Cookie Writable

It's impossible to check only using client-side JavaScript, but a server can help to achieve just that. 

#### Write/Read/Remove Cookie Code

Code snippet for write/read/remove cookie script.

```js
var cookie = {
    write: function(name, value, days, domain, path) {
        var date = new Date();
        days = days || 730; // two years
        path = path || '/';
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        var expires = '; expires=' + date.toGMTString();
        var cookieValue = name + '=' + value + expires + '; path=' + path;
        if (domain) {
            cookieValue += '; domain=' + domain;
        }
        document.cookie = cookieValue;
    },
    read: function(name) {
        var allCookie = '' + document.cookie;
        var index = allCookie.indexOf(name);
        if (name === undefined || name === '' || index === -1) return '';
        var ind1 = allCookie.indexOf(';', index);
        if (ind1 == -1) ind1 = allCookie.length;
        return unescape(allCookie.substring(index + name.length + 1, ind1));
    },
    remove: function(name) {
        if (this.read(name)) {
            this.write(name, '', -1, '/');
        }
    }
};
```
### Session

It's important to know that in JavaScript it is **not possible** to write a Session.
That is the server responsibility. The server-side team should implement Session management related Use Cases.

A page session lasts for as long as the browser is open and survives over page reloads and restores. Opening a page in a new tab or window will cause a new session to be initiated.

### LocalStorage

Stores data with no expiration date, storage limit is far larger (at least 5MB) and
information is never transferred to the server.

It's good to know that each localStorage from `http` and `https` in the same domain aren't shared.
Creating an iframe inside a website and using `postMessage` to pass the value to others. 

- [HOW TO?](http://stackoverflow.com/questions/10502469/is-there-any-workaround-to-make-use-of-html5-localstorage-on-both-http-and-https)

#### Check LocalStorage Writable

window.localStorage is not supported by all browsers, the SDK should check if it's available before using it.

```js
var testCanLocalStorage = function() {
   var mod = 'modernizr';
   try {
       localStorage.setItem(mod, mod);
       localStorage.removeItem(mod);
       return true;
   } catch (e) {
       return false;
   }
};
```

### Session Storage

Stores data for one session (data is lost when the tab is closed).

### Check SessionStorage Writable

```js
var checkCanSessionStorage = function() {
  var mod = 'modernizr';
  try {
    sessionStorage.setItem(mod, mod);
    sessionStorage.removeItem(mod);
    return true;
  } catch (e) {
    return false;
  }
}
```

## Event

In client browser, there are events `load` `unload` `on` `off` `bind` .... Here's some [polyfills](https://remysharp.com/2010/10/08/what-is-a-polyfill) for you to handle all different platforms.

### Document Ready

Please do make sure that the entire page is finished loading (ready) before starting execution of the SDK functions.

```js
// handle IE8+
function ready (fn) {
    if (document.readyState != 'loading') {
        fn();
    } else if (window.addEventListener) {
        // window.addEventListener('load', fn);
        window.addEventListener('DOMContentLoaded', fn);
    } else {
        window.attachEvent('onreadystatechange', function() {
            if (document.readyState != 'loading')
                fn();
            });
    }
}
```

> **DOMContentLoaded** -  fired when the document has been completely loaded and parsed, without waiting for stylesheets, images, and subframes to finish loading

> **load** event can be used to detect a fully-loaded page

> Information from JS Tip - https://github.com/loverajoel/jstips/blob/master/_posts/en/javascript/2016-02-15-detect-document-ready-in-pure-js.md

> [element-ready](https://github.com/sindresorhus/element-ready) from [sindresorhus](https://github.com/sindresorhus)

### Message Event

It's about the cross-origin communication between iframe and window, read the [API documentation](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage).

```js
// in the iframe
parent.postMessage("Hello"); // string

// ==========================================

// in the iframe's parent
// Create IE + others compatible event handler
var eventMethod = window.addEventListener ? "addEventListener" : "attachEvent";
var eventer = window[eventMethod];
var messageEvent = eventMethod == "attachEvent" ? "onmessage" : "message";

// Listen to message from child window
eventer(messageEvent,function(e) {
  // e.origin , check the message origin
  console.log('parent received message!:  ',e.data);
},false);
```

The Post message data should be **String**, for more advanced use in JSON, use **JSON String**. Although the modern browsers do support [Structured Clone Algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) on the parameter, not all browsers do.

### Orientation Change

Detect device orientation change

```js
window.addEventListener('orientationchange', fn);
```

Get Orientation Rotate Degree

```js
window.orientation; // => 90, -90, 0
```

Screen portrait-primary, portrait-secondary, landscape-primary, landscape-secondary (Experimental)

```js
// https://developer.mozilla.org/en-US/docs/Web/API/Screen/orientation
var orientation = screen.orientation || screen.mozOrientation || screen.msOrientation;
```

### Disable Scroll

In web page, use CSS style `overflow: hidden`, in some mobile webs, this CSS doesn't work, use JavaScript event.

```js
document.addEventListener('touchstart', function(e){ e.preventDefault(); });
// or
document.body.addEventListener('touchstart', function(e){ e.preventDefault(); });
// use move if you need some touch event
document.addEventListener('touchmove', function(e){ e.preventDefault(); });
// target modern browser
document.addEventListener('touchmove', function(e){ e.preventDefault(); }, { passive: false });
```

## Request

The communication between our SDK and Server is using Ajax Request. 
Most common use cases leverage jQuery's ajax http request to communicate with the Server. The good news is that there is an even better solution to achieve that.

### Image Beacon

Using the Image Beacon to ask the browser to perform a GET method `request` to get an Image.

*Ones should always remember to add timestamp (Cache Buster) to prevent caching in browser.*

```js
(new Image()).src = 'http://<DOMAIN>.com/collect?id=1111';
```

Some notice for GET Query String, there is the limit of length which is 2048 (Basically it depends on different browsers and server). The following trick helps to handle the case of exceeded length limit.

```js
if (length > 2048) {
    // do Multiple Post (form)
} else {
    // do Image Beacon
}
```

There are well-known problems using `encodeURI` or `encodeURIComponent`. However, it is better to understand how these two approaches work. [Reading details below](#encodeuri-or-encodeuricomponent).

For the image load **success/error callback**

```js
var img = new Image();
img.src = 'http://<DOMAIN>.com/collect?id=1111';
img.onload = successCallback;
img.onerror = errorCallback;
```

### Single Post

it is possible to use the native form element POST method to send a key value.

```js
var form = document.createElement('form');
var input = document.createElement('input');

form.style.display = 'none';
form.setAttribute('method', 'POST');
form.setAttribute('action', 'http://<DOMAIN>.com/track');

input.name = 'username';
input.value = 'attacker';

form.appendChild(input);
document.getElementsByTagName('body')[0].appendChild(form);

form.submit();
```

### Multiple Posts

The Service is often complex, especially when needing to send more data through a POST method.

```js
function requestWithoutAjax( url, params, method ){

    params = params || {};
    method = method || "post";

    // function to remove the iframe
    var removeIframe = function( iframe ){
        iframe.parentElement.removeChild(iframe);
    };

    // make a iframe...
    var iframe = document.createElement('iframe');
    iframe.style.display = 'none';

    iframe.onload = function(){
        var iframeDoc = this.contentWindow.document;

        // Make a invisible form
        var form = iframeDoc.createElement('form');
        form.method = method;
        form.action = url;
        iframeDoc.body.appendChild(form);

        // pass the parameters
        for( var name in params ){
            var input = iframeDoc.createElement('input');
            input.type = 'hidden';
            input.name = name;
            input.value = params[name];
            form.appendChild(input);
        }

        form.submit();
        // remove the iframe
        setTimeout( function(){
            removeIframe(iframe);
        }, 500);
    };

    document.body.appendChild(iframe);
}
```
```js
requestWithoutAjax('url/to', { id: 2, price: 2.5, lastname: 'Gamez'});
```

### Iframe

Iframe embedded in html can always be used to cover the use case of generating content within the page.

```js
var iframe = document.createElement('iframe');
var body = document.getElementsByTagName('body')[0];

iframe.style.display = 'none';
iframe.src = 'http://<DOMAIN>.com/page';
iframe.onreadystatechange = function () {
    if (iframe.readyState !== 'complete') {
        return;
    }
};
iframe.onload = loadCallback;

body.appendChild(iframe);
```

**Remove extra margin from INSIDE an iframe**

```html
<iframe src="..."
 marginwidth="0"
 marginheight="0"
 hspace="0"
 vspace="0"
 frameborder="0"
 scrolling="no"></iframe>
```

**Putting html content into an iframe**

```html
<iframe id="iframe"></iframe>

<script>
  var html_string= "content <script>alert(location.href);</script>";
  document.getElementById('iframe').src = "data:text/html;charset=utf-8," + escape(html_string);
  // alert data:text/html;charset=utf-8.....
  // access cookie get ERROR

  var doc = document.getElementById('iframe').contentWindow.document;
  doc.open();
  doc.write('<body>Test<script>alert(location.href);</script></body>');
  doc.close();
  // alert "top window url"

  var iframe = document.createElement('iframe');
  iframe.src = 'javascript:;\'' + encodeURI('<html><body><script>alert(location.href);</body></html>') + '\'';
  // iframe.src = 'javascript:;"' + encodeURI((html_tag).replace(/\"/g, '\\\"')) + '"';
  document.body.appendChild(iframe);
  // alert "about:blank"
</script>
```

### Script jsonp

This is the case where your server needs to send a JavaScript `response` and let the client browser execute it.
Just include the JS script link.

```js
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = '/yourscript?some=parameter&callback=jsonpCallback';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();
```

To learn more about jsonp

1. JSONP only works in GET HTTP requests.
2. JSONP lacks error handling, means you cannot detect cases in response status code 404, 500 and so on.
3. JSONP requests are always asynchronous.
4. Beware of CSRF attack.
5. Cross domain communication. Script response side (server-side) don't need to care about CORS.

### Navigator.sendBeacon()

Look at the [documentation](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon).

> This method addresses the needs of analytics and diagnostics code that typically attempt to send data to a web server prior to the unloading of the document. Sending the data any sooner may result in a missed opportunity to gather data. However, ensuring that the data has been sent during the unloading of a document is something that has traditionally been difficult for developers.

Send POST beacon through the API. It's cool.

```js
navigator.sendBeacon("/log", analyticsData);
```

### XMLHttpRequest

Writing XMLHttpRequest is not a good idea. I assume that you don't want to waste time on battling with IE or other browsers. Here are some polyfills or code you can try:

1. [window.fetch](https://github.com/github/fetch) - A window.fetch JavaScript polyfill. (check also [ky](https://github.com/sindresorhus/ky))
2. [got](https://github.com/sindresorhus/got) - Simplified HTTP/HTTPS requests
3. [microjs](http://microjs.com/#ajax) - list of ajax lib
4. more

### Fragment Identifier

Also known as hash mark `#`. Remember that requests with hash mark at the end are not passed within http requests.

For example, you are in the page `http://github.com/awesome#hueitan`

```js
// Sending a request with a parameter url which contains current url
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome#hueitan';

// actual request will be without #
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome';

// Solution, encodeURIComponent(url):
(new Image()).src = 'http://yourrequest.com?url=' + encodeURIComponent('http://github.com/awesome#hueitan');
```

### Maximum Number of Connections

Check the maximum number of the browser's request connections. [browserscope](http://www.browserscope.org/?category=network&v=top)

<h2 align="center">
 <img src="https://cloud.githubusercontent.com/assets/2560096/9082891/ac4dc26e-3b99-11e5-8178-606270a801c4.png" alt="max number of connection"/>
</h2>

## Component of URI

It's important to know if the SDK needs to parse the location url.
```
                         authority
                   __________|_________
                  /                    \
              userinfo                host                          resource
               __|___                ___|___                 __________|___________
              /      \              /       \               /                      \
         username  password     hostname    port     path & segment      query   fragment
           __|___   __|__    ______|______   |   __________|_________   ____|____   |
          /      \ /     \  /             \ / \ /                    \ /         \ / \
    foo://username:password@www.example.com:123/hello/world/there.html?name=ferret#foo
    \_/                     \ / \       \ /    \__________/ \     \__/
     |                       |   \       |           |       \      |
  scheme               subdomain  \     tld      directory    \   suffix
                                   \____/                      \___/
                                      |                          |
                                    domain                   filename
```

### Parsing URI

Here's a simple way using the native URL() Interface but it's not supported by all browsers.
It is also [not a standard yet](https://developer.mozilla.org/en-US/docs/Web/API/Window/URL).

```js
var parser = new URL('http://github.com/hueitan');
parser.hostname; // => "github.com"
```

The DOM 's `createElement('a')` can be used in browsers that don't have the `URL()` Interface yet.

```js
var parser = document.createElement('a');
parser.href = "http://github.com/hueitan";
parser.hostname; // => "github.com"
```

## Debugging

### Simulating Multiple Domains

To simulate multiple domains, there is no need to register different domain names.
Editing operating system's hosts file can do the trick.

```shell
$ sudo vim /etc/hosts
```

Add the following entries

```shell
# refer to localhost
127.0.0.1 publisher.net
127.0.0.1 sdk.net
```

Every website URL becomes accessible via `http://publisher.net` and `http://sdk.net`

### Developer Tools

Browsers come with debugging tools specific to every vendor. Obviously, these tools can be used to debug SDK JavaScript code - `Chrome Developer Tools` `Safari Developer Tools` `Firebug`. Developer tools also shortened as DevTools.

> The DevTools provide web developers deep access into the internals of the browser and their web application. Use the DevTools to efficiently track down layout issues, set JavaScript breakpoints, and get insights for code optimization.

### Console Logs

For testing expected output text and other general debugging, `Console Logs` can be used through the browser API `console.log()`. There are various typeways to format and output messages. There is more on this discussed at this link: [Console API](https://developer.mozilla.org/en/docs/Web/API/console).

![screen shot 2015-06-15 at 3 50 23 pm](https://cloud.githubusercontent.com/assets/2560096/8155377/411fb24a-1376-11e5-98da-f71f8ed29bcd.png)

### Debugging Proxy

Debugging proxy gives us a hand on testing SDK in development. 
Some of the areas covered are: 

- Debugging traffic
- modify cookies
- Inspecting headers
- Verifying the cache
- Editing http request/response
- SSL Proxying
- Debugging Ajax and more.

Here's some software you can try
- [FiddlerCore](http://www.telerik.com/fiddler/fiddlercore)
- [Charles](http://www.charlesproxy.com)
- [Cellist](https://itunes.apple.com/tw/app/cellist-http-debugging-proxy/id897814548?l=zh&mt=12)

### BrowserSync

[BrowserSync](http://www.browsersync.io) makes it easy to tweak and test faster by synchronizing file changes and interactions across multiple devices. It’s wicked-fast and totally free.

It really helps a lot to test the SDK across mutliple devices. Totally worth a try =)

### Debugging Node.js Apps

To debug SDK scripts in Chrome Developer Tools. (Node.js v6.3.0+ required)

```shell
  $ node --inspect-brk [script.js]
```

- [Official document](https://nodejs.org/en/docs/inspector/)
- [Paul Irish gave a talk in 15 minutes about how to use --inspect](https://www.youtube.com/watch?v=Xb_0awoShR8)

## Tips and Tricks

### Piggyback

Sometimes, including all the SDK source code is not required in some use cases.
That is the case of a simple 1x1 pixel request -- For example: make a request when someone lands on thank you (last) page. 
In such a scenario, the developer may include an image file with a the (url) link, as explained in the following snippet.

```html
<img height="1" width="1" alt="" style="display:none" src="https://yourUrlLink.com/t?timestamp=1234567890&type=page1&currency=USD&noscript=1" />
```

### Page Visibility API

Sometimes, the SDK wants to detect if a user has a particular page in focus. 
These polyfills  [visibly.js](https://github.com/addyosmani/visibly.js) and [visibilityjs](https://github.com/ai/visibilityjs) may help achieve just that.

### Document Referrer

The `document.referrer` can be used to get the url of current or previous page. 
It is however advised to remember that this referrer is "Browser Referrer" not the "Human Known Referrer". 
The case where a user clicks the **browser back button**, for example pageA -> pageB -> pageC -> (back button) pageB, current pageB's referrer is pageA, not pageC.

### Console Logs Polyfill

The following is not a special polyfill. It just makes sure that calling `console.log` API doesn't throw error event to client-side.

```js
if (typeof console === "undefined") {
    var f = function() {};
    console = {
        log: f,
        debug: f,
        error: f,
        info: f
    };
}
```

### EncodeURI or EncodeURIComponent

Understand the difference between `escape()` `encodeURI()` `encodeURIComponent()` [here](http://stackoverflow.com/a/3608791/1748884).

It's worth mentioning that using `encodeURI()` and `encodeURIComponent()` has exactly 11 characters different.
These characters are: # $ & + , / : ; = ? @ [more discussion](http://stackoverflow.com/a/23842171/1748884).

<h2 align="center">
 <img src="http://i.imgur.com/rHWC1r1.png" alt="encodeuri or encodeuricomponent"/>
</h2>

### YOU MIGHT NOT NEED JQUERY

As the title said, [you might not need jquery](http://youmightnotneedjquery.com/). It's really useful if you are looking for some utilities code - [AJAX](http://youmightnotneedjquery.com/#AJAX) [EFFECTS](http://youmightnotneedjquery.com/#effects), [ELEMENTS](http://youmightnotneedjquery.com/#elements), [EVENTS](http://youmightnotneedjquery.com/#events), [UTILS](http://youmightnotneedjquery.com/#utils)

### You Don't Need jQuery

> Free yourself from the chains of jQuery by embracing and understanding the modern Web API and discovering various directed libraries to help you fill in the gaps.

[http://blog.garstasio.com/you-dont-need-jquery/](http://blog.garstasio.com/you-dont-need-jquery/)

Useful Tips

1. [Selecting Elements](http://blog.garstasio.com/you-dont-need-jquery/selectors/)
2. [DOM Manipulation](http://blog.garstasio.com/you-dont-need-jquery/dom-manipulation/)

### Load Script with Callback

It's similar to [asynchrnous script loading](#asynchronous-syntax) with additional callback event

```js
function loadScript(url, callback) {
  var script = document.createElement('script');
  script.async = true;
  script.src = url;

  var entry = document.getElementsByTagName('script')[0];
  entry.parentNode.insertBefore(script, entry);

  script.onload = script.onreadystatechange = function () {
    var rdyState = script.readyState;

    if (!rdyState || /complete|loaded/.test(script.readyState)) {
      callback();

      // detach the event handler to avoid memory leaks in IE (http://mng.bz/W8fx)
      script.onload = null;
      script.onreadystatechange = null;
    }
  };
}
```

### Once Function

Implementation of the function `once`

> Quite often, there are functions that are needed only to run once.  Oftentimes these functions are in the form of event listeners which may be difficult to manage.  Of course if they were easy to manage, it is advised to just remove the listeners. The following is the JavaScript function to make that possible!

```js
// Copy from DWB
// http://davidwalsh.name/javascript-once
function once(fn, context) {
	var result;

	return function() {
		if(fn) {
			result = fn.apply(context || this, arguments);
			fn = null;
		}

		return result;
	};
}

// Usage
var canOnlyFireOnce = once(function() {
	console.log('Fired!');
});

canOnlyFireOnce(); // "Fired!"
canOnlyFireOnce(); // nada. nothing.
```

### Pixel Ratio Density

To better understand terms such as pixel, ratio, density, dimension are while developing mobile web -- the following links can provide more insights:

- [Device pixel ratio - Mobile Web Development](https://www.youtube.com/watch?v=u0rfDeaxehc)
- [Mobile device pixels - Mobile Web Development](https://www.youtube.com/watch?t=34&v=UUF4jD-xoYc)

### Get Style Value

**Get inline-style value**

```html
<span id="black" style="color: black"> This is black color span </span>
<script>
    document.getElementById('black').style.color; // => black
</script>
```

**Get Real style value**

```html
<style>
#black {
    color: red !important;
}
</style>

<span id="black" style="color: black"> This is black color span </span>

<script>
    document.getElementById('black').style.color; // => black

    // real
    var black = document.getElementById('black');
    window.getComputedStyle(black, null).getPropertyValue('color'); // => rgb(255, 0, 0)
</script>
```

ref: [https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)

### Check if Element in Viewport

There is more [here](http://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport/7557433#7557433).

```js
function isElementInViewport (el) {

    //special bonus for those using jQuery
    if (typeof jQuery === "function" && el instanceof jQuery) {
        el = el[0];
    }

    var rect = el.getBoundingClientRect();

    return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) && /*or $(window).height() */
        rect.right <= (window.innerWidth || document.documentElement.clientWidth) /*or $(window).width() */
    );
}
```

### Check if Element is Visible

```js
var isVisible = function(b) {
    var a = window.getComputedStyle(b);
    return 0 === a.getPropertyValue("opacity") || "none" === a.getPropertyValue("display") || "hidden" === a.getPropertyValue("visibility") || 0 === parseInt(b.style.opacity, 10) || "none" === b.style.display || "hidden" === b.style.visibility ? false : true;
}

var element = document.getElementById('box');
isVisible(element); // => false or true
```

### Get Viewport Size

```js
var getViewportSize = function() {
    try {
        var doc = top.document.documentElement
          , g = (e = top.document.body) && top.document.clientWidth && top.document.clientHeight;
    } catch (e) {
        var doc = document.documentElement
          , g = (e = document.body) && document.clientWidth && document.clientHeight;
    }
    var vp = [];
    doc && doc.clientWidth && doc.clientHeight && ("CSS1Compat" === document.compatMode || !g) ? vp = [doc.clientWidth, doc.clientHeight] : g && (vp = [doc.clientWidth, doc.clientHeight]);
    return vp;
}

// return as array [viewport_width, viewport_height]
```

### User Tracking

Assuming that an Evil Advertisement Company wants to track a user, Evil may well generate a personalized unique hash by using [fingerprinting](https://github.com/Valve/fingerprintjs2). However, Ethical Company uses cookies and offers [Opt-out](https://en.wikipedia.org/wiki/Opt-out) solution.

### Opt Out

[DIGITAL ADVERTISING ALLIANCE, POWERED BY YOURADCHOICES](http://optout.aboutads.info/#!/) provides a tool that helps anyone to opt-out from all the participating companies.

## WTF

### Misspelling Of Referrer

Fun fact about why the HTTP Request Header having the field name `referer` not `referrer`

_According to the [Wikipedia](https://en.wikipedia.org/wiki/HTTP_referer#Etymology)_

> The `misspelling of referrer` originated in the original proposal by computer scientist `Phillip Hallam-Baker` to incorporate the field into the HTTP specification. The misspelling was set in stone by the time of its incorporation into the `Request for Comments` standards document `RFC 1945`; document co-author `Roy Fielding` has remarked that neither "referrer" nor the misspelling "referer" were recognized by the standard `Unix spell checker` of the period. "Referer" has since become a widely used spelling in the industry when discussing HTTP referrers; usage of the misspelling is not universal, though, as the correct spelling "referrer" is used in some web specifications such as the `Document Object Model`.

<h2 align="center">
    <img src="https://cloud.githubusercontent.com/assets/2560096/12190613/96a7d652-b605-11e5-8109-1c6fefa9dfc4.png"/>
</h2>

## Template

This guide provides **templates** and **boilerplates** to building an SDK. 
  
- [TEMPLATE.md](./Template/README.md)

## Books/Nice to Reads

1. [Third-Party JavaScript](http://thirdpartyjs.com)
2. [JQuery Plugin](https://learn.jquery.com/plugins/)
3. [LightningJS](https://github.com/olark/lightningjs)

*(inspired by [http-api-design](https://github.com/interagent/http-api-design))*

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore -->
<table>
  <tr>
    <td align="center"><a href="https://www.linkedin.com/in/hueitan"><img src="https://avatars0.githubusercontent.com/u/2560096?v=4" width="100px;" alt="Huei Tan"/><br /><sub><b>Huei Tan</b></sub></a><br /><a href="https://github.com/hueitan/javascript-sdk-design/commits?author=hueitan" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/murindwaz"><img src="https://avatars1.githubusercontent.com/u/259806?v=4" width="100px;" alt="Pascal Maniraho"/><br /><sub><b>Pascal Maniraho</b></sub></a><br /><a href="https://github.com/hueitan/javascript-sdk-design/commits?author=murindwaz" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/adamellsworth"><img src="https://avatars0.githubusercontent.com/u/7034617?v=4" width="100px;" alt="Adam"/><br /><sub><b>Adam</b></sub></a><br /><a href="#content-adamellsworth" title="Content">🖋</a></td>
  </tr>
</table>

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
