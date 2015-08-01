---
layout: post
title: "$provide.decorator()"
date: 2014-05-08 09:41:19 +0200
comments: true
categories: angular
permalink: /blog/2014/05/08/$provide-dot-decorator.html
---

## Modifying the behavior of AngularJS' core services

After using [Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk), I wondered how they are able to collect performance metrics at this level of detail. Thoughts full of horrible code rushed to my mind. I opened the source code in fear, always ready to close it as fast as possible. After some reading, I discovered [these lines](https://github.com/angular/angularjs-batarang/blob/master/js/inject/debug.js#L712):

{% highlight javascript %}
$provide.decorator('$rootScope', function ($delegate) {

    // ...

    $delegate.__proto__.$watch = function (watchExpression, applyFunction) {
    	// ...
    };

    // ...

    return $delegate;
});
{% endhighlight %}

Where once was fear there is now only awesome! I just discovered another well-architected feature of AngularJS, which &ndash; unfortunately &ndash; isn't well documented.


## Meet the decorator pattern

In software design, a decorator is **an object that modifies the behaviour of another, already existing object without touching its code**. This is achieved by wrapping either the whole object or just parts of it and then using the wrapped object instead of the original one.

![Conceptual view of the decorator pattern]({{ site.baseurl }}/images/angular-decorate.png)


## Decorators in Angular

Let's create a basic AngularJS decorator to modify the behavior of the popular [ngResource](https://docs.angularjs.org/api/ngResource/service/$resource) service:

{% highlight javascript %}
var API_PREFIX = '/api/v1';

angular.module('ngResource+apiPrefix', [
    'ngResource'
]).config(function ($provide) {
	$provide.decorator('$resource', function ($delegate) {
	
		// Return a new constructor function that prepends an API
		// prefix to the passed resource URL (first parameter).
    	return function decoratedResource() {
        	arguments[0] = API_PREFIX + arguments[0];
	        return $delegate.apply(this, arguments);
    	};
    
	});
});
{% endhighlight %}

This code will prepend `'/api/v1'` to all your `$resource` REST urls. Therefore, when you define new models with `$resource('/model')`, they will use `'/api/v1/model'` instead.

To accomplish that, we create a new module named `ngResource+apiPrefix` that depends on `ngResource`. We call `$provide.decorate()` and pass in the name of the service we want to decorate, along with a function whose parameters are going to be injected.

The `$delegate` argument is resolved with an instance of the specified service. We now replace the original `$resource`  function with a new one that does three things:

1. Prepend a common `API_PREFIX` to the resource url.
2. Invoke the original `$resource` function with the modified arguments
3. Return the original object

In order to load this decorator, we would have to load the module:

{% highlight javascript %}
var app = angular.module('app', ['ngResource+apiPrefix']);
{% endhighlight %}

From now on, all instances of `$resource` are decorated. There's no way to access the original `$resource` service again.


## Syntactic Sugar

The snippet above is a little verbose, because it has some boilerplate code in it. We can get rid of that by using the [angular-decorate](https://github.com/damienklinnert/angular-decorate) plugin.


{% highlight javascript %}
var API_PREFIX = '/api/v1';

angular.module('ngResource+apiPrefix', [
    'ngResource'
]).decorate('$resource', function ($delegate) {
	
    return function decoratedResource() {
        arguments[0] = API_PREFIX + arguments[0];
        return $delegate.apply(this, arguments);
    };
    
});
{% endhighlight %}

Simple, isn't it?

*By the way, the plus sign in the module name is a convention that orginates in ObjectiveC and helps to identify the intention of a decorator.*


## Related Material

 - [$provide.decorate() documentation](https://docs.angularjs.org/api/auto/object/$provide#decorator)
 - [angular-decorate module on github](https://github.com/damienklinnert/angular-decorate)
 - [Hacking Core Directives in AngularJS](http://briantford.com/blog/angular-hacking-core.html?utm_source=javascriptweekly&utm_medium=email)
 - [AngularJS $provide.decorate() slides](https://www.slideshare.net/damienklinnert/angular-decorate)

 
<iframe src="http://www.slideshare.net/slideshow/embed_code/33332055" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/damienklinnert/angular-decorate" title="Angular decorate" target="_blank">Angular decorate</a> </strong> from <strong><a href="http://www.slideshare.net/damienklinnert" target="_blank">Damien Klinnert</a></strong> </div>
