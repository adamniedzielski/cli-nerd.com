---
layout: post
title: "How To Crush Unit Test Procrastination"
date: 2015-09-01 22:00:00 +1000
---

Recently I reviewed a couple of changes to a legacy code base. Neither the new nor the old code were tested. Nevertheless, I asked the author to write some unit tests for the new code. He disagreed.

> “Look, I know that the project has a quality problem. But instead of writing some tests for the new stuff now, lets write all the tests for the core logic later. The test coverage needs to reflect the important pieces of the code base.”


### Did You Say Procrastination?

Pheeeeww! This developer seems to be taking tests quite seriously. He definitely knows about the value of a strong test suite. And he’s not alone: the rest of the team does too! Awesome! But how is it then, that no tests exist?

I have an assumption. Did you ever face a task so large that you didn’t know how to start? A task that was probably too important to screw it up? If so, then it’s likely that you deferred it for as long as possible. This is known as **procrastination**.

Most of these tasks have a definite deadline. As developers that often applies to new features or bug fixes. Someone will ask questions if we don’t deliver.

However, this is not quite true for unit tests. These are the responsibility of us, the developers. No one else will notice their lack. We can simply defer them again and again. As a consequence, no one will write any tests! Do you see the problem?


### A Simple Cure

Luckily there’s an easy way to stop procrastinating: Chop the large task into many small pieces and tackle them one by one. Here are some guidelines to get started in a legacy code base:

- If you work in a code base with zero tests, chances are that you need to configure a test runner first. In this case I usually start with a smoke test, e.g. by requiring a class and making sure that I can instantiate it.

- If someone already configured the test runner, you could set up continuous integration for this project

- If you have problems understanding what a piece of code does, write some tests to understand the behaviour. Explore the code first and then document your results with tests.

- If you fix a bug, try to come up with a regression test for it.

- If you develop new code, write at least a test or two for the most critical parts in it.

Following this guidelines, you obviously won’t get a strong test suite immediately, but that’s ok. The point here is to get started! Make this a habit for all of your coding and you’ll soon have many projects with decent test coverage.

### Crushing It

Next time you find yourself modifying a legacy code base, even if it is just a couple of lines, contribute at least a single unit test!