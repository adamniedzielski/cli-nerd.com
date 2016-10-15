---
layout: post
title: "Building the Case for Static Types in your React Applications"
date: 2016-10-15 21:00:00 +1100
---

*React.js* has grown quite popular over the last couple of months and thus it's no surprise that me and my colleagues
got to work on a range of projects built with some combination of tools like *React*, *Redux*, *Immutable*,
*Webpack* and *JSX*.

Most of our projects have relatively young code bases, only a couple of them are older than a year. It's those older
projects &ndash; the ones which have already gone through some changes in teams and requirements already &ndash; that I'd like
to talk about here.

The general opinion amongst me and my colleagues about *React* and the other tools mentioned previously has been
very positive. We enjoyed building applications with these tools and to us they are a big leap forward when it
comes to the development of single page apps.

I'm happy to see techniques of functional programming becoming more popular. Our web apps have been
more stable since we are building them with pure functions, immutability and atomic state management in mind.

However, most of the *React* projects that I have worked on were not (yet) using static types.

I don't want to get into the heated fundamental discussions about static vs dynamic types. Instead I'd like to
give just one particular example where types would be tremendously useful for your web application.


## A Seemingly Trivial Task

Getting a user interface right is hard and thus you'll usually want to quickly iterate on it in order to learn
and adjust it to your user's goal.

For that reason changing the interface of your application is a fairly common task. Ideally, such changes should be
quick to do and low in risk.

Here's an example: Let's say your task was to display a user's role in his profile. In a *React* code base without
types you would identify the correct component and read something similar to this:

    export function UserProfile({user}) {
      return <div>
        <p>{{user.fullName}}</p>
        <span className={{ user.isOnline ? "online" : "" }}></span>
      </div>;
    };

Immediately a question comes to your mind: is the user's role already part of the user object?

To answer that question, you'll start searching the entire code base to find out how user objects are created and
what attributes they have. The larger the code base, the more time it will take you to find the correct place.

Theoretically, you'd also have to check every piece of code that renders the `UserProfile` component only to then
begin the actual work: read all the code in between and check that the user object will actually
always carry a `role` attribute.

This would cost even more time and even worse: it's error prone &ndash; especially in a larger code base.

In reality we rarely do all this work and check all those places. Instead we'll check just one or maybe two. The
remaining pieces stay untested. I can see how often this actually causes problems just by looking into our frontend
exceptions &ndash; believe me, there's a lot of `TypeError` and `cannot access property x of null` exceptions.

Since uncaught exceptions immediately halt the execution of the current event from the event loop, those exceptions
can break a lot of things in your application.

For that reason a **trivial change can suddenly take a lot of time and be very risky**.


## A solution?

The first thing that comes into mind is *React*'s `PropType` feature, which allows you to specify the properties a
component requires. *React* will then check those properties at runtime and throw an exception if they don't conform
to the specification.

    UserProfile.propTypes = {
      user: React.PropTypes.shape({
        fullName: React.PropTypes.string.isRequired,
        isOnline: React.PropTypes.bool.isRequired,
        role: React.PropTypes.string.isRequired
      }).isRequired
    }

This helps to find out which properties the user object has, but it doesnt solve the real problem. You'll still
have to check every place where the component is rendered and verify that the correct properties are passed to it.

Luckily there is an actual way to solve this problem: use static types!

## A solution!

Here's the previous example rewritten in [Elm](http://elm-lang.org), my favourite language for the browser:

    type alias User =
      { fullName : String
      , isOnline : Boolean
      , role : String
      }

    userProfile : User -> Html
    userProfile user =
      div []
        [ p [] [ text user.fullName ]
        , span [ class (if user.isOnline then "online" else "") ] []
        ]

Sure, *Elm* has a different syntax that you need to get used to, but apart from that it doesnt look too different
from the `PropTypes` example.

Nonetheless, the `Elm` code is many times more helpful. Not only will your *IDE* immediately tell you which
properties a user has. Your compiler will also do **all the heavy work** for you and guarantee that each and
every user passed to this component has the correct attributes.

In fact, the `Elm` compiler allows you to only annotate a subset of types - it will automatically
infer the remaining types for you. Gone are the days of verbose Java annotations like `User user = new User()`.

And in case  the compiler finds a type error, it will [print a very friendly message](http://elm-lang.org/blog/compiler-errors-for-humans).

Guess how many type errors all participants of the first Elm conference had when counting them together: **0!**

## Conclusion

This simple example of a fairly common task shows how much value static types can add to your web application.

Static types help you to truly modularise your code and isolate pieces for easier and risk free change. **Thanks to
static types, trivial tasks can stay trivial**!

I highly recommend you to check out [Elm](http://elm-lang.org). If you're already using *React*, you will love it!

If your not keen to learn a new language, you can also use tools like *TypeScript* or *flow*. They solve the same problem,
but are similar to *JavaScript*.
