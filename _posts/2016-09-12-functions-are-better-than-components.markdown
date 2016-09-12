---
layout: post
title: "Functions > Components"
date: 2016-09-12 21:00:00 +1100
---

I've introduced a fair amount of developers to the [Elm programming language](https://elm-lang.org)
at the last couple of [Elm meetups in Sydney](http://www.meetup.com/en-AU/Sydney-Elm-Meetup/). This has been a heaps
of fun, especially because total beginners make great progress in almost no time!

Then, last week, I introduced one of my team members to [React](https://facebook.github.io/react/), a JavaScript
library for building UIs (he's unfamiliar with frontend development). Walking him through the code I've
realised how complex React actually is – and how much explanation was necessary to get him to be productive.

I'd like to share my observations and explain how they let me to believe that functions are a much better way
of building UIs than React components are.


## A Quick Intro to Elm's Syntax

Before we can jump into actually building a UI in Elm, we need to learn the syntax for three basic elements of the
language:

1. Creating a list of the numbers 1 to 3

   ````elm
   numbers = [ 1, 2, 3 ]
   ````

2. Defining a function `add` that takes two arguments

   ````elm
   add a b = a + b
   ````
   *In JavaScript this would look like `function add(a, b) { return a + b }`*

3. Applying the `add function` to two numbers

   ````elm
   three = add 1 2
   ````
   *In JavaScript this would look like `three = add(1, 2)`*

Admittedly, getting used to this syntax without takes some time, especially when you're used to heaps
`(`, `)`, `{`, `}` and `,` used in languages like JavaScript. But I promise that those are the only new things
you'll have to learn.


## Building UIs in Elm

Let's immediately jump into a couple of examples (we're going to use the ingenious
[html](https://package.elm-lang.org/packages/elm-lang/html/latest) package).

1. This is how you show some text:

   ````elm
   view =
     text "hello world"
   ````
   *`text` is a function that takes a `String` and wraps it into a DOM node.*

1. We can make a title out of that:

   ````elm
   view =
     h1 [ class "nice-heading" ] [ text "hello world" ]
   ````
   *`h1` is a function that takes two arguments: a list of attributes and a list of child nodes.*


1. An unordered list with some links:

   ````elm
   view =
     ul []
       [ li [] [ a [ href="/" ] [ text "Home" ] ]
       , li [] [ a [ href="/about" ] [ text "About" ] ]
       , li [] [ a [ href="/contact" ] [ text "Contact" ] ]
       ]
   ````
   *`ul` has three `li` children, each with an `a` child and an `href` attribute.*

This is not only expressive, but also remarkably simple! The only concepts we have to understand are **functions** and
**values**. Every programmer has worked with those. There's nothing more to it. No magic. No other concepts.

In fact it is so simple that most beginners will immediately avoid duplication by refactoring this. Just pull out a new
function:

````elm
navigationItem href title =
  li [] [ a [ href=href ] [ text title ] ]
````

It's hard to overstate the incredible progress an absolute beginner makes during the first couple of hours of learning
Elm.

## Explaining React Components 1

How about React? First of all, React is a JavaScript library for building UIs. However, it's often used in combination
with JSX - a JavaScript syntax extension that looks somewhat like XML. Here are some examples:

1. This is how you show some text:

   ````jsx
   const view = "hello world"
   ````
   *We can directly use a string here.*

1. Again, we can make a title out of that:

   ````jsx
   const view = <h1 className="nice-heading">hello world</h1>
   ````
   *Here we're using the `h1` component, we'll get to what a component is in a second.*

1. And, the last example (a list with links):

   ````jsx
   const view =
     <ul>
       <li><a href="/">Home</a></li>
       <li><a href="/about">About</a></li>
       <li><a href="/contact">Contact</a></li>
     </ul>
   ````
   *This is nesting `<ul>`, `<li>` and `<a>` components.*


Ok, this almost looks like HTML. It has some subtle differences (e.g. the `class` attribute being replaced with
`className`). To be fair, the syntax inside of JavaScript code also takes a while getting used to. But it's
straightforward to explain. This is good!


## Explaining React Components 2

Now, how do we make our own components? That's where the trouble begins.

First of all, there are three ways of defining new components:

1. Using `React.createClass()`:

   ````jsx
   const NavigationItem = React.createClass({
     render() {
       return <li><a href={this.props.href}>{this.props.title}</a></li>;
     }
   });
   ````
   *Beginner question: "Where is the magical `this.props` coming from?"*

1. Using ES6 classes:

   ````jsx
   class NavigationItem extends React.Component {
     render() {
       return <li><a href={this.props.href}>{this.props.title}</a></li>;
     }
   }
   ````
   *Beginner thought: "I guess `this.props` is inherited from the parent class"*

1. Using functions:

   ````jsx
   function NavigationItem(props) {
     return <li><a href={props.href}>{props.title}</a></li>;
   }
   ````
   *Beginner thought: "`props` is just an argument, how do I pass them in?""*

We've just introduced three **different** ways of making components – all of them with more or less subtle differences.
Try explaining to a beginner when and why to use which of those.


## Explaining React Components 3

It feels as if we've just learned about two separate concepts. How do they fit together? My colleague had two particular
questions:

1. How can I use my components with JSX?

   ````jsx
   const view =
     <ul>
       <NavigationItem />
       <NavigationItem />
       <NavigationItem />
     </ul>
   ````
   *Once you defined your component, wrap it with `< >` to use it within JSX - the compiler will magically do this*

2. How do I pass those properties to my component?

   ````jsx
   <NavigationItem href="/" title="Home" />
   ````
   *All attributes are magically added to the `props` object you've seen before*

Ok, I think now we've covered the minimum fundamentals required to build something with React. Explaining all this to a
beginner is .. tough.


## Functions > Components?

Time to reflect on what we've covered. To build the same basic UI,

- in Elm, all we need is functions and values
- in React, we had to understand JSX, components, props and the different way of making components

What's remarkable here is that Elm is a language, whearas React is supposed to be just a library. One would expect
that learning a completely new language is harder and more complex than just using a library. However, from my
experience that's not the case here.


Don't get me wrong; I think React is a great tool and a huge step forward when compared to previous approaches to
JavaScript application development. However, since I've learned about Elm I doubt that it's the best tool available.

If this post got you interested in Elm, give it a try!

 - Checkout the [Elm website](http://elm-lang.org/), it lists many of the benefits
 - Play with the [Elm examples](http://elm-lang.org/examples)
 - Read the [Guide to Elm Development](http://guide.elm-lang.org/) for a walk through the most important topics


Last but not least, if you happen to be around Sydney, come and join the
[Elm Meetup in Sydney](http://www.meetup.com/en-AU/Sydney-Elm-Meetup/) (beginners welcome!!!)