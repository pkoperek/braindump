---
layout: post
title: #403 Error CSRF token missing. Uploading a file with Lusca, Express.js and Angular.js
comments: true
---

TL;DR: When you try to upload a file to Express.js app from angular, use the `$http` service and don't try to push it with a `<form .../>` element.

I tried to start with the most obvious and simple way: just create a form, with file input element, create a simple backend endpoint in Express and combine them together. It turns out, that this approach doesn't work well when you integrate the lusca anti-CSRF library. Upon request call there is going to be a ugly message that the CSRF token wasn't found.

Funny enough: angular has [builtin support for XSRF protection](https://docs.angularjs.org/api/ng/service/$http). At least in theory the integration with lusca should be transparent. I started with checking the communication in dev console in chrome: the XSRF-TOKEN cookie was indeed being set in response messages but missing in requests. Then I modified the local copy of lusca to check when the token was actually being set. I noticed that the first call I was doing to the API was actually the file upload one. That meant that there was no chance the server was able to set up the token prior to the actual API call. This suggested that there is a problem on the client side. I've tried some weird approaches (e.g. with adding a hidden input field with `_csrf` name - a solution suggested [here](https://github.com/krakenjs/lusca/issues/54)), however nothing worked. 
Then the enlightenment came: the itegration was implemented in the `http` service and the form send wasn't using AJAX at all! I switched to sending the file through `http` service AJAX calls and ... it just started working! 
