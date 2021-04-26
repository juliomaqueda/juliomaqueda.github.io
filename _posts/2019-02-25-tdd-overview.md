---
layout: post
title: Test Driven Development overview
description: Test Driven Development overview
date:  2019-02-25
time: 12 mins
tags: tdd testing
language: english
---

Internal session I gave to the SWAT team at [Ixxus](https://www.linkedin.com/company/ixxus/) on the use of TDD and its benefits. Fun journey from the common development cycle to a more tests-aware approach.

<!-- more -->

## What's wrong with our tests?

Nothing better than starting with a tale...

### The chef's best stew ever analogy

Some of the reasons that move people to include TDD as part of their development mindset can be fully explained with the short story of a chef who, a day of divine inspiration, prepared the best stew ever seen mixing ingredients willy-nilly. At the very first instant he tasted that incredible flavour, he wanted to note down all metrics about the ingredients he used, so he could replicate the same formula in further stews.

However, he realised he had just some bare ideas about what he used, so he needed to open the pot and start digging and pulling things out. Some pieces were clearly identifiable, some others required a bit of effort, and the remains were practically impossible to classify.

<div class="centered"><img src="/assets/tdd-overview/stew-and-chef.png" width="200"/></div>

## Revisiting the old fashioned coding path

If we had to picture the common development cycle, we'd probably come up with something similar to the diagram below:

<div class="centered"><img src="/assets/tdd-overview/old-fashioned-coding-path.png"/></div>

Where we can identify clear place to improve:

* Coding from the design phase usually requires time to discover all possible scenarios.
* There are no leads that guarantee the solution is following an MVP approach.
* Tests should be clear and concise. Tons of hours are wasted trying to make tests pass.
* Code shouldn't be refactored just to allow better access from tests.
* Having to figure out what your code does during the test phase can become a nightmare.

## TDD - Tests first, then code

Let's start by answering the most typical questions about TDD...

#### What is TDD?

TDD `is` primary a design technique, `not` a testing technique. Tests first, then code.

#### Why TDD?

> Because it is the simplest way to achieve both good quality code and good test coverage.
>
> -Kent Beck-

#### Who can use TDD?

Every single developer.

#### How do I apply TDD?

You're lucky because there are just three basic laws to comply:

* Don't write any production code until you have written a failing test. You can use your design, class diagrams, and so on to help you naming the objects properly.

* Don't write more than one test that is sufficient to fail. The idea is having tests covering the required features and behaviors one by one, so there is no point in having multiple tests failing.

* Write only the minimal amount of production code to pass the currently failing test.

## Understanding TDD - Fake it until you make it

### Red - Green - Refactor cycle

<div class="centered"><img src="/assets/tdd-overview/red-green-refactor-cycle.png" width="300"/></div>

`Red`: you always need to start by creating a short failing test. It's recommended not to create the tested class before this step, so this first test will probably fail due to compilation errors. In the red phase you act like a demanding user who wants to use the code that's about to be written in the simplest possible way. You have to write a test that uses a piece of code **as if it were already implemented**. You shouldn't think about how you are going to write the production code at this point. This is the phase where **you design how your code will be used by clients**.

`Green`: create the minimal amount of production code to make the test pass. Enjoy the exciting moment you get seeing a test becoming green. Your mind will periodically get addicted to green.

`Refactor`: review the solution, clean your code, extract methods, and remove duplicate code. Avoid refactoring beforehand as the chief goal is having the test passing.

> Make it green, then make it clean

## New coding path paradigm

<div class="centered"><img src="/assets/tdd-overview/proposed-coding-path.png"/></div>

### Benefits

* Quick feedback.
* Higher quality.
* Cleaner code.
* It lets you know when you're finished.
* The hard stuff and surprises are tackled later on.
* Provides concrete evidences that your code works.
* Failing code is usually narrowed to most recent changes.
* Simplicity.
* Coverage.
* Well-designed applications.
* Fast debugging.
* It validates your design.
* Requires discipline.

### Proposed best practices

* Before you start, make a TODO list.
* Write operations down. List the null tests, exceptions, and some others.
* Each test should test a single concept.
* Limit the scope of your test to the object under test.
* Follow the Given (preconditions) - When (production code calls) - Then (check the results) blueprint.
* Strive for as few assertions per test as possible (ideally only one).
* Make small and incremental changes continually.
* Readability matters a lot in tests, add comments when needed.
* When you get stuck on a test, try starting with the assertion(s) and then work on the setup.
* Note down refactoring that come to your mind while you are writing production code.
* Mocks preferred over static data.
* Think of what the object does, not what the object is.
* Treat your tests as production code, they must be maintained and refactored.
