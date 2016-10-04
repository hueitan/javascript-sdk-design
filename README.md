# JavaScript SDK Design Guide

## Introduction

This guide gives you an introduction to developing a [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) SDK on desktop and mobile web in different platforms and browsers (<99.99% I might skip some browsers), for those developed for non-browser supports (hardware, embedded, node/io js) are excluded in this document and will be considered in the future.

Since I didn't find out a better documentation for the [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) SDK,
I'm here to gather and note down the knowledge of my personal experiences. **JavaScript-SDK-Design** is not just about SDK only, it's the connection between user and browser machine. The more native we write, the more we think, we do care about the performances and differences between platforms and browsers.

Feel free to [edit](https://github.com/huei90/JavaScript-sdk-design/edit/master/README.md) or you can drop me suggestions on the [issue list](https://github.com/huei90/JavaScript-sdk-design/issues).

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
* [What is SDK](#what-is-sdk)
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
    * [Check Third Party Cookie Writable](#check-third-party-cookie-writable)
    * [Write Read Remove Cookie Code](#write-read-remove-cookie-code)
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
* [WTF](#wtf)
 * [Misspelling Of Referrer](#misspelling-of-referrer)
* [Template](#template)
* [Book to Read](#booknice-to-read)
* [Reference](#reference)

## What is SDK

I know it's very common, but it is.

"Short for **software development kit**, a programming package that enables a programmer to develop applications for a specific platform. Typically an SDK includes one or more APIs, programming tools, and documentation."

## Design Philosophy

It depends on your purpose of your SDK service and usage,
but there must be **native**, **short**, **fast**, **clean**, **readable** and **testable**.

Written in native JavaScript code, compiler language like
Livescript, Coffeescript, Typescript and others are **not** recommend.
There must be a better way to write your own JavaScript code in native faster than others.

Please **don't** involve jQuery in your SDK unless it's really important,
you can have other jQuery-like libraries, zepto.js, for the DOM manipulation.
Or if you need the HTTP [ajax request](https://github.com/huei90/JavaScript-sdk-design#request), use another light library like `window.fetch`.

Once every SDK version released, make sure that it can be fitted into older and newer SDK version in the future.
Hence, remember to write your **Documentation** for your SDK, comment in your code, unit test and user scenario test.

## Scope

*Based on the book [Third-Party JavaScript](http://thirdpartyjs.com)*

In which case, you should design a JavaScript SDK for your application?

1. [Embedded widgets](https://github.com/huei90/javascript-sdk-design/blob/master/SCOPE.md#embedded-widgets) - Small interactive applications embedded on the publisher's web page (Disqus, Google Maps, Facebook Widget)
2. [Analytics and metrics](https://github.com/huei90/javascript-sdk-design/blob/master/SCOPE.md#analytics-and-metrics) - For gathering intelligence about visitors and how they interact with the publisher's website (GA, Flurry, Mixpanel)
3. [Web service API wrappers](https://github.com/huei90/javascript-sdk-design/blob/master/SCOPE.md#web-service-api-wrappers) - For developing client-side applications that communicate with external web services. (Facebook Graph API)

In what case we should use SDK in JavaScript environment? [Suggest one](https://github.com/huei90/JavaScript-sdk-design/edit/master/README.md).

## Include the SDK

You are suggested to use **Asynchronous Syntax** for your script loaded.
We want to optimize the user experience on the website as we don't want our SDK library to interfere the main web loaded.

### Asynchronous Syntax

```html
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();
</script>
```

Target on the modern browser, you can use `async`.

```html
<script async src="http://xxx.com/sdk.js"></script>
```

### Traditional Syntax

```html
<script type="text/javascript" src="http://xxx.com/sdk.js"></script>
```

### Comparison

Here's the simple graph to show the differentiate between Asynchronous and Traditional Syntax.

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
> You should avoid and minimize the use of blocking JavaScript, especially external scripts that must be fetched before they can be executed. Scripts that are necessary to render page content can be inlined to avoid extra network requests, however the inlined content needs to be small and must execute quickly to deliver good performance. Scripts that are not critical to initial render should be made asynchronous or deferred until after the first render.

### Problem of Asynchronous

When you are using Asynchronous,
you cannot execute your SDK function which script written within the page.

```js
<script>
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```

The result will lead to undefined because the `SDKName()` execute before the script loaded,
therefore we should do some tricks and make sure the script execute successfully.
The event will store in the `SDKName.q` queue array,
your SDK should handle and execute `SDKName.q` and reinitial the namespace `SDKName`.

```js
<script>
  (function () {
    // add a queue event here
    SDKName = SDKName || function () {
      (SDKName.q = SDKName.q || []).push(arguments);
    };
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName('some arguments');
</script>
```

**Or** using [].push

```js
<script>
  (function () {
    // add a queue event here
    SDKName = window.SDKName || (window.SDKName = []);
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = 'http://xxx.com/sdk.js';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();

  // execute your script immediately here
  SDKName.push(['some arguments']);
</script>
```

### Others

There are other different ways to include a script

**Import in ES2015**

```js
import "your-sdk";
```

**Modular include a Script**

For full source code, see this awesome tutorial. [Loading JavaScript Modules](https://libraryinstitute.wordpress.com/2010/12/01/loading-javascript-modules/)

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

Please avoid using your special case for version like `brand-v<timestamp>.js` `brand-v<datetime>.js` `brand-v1-v2.js`,
it may cause the developer who use the SDK on confusing which is the latest version.

Use [Semantic Versioning](http://semver.org) to define your SDK Version in the form "MAJOR.MINOR.PATCH".
Version in `v1.0.0` `v1.5.0` `v2.0.0` is easier for them to trace and search for the changelog documentation.

Normally, we can have different ways to state the SDK version, it depends on your service and design.

Using Query String path.
```
http://xxx.com/sdk.js?v=1.0.0
```

Using the Folder Naming.
```
http://xxx.com/v1.0.0/sdk.js
```

Using hostname (subdomain).
```
http://v1.xxx.com/sdk.js
```

For the further development, you are advised to use  `stable` `unstable` `alpha` `latest` `experimental` version.
```
http://xxx.com/sdk-stable.js
http://xxx.com/sdk-unstable.js
http://xxx.com/sdk-alpha.js
http://xxx.com/sdk-latest.js
http://xxx.com/sdk-experimental.js
```

## Changelog Document

You should notice that your SDK user will not know if you upgrade your SDK without announcement.
Remember to write a changelog to document your major, minor and even bug fix change.
It will be a good developer experience if we can trace the changing API for the SDK. - [Keep a Changelog](http://keepachangelog.com) ([Github Repo](https://github.com/olivierlacan/keep-a-changelog))

Each version should have:

```
[Added] for new features.
[Changed] for changes in existing functionality.
[Deprecated] for once-stable features removed in upcoming releases.
[Removed] for deprecated features removed in this release.
[Fixed] for any bug fixes.
[Security] to invite users to upgrade in case of vulnerabilities.
```

In addition, [commit-message-emoji](https://github.com/dannyfritz/commit-message-emoji) uses emoji to explain the commit's changes itself.

## Namespace

You should not define more than one global namespace in your SDK and
prevent using the common word for your namespace to avoid collision with other libraries.

On your SDK mainland, you should use `(function () { ... })()` to wrap all your source.

This is an increasingly common practice, employed by many popular JavaScript libraries (jQuery, Node.js, etc.). This technique creates a closure around the entire contents of the file which, perhaps most importantly, creates a private namespace and thereby helps avoid potential name clashes between different JavaScript modules and libraries. [#](http://www.toptal.com/javascript/interview-questions)

To avoid **namespace collision**

Learning this from Google Analytics, you can define your namespace by changing the value `ga`

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

The domain scope of using cookie is quite complex while involving the `subdomain` and `path`.

For `path=/`,
you have a cookie `first=value1` in domain `http://github.com`,
another cookie `second=value2` in domain `http://sub.github.com`

|  | http://github.com | http://sub.github.com
|:---------:|:------------:|:------------:
first=value1	 | ✓ | ✓
second=value2 | ✘ | ✓

You have a cookie `first=value1` in domain `http://github.com`,
cookie `second=value2` in domain path `http://github.com/path1`
and cookie `third=value3` in domain `http://sub.github.com`,


|  | http://github.com | http://github.com/path1 | http://sub.github.com
|:---------:|:------------:|:------------:|:------------:
first=value1	 | ✓ | ✓ | ✓
second=value2 | ✘ | ✓ | ✘
third=value3 | ✘ | ✘ | ✓

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

#### Check Third Party Cookie Writable

It's impossible to check only using client side JavaScript, you need a server to do that. ([Example](https://dl.dropboxusercontent.com/u/105727/web/3rd/third-party-cookies.html))

#### Write Read Remove Cookie Code

Snippet Code for write/read/remove cookie script.

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

It's important to know that JavaScript is **not possible** to write Session,
please refer to the server side team to implement Session.

A page session lasts for as long as the browser is open and survives over page reloads and restores. Opening a page in a new tab or window will cause a new session to be initiated.

### LocalStorage

Stores data with no expiration date, storage limit is far larger (at least 5MB) and
information is never transferred to the server.

It's good to know that each localStorage from `http` and `https` in the same domain aren't shared.
You can create an iframe inside the website and use `postMessage` to pass the value to others. [HOW TO?](http://stackoverflow.com/questions/10502469/is-there-any-workaround-to-make-use-of-html5-localstorage-on-both-http-and-https)

#### Check LocalStorage Writable

window.localStorage is not support in every browser, SDK should check available before using it.

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

Please do make sure that the entire page is finished loading(ready) before starting execution the SDK functions.

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

> Information from JS Tip - https://github.com/loverajoel/jstips/blob/gh-pages/_posts/en/2016-02-15-detect-document-ready-in-pure-js.md

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

The post message data should be **String**, for more advanced used in json, use **JSON String**. Although the modern browser do support [Structured Clone Algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm) on the parameter, but not the 100% browser.

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

In web page, use CSS style `overflow: hidden`, in some mobile web, this css doesn't work, use javascript event.

```js
document.addEventListener('touchstart', function(e){ e.preventDefault(); }); // prevent scroll
// or 
document.body.addEventListener('touchstart', function(e){ e.preventDefault(); }); // prevent scroll
// use move if you need some touch event
document.addEventListener('touchmove', function(e){ e.preventDefault(); }); // prevent scroll
```

## Request

The communication between our SDK and Server is using Ajax Request,
as we know we can use jQuery ajax http request to communicate with Server,
but there's a better solution to implement it.

### Image Beacon

Use the Image Beacon to ask the browser to perform a method GET Request to get an Image.

*Remember to add timestamp (Cache Buster) to prevent caching in browser.*

```js
(new Image()).src = 'http://xxxxx.com/collect?id=1111';
```

Some notice for GET Query String, there is the limit of length which is 2048(Basically It depends on different browser and server). You should do some tricks to handle if exceed length limit.

```js
if (length > 2048) {
    // do Multiple Post (form)
} else {
    // do Image Beacon
}
```

You may have the problem on `encodeURI` or `encodeURIComponent`, it's better if you understand them. [See below](#encodeuri-or-encodeuricomponent).

For the image load **success/error callback**

```js
var img = new Image();
img.src = 'http://xxxxx.com/collect?id=1111';
img.onload = successCallback;
img.onerror = errorCallback;
```

### Single Post

Use the native form element method POST to send a key value.

```js
var form = document.createElement('form');
var input = document.createElement('input');

form.style.display = 'none';
form.setAttribute('method', 'POST');
form.setAttribute('action', 'http://xxxx.com/track');

input.name = 'username';
input.value = 'attacker';

form.appendChild(input);
document.getElementsByTagName('body')[0].appendChild(form);

form.submit();
```

### Multiple Post

The Service is often complex, we need to send more data through method POST.

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

When you need to generate a content within the page, you can use iframe to embed your html.

```js
var iframe = document.createElement('iframe');
var body = document.getElementsByTagName('body')[0];

iframe.style.display = 'none';
iframe.src = 'http://xxxx.com/page';
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

This is the case that your server needs to response JavaScript code and let the client browser execute it.
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

Know more about jsonp

1. JSONP only works in GET HTTP request.
2. JSONP lacks error handling, means you cannot detect case in response status code 404, 500 and so on.
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

Writing XMLHttpRequest is not a good idea. I assume that you don't want to waste time on battling with IE or other browsers. Here's some polyfills or code you can try

1. [window.fetch](https://github.com/github/fetch) - A window.fetch JavaScript polyfill.
2. [got](https://github.com/sindresorhus/got) - Simplified HTTP/HTTPS requests
3. [microjs](http://microjs.com/#ajax) - list of ajax lib
4. more

### Fragment Identifier

Also known as hash mark `#`. Remember that request with hash mark at the end is not pass within http request.

For example, you are in the page `http://github.com/awesome#huei90`

```js
// Sending a request with a parameter url which contains current url
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome#huei90';

// actual request will be without #
(new Image()).src = 'http://yourrequest.com?url=http://github.com/awesome';

// Solved, encodeURIComponent(url);
(new Image()).src = 'http://yourrequest.com?url=' + encodeURIComponent('http://github.com/awesome#huei90');
```

### Maximum Number of Connection

Check the maximum number of browser's request connection. [browserscope](http://www.browserscope.org/?category=network&v=top)

<h2 align="center">
 <img src="https://cloud.githubusercontent.com/assets/2560096/9082891/ac4dc26e-3b99-11e5-8178-606270a801c4.png" alt="max number of connection"/>
</h2>

## Component of URI

It's important to know if your SDK needs to parse the location url.
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

Here's a simplest by using the native URL() Interface but it doesn't support all the browsers also [not standard yet](https://developer.mozilla.org/en-US/docs/Web/API/Window/URL).

```js
var parser = new URL('http://github.com/huei90');
parser.hostname; // => "github.com"
```

For the browser which doesn't have the URL() Interface, try DOM createElement('a').

```js
var parser = document.createElement('a');
parser.href = "http://github.com/huei90";
parser.hostname; // => "github.com"
```

## Debugging

### Simulating Multiple Domains

You don't need to register different domain names to simulate multiple domain, you can just edit your operating system's hosts file.

```shell
$ sudo vim /etc/hosts
```

Add the following entries

```shell
# refer to localhost
127.0.0.1 publisher.net
127.0.0.1 sdk.net
```

Then you can access the page `http://publisher.net` and `http://sdk.net`

### Developer Tools

Use the web debugging tools from browser vendors when debugging your SDK JavaScript code - `Chrome Developer Tools` `Safari Developer Tools` `Firebug`. Developer tools also short for DevTools.

> The DevTools provide web developers deep access into the internals of the browser and their web application. Use the DevTools to efficiently track down layout issues, set JavaScript breakpoints, and get insights for code optimization.

### Console Logs

For testing expected output text and other general debugging, `Console Logs` can be used throught the browser API `console.log()`. There are various typeways to format and output your messages, find out more [Console API](https://developer.mozilla.org/en/docs/Web/API/console).

![screen shot 2015-06-15 at 3 50 23 pm](https://cloud.githubusercontent.com/assets/2560096/8155377/411fb24a-1376-11e5-98da-f71f8ed29bcd.png)

### Debugging Proxy

Sometimes debugging proxy will give you a hand on testing your SDK. Debug traffic, modify cookies, headers, cache, edit http request/response, SSL Proxying, ajax debugging and more.

Here's some software you can try
- [FiddlerCore](http://www.telerik.com/fiddler/fiddlercore)
- [Charles](http://www.charlesproxy.com)
- [Cellist](https://itunes.apple.com/tw/app/cellist-http-debugging-proxy/id897814548?l=zh&mt=12)

### BrowserSync

[BrowserSync](http://www.browsersync.io) makes your tweaking and testing faster by synchronising file changes and interactions across multiple devices. It’s wicked-fast and totally free.

It really helps a lot if you need to test your SDK result in multiple cross devices. Try it =)

## Tips and Tricks

### Piggyback

Sometimes, we don't want our developers include all the SDK source,
we just need to do a simple 1x1 pixel request (for example: return a request when landing on thank you/last page). All we need to do is ask the developer to include an image file with our url link.

```html
<img height="1" width="1" alt="" style="display:none" src="https://yourUrlLink.com/t?timestamp=1234567890&type=page1&currency=USD&noscript=1" />
```

### Page Visibility API

Sometimes, your SDK wants to detect if the user has your page in focus. Try the polyfills  [visibly.js](https://github.com/addyosmani/visibly.js) and [visibilityjs](https://github.com/ai/visibilityjs).

### Document Referrer

Use `document.referrer` to get the url of current previous page. But remember that this referrer is "Browser Referrer" not the "Human Known Referrer". If you click the **browser back button**, for example pageA -> pageB -> pageC -> (back button) pageB, current pageB's referrer is pageA, not pageC.

### Console Logs Polyfill

It's not a real polyfill, just make sure that calling `console.log` API doesn't throw error event to client side.

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

Remember that using `encodeURI()` and `encodeURIComponent()` has exactly 11 characters difference.
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

It's similar to [asynchrnous script loading](#asynchronous-syntax) with addition callback event

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

> Every so often you have a function which you only want to run once.  Oftentimes these functions are in the form of event listeners which may be difficult to manage.  Of course if they were easy to manage, you'd just remove the listeners but that's a perfect world and sometimes you simply want the ability to only allow a function to be called once.  Here's the JavaScript function to make that possible!

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
canOnlyFireOnce(); // nada
```

### Pixel Ratio Density

If you stuck on the term pixel, ratio, density, dimension, what while developing mobile web, try understanding the video, it may help.

[Device pixel ratio - Mobile Web Development](https://www.youtube.com/watch?v=u0rfDeaxehc) <br/>
[Mobile device pixels - Mobile Web Development](https://www.youtube.com/watch?t=34&v=UUF4jD-xoYc)

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

You can find out more [here](http://stackoverflow.com/questions/123999/how-to-tell-if-a-dom-element-is-visible-in-the-current-viewport/7557433#7557433).

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
var isVisible: function(b) {
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

// return as rray [viewport_width, viewport_height]
```

## WTF

### Misspelling Of Referrer

Do you know that why the HTTP Request Header having the field name `referer` not `referrer`

_According to the [Wikipedia](https://en.wikipedia.org/wiki/HTTP_referer#Etymology)_

> The `misspelling of referrer` originated in the original proposal by computer scientist `Phillip Hallam-Baker` to incorporate the field into the HTTP specification. The misspelling was set in stone by the time of its incorporation into the `Request for Comments` standards document `RFC 1945`; document co-author `Roy Fielding` has remarked that neither "referrer" nor the misspelling "referer" were recognized by the standard `Unix spell checker` of the period. "Referer" has since become a widely used spelling in the industry when discussing HTTP referrers; usage of the misspelling is not universal, though, as the correct spelling "referrer" is used in some web specifications such as the `Document Object Model`.

<h2 align="center">
    <img src="https://cloud.githubusercontent.com/assets/2560096/12190613/96a7d652-b605-11e5-8109-1c6fefa9dfc4.png"/>
</h2>

## Template

Someone asks for the template/boilerplate of the SDK, here some examples for you. [TEMPLATE.md](https://github.com/huei90/JavaScript-sdk-design/blob/master/Template/README.md)

## Book/Nice to Read

1. [Third-Party JavaScript](http://thirdpartyjs.com)
2. [JQuery Plugin](https://learn.jquery.com/plugins/)
3. [LightningJS](https://github.com/olark/lightningjs)

## Reference

1. [What is Software Development Kit](http://www.webopedia.com/TERM/S/SDK.html)
2. [A window.fetch JavaScript polyfill.](https://github.com/github/fetch)
3. [POST Request](http://stackoverflow.com/questions/692196/post-request-JavaScript/25423688#25423688)
4. [Semantic VersioningVersioning 2.0.0](http://semver.org)
5. [HTTP API design guide extracted from work on the Heroku Platform API](https://github.com/interagent/http-api-design)
6. [Understanding URIs](http://medialize.github.io/URI.js/about-uris.html)
7. [URI Parsing with JavaScript](https://gist.github.com/jlong/2428561)
8. [Modernizr: the feature detection library for HTML5/CSS3](http://modernizr.com)
9. [HTML5 Web Storage](http://www.w3schools.com/html/html5_webstorage.asp)
10. [Check if third-party cookies are enabled](http://stackoverflow.com/questions/3550790/check-if-third-party-cookies-are-enabled)
11. [Introduction to Analytics.js - Universal Analytics Web Tracking](https://developers.google.com/analytics/devguides/collection/analyticsjs/)
12. [Facebook Conversion Tracking Pixel](https://www.facebook.com/help/421433191260652)
13. [What is the maximum length of a URL](http://www.boutell.com/newfaq/misc/urllength.html)
14. [YOU MIGHT NOT NEED JQUERY](http://youmightnotneedjquery.com/)
15. [What is a Polyfill?](https://remysharp.com/2010/10/08/what-is-a-polyfill)
16. [Asynchronous and deferred JavaScript execution explained](http://peter.sh/experiments/asynchronous-and-deferred-javascript-execution-explained/)
17. [generate random UUIDs](https://gist.github.com/jed/982883)
18. [DOMContentLoaded and Load Event](http://stackoverflow.com/questions/2414750/difference-between-domcontentloaded-and-load-events)
19. [Must See JavaScript Dev Tools That Put Other Dev Tools to Shame](https://medium.com/javascript-scene/must-see-javascript-dev-tools-that-put-other-dev-tools-to-shame-aca6d3e3d925)
20. [Learn the art of writing quality READMEs.](https://github.com/noffle/art-of-readme)

*(inspired by [http-api-design](https://github.com/interagent/http-api-design))*
