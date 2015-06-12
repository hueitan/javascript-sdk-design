# Javascript SDK Design Guide

## Introduction

This guide gives you an introduction to develop a javascript sdk
on desktop and mobile web in different platforms and browsers (<99.99% I might skip some browsers).

Since I didn't find out a better documentation for the javascript sdk,
I'm here to collect and note down the knowledges from my experiences.

Feel free to [edit](https://github.com/huei90/javascript-sdk-design/edit/master/README.md) it if you think there's a better design methodology or
any others I didn't mention here.

*(inspired by [http-api-design](https://github.com/interagent/http-api-design))*

## Content
* [What is SDK](#what-is-sdk)
* [Design Philosophy](#design-philosophy)
* [Include the SDK](#include-the-sdk)
  * [Asynchronous Syntax](#asynchronous-syntax)
  * [Traditional Syntax](#traditional-syntax)
  * [Comparison](#comparison)
  * [Problem of Asynchronous](#problem-of-asynchronous)
* [SDK Versioning](#sdk-versioning)
* [Changelog Document](#changelog-document)
* [Namespace](#namespace)
* [Storage Mechanism](#storage-mechanism)
  * [Cookie](#cookie)
    * [Check Cookie Writable](#check-cookie-writable)
    * [Check Third Party Cookie Writable](#check-third-party-cookie-writable)
  * [Session](#session)
  * [LocalStorage](#localstorage)
    * [Check LocalStorage Writable](#check-localstorage-writable)
  * [Session Storage](#session-storage)
    * [Check SessionStorage Writable](#check-sessionstorage-writable)
* [Event](#event)
  * [Document Ready](#document-ready)
* [Request](#request)
  * [Image Beacon](#image-beacon)
  * [Single Post](#single-post)
  * [Multiple Post](#multiple-post)
  * [Iframe](#iframe)
  * [Script jsonp](#script-jsonp)
  * [Navigator.sendBeacon()](#navigatorsendbeacon)
  * [XMLHttpRequest](#xmlhttprequest)
* [Component of URI](#component-of-uri)
 * [Parsing URI](#parsing-uri)
* [Tips and Tricks](#tips-and-tricks)
 * [Piggyback](#piggyback)
 * [BrowserSync](#browsersync)
 * [Page Visibility API](#page-visibility-api)
 * [Document Referrer](#document-referrer)
 * [EncodeURI or EncodeURIComponent](#encodeuri-or-encodeuricomponent)
* [Example](#example)
* [Book to Read](#book-to-read)
* [Reference](#reference)

## What is SDK

I know it's very common, but it is.

"Short for **software development kit**, a programming package that enables a programmer to develop applications for a specific platform. Typically an SDK includes one or more APIs, programming tools, and documentation."

## Design Philosophy

It depends on your purpose of your SDK service and usage,
but there must be **native**, **short**, **fast**, **clean**, **readable** and **testable**.

Written in native javascript code, compiler language like
livescript, coffeescript, typescript and others are not recommend.
There must be a better way to write your own javascript code in native faster than others.

Please don't involve jQuery in your SDK unless it's really important,
you can have other jQuery-like libraries, zepto.js, for the DOM manipulation.
Or if you need the http ajax request, use other light library like `window.fetch`.

Once every SDK version released, make sure that it can be fitted into older and newer SDK version in the future.
Therefore, remember to write your **Documentation** for your SDK, comment for your code, unit test and user scenario test.

## Include the SDK

You are suggest to use **Asynchrnonous Syntax** for your script loaded.
We want to optimize the user experience on the website as we don't want our SDK library interfere the main web loaded.

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

Target on modern browser, you can use `async`.

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

For the futher development, you are advised to use  `stable` `unstable` `alpha` `latest` `experimental` version.
```
http://xxx.com/sdk-stable.js
http://xxx.com/sdk-unstable.js
http://xxx.com/sdk-alpha.js
http://xxx.com/sdk-latest.js
http://xxx.com/sdk-experimental.js
```

## Changelog Document

You should notice that your SDK user will not know if you upgrade your sdk without announcement.
Remember to write a changelog to document your major, minor and even bug fix change.
It will be a good developer experience if we can trace the changing API for the SDK. - [Keep a Changelog](http://keepachangelog.com), [Github Repo](https://github.com/olivierlacan/keep-a-changelog)

## Namespace

You should not define more than one global namespace in your SDK and
prevent using the common word for your namespace to avoid collision with other libraries.

On your SDK mainland, you should use `(function () { ... })()` to wrap all your source.

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

It's impossible to check only using client side javascript, you need a server to do that. ([Example](https://dl.dropboxusercontent.com/u/105727/web/3rd/third-party-cookies.html))

### Session

It's important to know that Javascript is not possible to write Session,
please refer to the server side team to implement Session.

### LocalStorage

Stores data with no expiration date, storage limit is far larger (at least 5MB) and
information is never transferred to the server.

It's good to know that each localStorage from `http` and `https` in same domain aren't shared.
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

In client browser, there are events `load` `unload` `on` `off` `bind` .... Here's some polyfills for you to handle all different platforms.

### Document Ready

```js
// handle IE8+
function ready (fn) {
    if (document.readyState != 'loading') {
        fn();
    } else if (document.addEventListener) {
        document.addEventListener('DOMContentLoaded', fn);
    } else {
        document.attachEvent('onreadystatechange', function() {
            if (document.readyState != 'loading')
                fn();
            });
    }
}

```

## Request

The communication between our SDK and Server is using Ajax Request,
as we know we can use jQuery ajax http request to communicate with Server,
but there's a better solution to implement it.

### Image Beacon

Use the Image Beacon to ask browser to perform a method GET Request to get an Image.

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

body.append(iframe);
```

### Script jsonp

This is the case that your server need to response javascript code and let the client browser execute it.
Just include the JS script link.

```js
  (function () {
    var s = document.createElement('script');
    s.type = 'text/javascript';
    s.async = true;
    s.src = '/yourscript?some=parameter';
    var x = document.getElementsByTagName('script')[0];
    x.parentNode.insertBefore(s, x);
  })();
```

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

## Component of URI

It's important to know if your SDK need to parse the location url.
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

Here's a simplest by using the native URL() Interface but it doesn't support on all the browsers.

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

## Tips and Tricks

### Piggyback

Sometimes, we don't want our developers include all the SDK source,
we just need to do a simple 1x1 pixel request (for example: return a request when landing on thank you/last page). All we need to do is ask the developer to include an image file with our url link.

```html
<img height="1" width="1" alt="" style="display:none" src="https://yourUrlLink.com/t?timestamp=1234567890&type=page1&currency=USD&noscript=1" />
```

### BrowserSync

[BrowserSync](http://www.browsersync.io) makes your tweaking and testing faster by synchronising file changes and interactions across multiple devices. It’s wicked-fast and totally free.

It really helps alot if you need to test your sdk result in multiple cross devices. Try it =)

### Page Visibility API

Sometimes, your SDK wants to detect the user is whether focus your page. Try the polyfills  [visibly.js](https://github.com/addyosmani/visibly.js) and [visibilityjs](https://github.com/ai/visibilityjs).

### Document Referrer

Use `document.referrer` to get the url of current previous page. But remember that this referrer is "Browser Referrer" not the "Human Known Referrer". If you click the **browser back button**, for example pageA -> pageB -> pageC -> (back button) pageB, current pageB's referrer is pageA, not pageC.

### EncodeURI or EncodeURIComponent

Understand the difference between `escape()` `encodeURI()` `encodeURIComponent()` [here](http://stackoverflow.com/a/3608791/1748884).

Remember that using `encodeURI()` and `encodeURIComponent()` has exactly 11 characters difference.
These characters are: # $ & + , / : ; = ? @ [more discussion](http://stackoverflow.com/a/23842171/1748884).

## Example

See next page [EXAMPLE.md](https://github.com/huei90/javascript-sdk-design/blob/master/EXAMPLE.md) to find out how others provide their javascript sdk.

## Book to Read

1. [Third-Party Javascript](http://thirdpartyjs.com)

## Reference

1. [What is Software Development Kit](http://www.webopedia.com/TERM/S/SDK.html)
2. [A window.fetch JavaScript polyfill.](https://github.com/github/fetch)
3. [POST Request](http://stackoverflow.com/questions/692196/post-request-javascript/25423688#25423688)
4. [Semantic VersioningVersioning 2.0.0](http://semver.org)
5. [HTTP API design guide extracted from work on the Heroku Platform API](https://github.com/interagent/http-api-design)
6. [Understanding URIs](http://medialize.github.io/URI.js/about-uris.html)
7. [URI Parsing with Javascript](https://gist.github.com/jlong/2428561)
8. [Modernizr: the feature detection library for HTML5/CSS3](http://modernizr.com)
9. [HTML5 Web Storage](http://www.w3schools.com/html/html5_webstorage.asp)
10. [Check if third-party cookies are enabled](http://stackoverflow.com/questions/3550790/check-if-third-party-cookies-are-enabled)
11. [Introduction to Analytics.js - Universal Analytics Web Tracking](https://developers.google.com/analytics/devguides/collection/analyticsjs/)
12. [Facebook Conversion Tracking Pixel](https://www.facebook.com/help/421433191260652)
13. [What is the maximum length of a URL](http://www.boutell.com/newfaq/misc/urllength.html)
