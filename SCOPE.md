# TOC
* [Embedded widgets](#embedded-widgets)
  * [Disqus for Web](#disqus-for-web)
  * [Google Maps Async load](#google-maps-async-load)
  * [UserVoice Javascript SDK](#uservoice-javascript-sdk)
  * [Facebook SDK](#facebook-sdk)
  * [IMA HTML5 SDK](#ima-html5-sdk)
* [Analytics and Metrics](#analytics-and-metrics)
  * [Google Analytics](#google-analytics) 
  * [Mixpanel](#mixpanel)
* [Web service API wrappers](#web-service-api-wrappers)
  * [Google Tag Manager](#google-tag-manager)
* [More](#more)

## Embedded widgets

Small interactive applications embedded on the publisher's web page.

## Disqus for Web

> https://help.disqus.com/customer/portal/articles/472097-universal-embed-code

```html
<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = '<example>'; // Required - Replace '<example>' with your forum shortname

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
```

![screen shot 2015-12-04 at 10 47 38 am](https://cloud.githubusercontent.com/assets/2560096/11580590/f679d9b6-9a73-11e5-9e0c-2c7f2b88f506.png)

## Google Maps Async load

> https://developers.google.com/maps/documentation/javascript/tutorial

```js
function initialize() {
  var mapOptions = {
    zoom: 8,
    center: new google.maps.LatLng(-34.397, 150.644)
  };

  var map = new google.maps.Map(document.getElementById('map-canvas'),
      mapOptions);
}

function loadScript() {
  var script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = 'https://maps.googleapis.com/maps/api/js?v=3.exp' +
      '&signed_in=true&callback=initialize';
  document.body.appendChild(script);
}

window.onload = loadScript;
```

![screen shot 2015-12-04 at 10 49 34 am](https://cloud.githubusercontent.com/assets/2560096/11580606/18879732-9a74-11e5-81a6-849f068e602d.png)

## UserVoice Javascript SDK

> https://developer.uservoice.com/docs/widgets/install/

```js
// Include the UserVoice JavaScript SDK (only needed once on a page)
UserVoice=window.UserVoice||[];
(function(){
  var uv=document.createElement('script');
  uv.type='text/javascript';
  uv.async=true;
  uv.src='//widget.uservoice.com/[YOUR_API_KEY].js';
  var s=document.getElementsByTagName('script')[0];
  s.parentNode.insertBefore(uv,s)
})();
```

![screen shot 2015-12-04 at 10 51 55 am](https://cloud.githubusercontent.com/assets/2560096/11580647/6163734a-9a74-11e5-82b7-2c6858aef1bd.png)

### Facebook SDK

> https://developers.facebook.com/docs/javascript/quickstart/v2.3

```html
<script>
  window.fbAsyncInit = function() {
    FB.init({
      appId      : 'your-app-id',
      xfbml      : true,
      version    : 'v2.3'
    });
  };

  (function(d, s, id){
     var js, fjs = d.getElementsByTagName(s)[0];
     if (d.getElementById(id)) {return;}
     js = d.createElement(s); js.id = id;
     js.src = "//connect.facebook.net/en_US/sdk.js";
     fjs.parentNode.insertBefore(js, fjs);
   }(document, 'script', 'facebook-jssdk'));
</script>
```

### IMA HTML5 SDK

- [Code Sample](https://github.com/googleads/googleads-ima-html5)

## Analytics and Metrics

For gathering intelligence about visitors and how they interact with the publisher's website.

## Google Analytics

> https://developers.google.com/analytics/devguides/collection/analyticsjs/

```html
<!-- Google Analytics -->
<script>
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','//www.google-analytics.com/analytics.js','ga');

ga('create', 'UA-XXXX-Y', 'auto');
ga('send', 'pageview');

</script>
<!-- End Google Analytics -->
```

## Mixpanel

> https://mixpanel.com/help/reference/tracking-an-event

```html
<!-- start Mixpanel --><script type="text/javascript">(function(f,b){if(!b.__SV){var a,e,i,g;window.mixpanel=b;b._i=[];b.init=function(a,e,d){function f(b,h){var a=h.split(".");2==a.length&&(b=b[a[0]],h=a[1]);b[h]=function(){b.push([h].concat(Array.prototype.slice.call(arguments,0)))}}var c=b;"undefined"!==typeof d?c=b[d]=[]:d="mixpanel";c.people=c.people||[];c.toString=function(b){var a="mixpanel";"mixpanel"!==d&&(a+="."+d);b||(a+=" (stub)");return a};c.people.toString=function(){return c.toString(1)+".people (stub)"};i="disable track track_pageview track_links track_forms register register_once alias unregister identify name_tag set_config people.set people.set_once people.increment people.append people.union people.track_charge people.clear_charges people.delete_user".split(" ");
for(g=0;g<i.length;g++)f(c,i[g]);b._i.push([a,e,d])};b.__SV=1.2;a=f.createElement("script");a.type="text/javascript";a.async=!0;a.src="undefined"!==typeof MIXPANEL_CUSTOM_LIB_URL?MIXPANEL_CUSTOM_LIB_URL:"//cdn.mxpnl.com/libs/mixpanel-2-latest.min.js";e=f.getElementsByTagName("script")[0];e.parentNode.insertBefore(a,e)}})(document,window.mixpanel||[]);
mixpanel.init("YOUR TOKEN");</script><!-- end Mixpanel -->
```

## Web service API wrappers

For developing client-side applications that communicate with external web services.

## Google Tag Manager

> https://developers.google.com/tag-manager/quickstart

```js
<!-- Google Tag Manager -->
<noscript><iframe src="//www.googletagmanager.com/ns.html?id=GTM-XXXX"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'//www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXX');</script>
<!-- End Google Tag Manager -->
```

## More

Some other [SDKs](https://github.com/vsouza/awesome-ios#sdk).

[Suggest one Now!](https://github.com/huei90/javascript-sdk-design/edit/master/SCOPE.md)
