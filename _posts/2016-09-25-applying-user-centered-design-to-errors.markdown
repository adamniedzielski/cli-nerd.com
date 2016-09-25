---
layout: post
title: "Applying User-Centered Design to Errors"
date: 2016-09-25 22:00:00 +1100
---

I've spent a significant amount of my career investigating, handling and raising all
sorts of errors – be it logical and fencepost errors, disk and network failures or compiler errors and runtime
exceptions. Some are more likely to occur, others might have a bigger impact. They all have one thing in common
though: Someone will need to deal with them.


## Well-Designed Errors Yield Better Programs

Before I've got into functional programming, I've never really thought about errors in general. Sure, I would
add some *null checks* here and there or maybe wrap a piece of code with a *try .. catch* block. But I didn't
really think about underlying concepts, principles and techniques of handling errors.

However, *Haskell* and *Elm* changed that for two reasons.

1. *Haskell* has made me familiar with ideas like `Maybe`, `Either` and `IO`.
1. *Elm* has shown me what a [well-designed compiler error](http://elm-lang.org/blog/compiler-errors-for-humans)
   should look like:


   ````
   The type annotation for `testUser` does not match its definition.

   6| testUser : User
                 ^^^^
   The type annotation is saying:

       { ..., firstName : String }

   But I am inferring that the definition has this type:

       { ..., firstName : Maybe a }
   ````


Thanks to those languages, I'm now convinced that the way we deal with errors has a big impact on the quality of
our programs. This implies that we can improve the quality of our programs by improving the quality of the errors
that we raise and expose.

To have our own errors match the quality of the *Elm* compiler, we need to focus on our users when designing them.

## Qualities of Well-Designed Errors

As a user I generally don't want to worry about errors at all. When we acknowledge and embrace this, we can derive
two important guidelines:

1. Before exposing an error to a user think really hard about it. Can you handle the error in your own program,
   without exposing it to the user? While this is preferred, it only applies to a small subset of errors. A hidden
   error can lead to pretty frustrating debugging sessions. If in doubt, expose the error!

1. Do everything you can to help your users handle the error correctly. This might seem obvious to you, but the
   majority of errors I've encountered look as if no one ever considered this.

When you're trying to follow these two guidelines for the first time, you'll notice that this is much harder than
it seems. Well-designed errors actually require some thought and effort.

Here's a couple of ideas to help you get started:

- **Be explicit** about things that can go wrong. Make potential errors part of the API. Return types like `Result`,
  `Either` or `Promise` instead of just throwing exceptions. Those types are great because they force your users
  to handle the error case.
- **Fail early** if an error occurs. You don't want to carry around the error and blow up much later. It should be
  easy to figure out where an error is coming from.
- **Provide relevant information** with every error. Don't just say that reading a file failed, but also mention
  which file couldn't be read and why the read failed.
- **Stay within the abstraction**. When writing to say a database, the database library should not directly raise
  a file or network error. Instead it should return a "write-file error" that contains the actual reason. Use domain
  terminology.
- **Group related errors** into the same namespace / module / file. This makes it easier to get an overview of what
  else could go wrong.
- **Utilise the type system** if you can. Ideally the types tell me what kind of errors can occur for each operation.
- **Recommend solutions** in your documentation. Let your users know what they need to do to prevent this error from
  happening again.


I hope that this list of guidelines helps you when designing errors for your next program.