---
layout: post
title: "7 Reasons to Upgrade to Node v4 Now"
date: 2015-09-09 22:00:00 +1000
---

Today [Node v4 got released](https://nodejs.org/en/blog/release/v4.0.0/). It is the first stable release after the
io.js merge and thus brings us a bunch of shiny, new ES6 language additions. While there's already a great
[overview of the ES6 additions](https://nodejs.org/en/docs/es6/), this post shows how to make use of them!

In particular, I'm going to address the following additions:


1. [Template Strings](#template-strings)
1. [Classes](#classes)
1. [Arrow Functions](#arrow-functions)
1. [Object Literals](#object-literals)
1. [Promises](#promises)
1. [String Methods](#string-methods)
1. [let and const](#let-and-const)


## 1. Template Strings

If you ever tried to create a multiline string in JavaScript, you probably ended up doing something similar to this:

{% highlight javascript %}
var message = [
    'The quick brown fox',
    'jumps over',
    'the lazy dog'
].join('\n');
{% endhighlight javascript %}


While this works for a small amount of text, it gets messy for a couple of sentences. Therefore, a clever developer came
up with a hack called [multiline](https://github.com/sindresorhus/multiline):

{% highlight javascript %}
var multiline = require('multiline');
var message = multiline(function () {/*
    The quick brown fox
    jumps over
    the lazy dog
*/});
{% endhighlight javascript %}


Luckily, ES6 brings us [template strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings):

{% highlight javascript %}
var message = `
    The quick brown fox
        jumps over
        the lazy dog
`;
{% endhighlight javascript %}


In addition, they also bring us string interpolation:

{% highlight javascript %}

var name = 'Schroedinger';

// stop doing this ...
var message = 'Hello ' + name + ', how is your cat?';
var message = ['Hello ', name, ', how is your cat?'].join('');
var message = require('util').format('Hello %s, how is your cat?', name);

// and instead do that ...
var message = `Hello ${name}, how is your cat?`;
{% endhighlight javascript %}

[Check out the details about template strings on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings).


## 2. Classes

Defining classes in ES5 looks somewhat strange and definitely takes time to get used to:

{% highlight javascript %}
var Pet = function (name) {
    this._name = name;
};

Pet.prototype.sayHello = function () {
    console.log('*scratch*');
};

Object.defineProperty(Pet.prototype, 'name', {
  get: function () {
    return this._name;
  }
});


var Cat = function (name) {
    Pet.call(this, name);
};

require('util').inherits(Cat, Pet);

Cat.prototype.sayHello = function () {
    Pet.prototype.sayHello.call(this);
    console.log('miaaaauw');
};
{% endhighlight javascript %}


Luckily we can now use the new ES6 syntax in Node:

{% highlight javascript %}
class Pet {
    constructor(name) {
        this._name = name;
    }
    sayHello() {
        console.log('*scratch*');
    }
    get name() {
        return this._name;
    }
}

class Cat extends Pet {
    constructor(name) {
        super(name);
    }
    sayHello() {
        super.sayHello();
        console.log('miaaaauw');
    }
}
{% endhighlight javascript %}

An extends keyword, constructors, calls to the super class and properties. How awesome? But there's more.
[Check out the comprehensive guide on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)


## 3. Arrow Functions

The dynamic binding of `this` on function invocation always causes some confusion, and people worked around it in a
couple of ways:

{% highlight javascript %}
Cat.prototype.notifyListeners = function () {
    var self = this;
    this._listeners.forEach(function (listener) {
        self.notifyListener(listener);
    });
};
{% endhighlight javascript %}
{% highlight javascript %}
Cat.prototype.notifyListeners = function () {
    this._listeners.forEach(function (listener) {
        this.notifyListener(listener);
    }.bind(this));
};

{% endhighlight javascript %}


Now you can just use fat arrow functions:

{% highlight javascript %}
Cat.prototype.notifyListeners = function () {
    this._listeners.forEach((listener) => {
        this.notifyListener(listener);
    });
};
{% endhighlight javascript %}

[Check out arrow functions in more detail](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).


## 4. Object Literals

When using object literals, you can now use a nice little shortcut:

{% highlight javascript %}
var age = 10, name = 'Petsy', size = 32;

// instead of this ...
var cat = {
    age: age,
    name: name,
    size: size
};

// ... do this ...
var cat = {
    age,
    name,
    size
};
{% endhighlight javascript %}

Additionally, you can now easily [add functions to your object literals](https://github.com/lukehoban/es6features#enhanced-object-literals).


## 5. Promises

Instead of depending on third party libraries like `bluebird` or `Q`, you can now use [native promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). Those expose the
following apis:

{% highlight javascript %}
var p1 = new Promise(function (resolve, reject) {});
var p2 = Promise.resolve(20);
var p3 = Promise.reject(new Error());
var p4 = Promise.all(p1, p2);
var p5 = Promise.race(p1, p2);

// and obviously
p1.then(() => {}).catch(() => {});
{% endhighlight javascript %}


## 6. String Methods

We got a couple of new string utility functions too:

{% highlight javascript %}
// replace `indexOf()` in a number of cases
name.startsWith('a')
name.endsWith('c');
name.includes('b');

// repeat the string three times
name.repeat(3);
{% endhighlight javascript %}

Go and tell those ruby kids! Strings also [support better unicode handling now](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla#Additions_to_the_String_object).


## 7. let and const

Guess the return value of the following function call:

{% highlight javascript %}
var x = 20;
(function () {
    if (x === 20) {
        var x = 30;
    }
    return x;
}()); // -> undefined
{% endhighlight javascript %}

Yep, `undefined`. Replace `var` with `let` and you get the expected behaviour:

{% highlight javascript %}
let x = 20;
(function () {
    if (x === 20) {
        let x = 30;
    }
    return x;
}()); // -> 20
{% endhighlight javascript %}

The reason: `var` is function-scoped, while `let` is block-scoped (which is what most people expect). Because of that,
it is save to say that `let` is the new `var`. You can get [more details about let on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let).

Bonus: Node now [supports the `const` keyword](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const), which prevents you from reassigning a different value to the same reference:

{% highlight javascript %}
var MY_CONST = 42; // no, no
const MY_CONST = 42; // yes, yes

MY_CONST = 10 // with const, this is no longer possible
{% endhighlight javascript %}


## Wrapping Up

Node v4 brings even more ES6 additions, but I hope that these seven examples already convinced you to update and use the
latest version.

There are many more language features (e.g. maps/sets, symbols and generators, just to mention a few more). Make
sure that you also check the [overview of ES6 additions to Node v4](https://nodejs.org/en/docs/es6/). Happy updating!
