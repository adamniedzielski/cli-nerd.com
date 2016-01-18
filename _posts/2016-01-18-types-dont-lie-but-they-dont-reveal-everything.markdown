---
layout: post
title: "Types Don't Lie, But They Won't Reveal Everything"
date: 2016-01-18 21:00:00 +1100
---

When reimplementing the monad functions `>>=`, `>>`, `return` and `fail` for `Maybe` in Haskell, I realized that I didn't know how `>>` exactly worked.

The type signature for `>>` is:

    (>>) :: Monad m => m a -> m b -> m b


Here's my initial implementation:

    _ >> b = b


While testing my implementation, I discovered some odd behaviour. These examples return the correct results:

    Just 3 >> Just 5 = Just 5
    Just 3 >> Nothing = Nothing
    Nothing >> Nothing = Nothing


But the next one returns an incorrect one (a correct implementation would yield `Nothing`):

    Nothing >> Just 3 = Just 3


Oops! My implementation completely ignored the monadic context. The correct implementation for `>>` is:

    a >> b = a >>= \_ -> b


Looks slightly more complex, but it's really nothing more than invoking `>>=` and discarding its result.

A small mistake, but a great example that you can't necessarily guess a function's implementation just from its type signature.