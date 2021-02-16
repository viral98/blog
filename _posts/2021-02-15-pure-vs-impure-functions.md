---
layout: post
title: Pure Functions!
---

![ipc](https://user-images.githubusercontent.com/25403969/107893419-188f2300-6f51-11eb-8ea1-0bca335ac19f.png)

## What is a "Pure" function

*Short answer* - A function that will give you a predictable output for a fixed input regardless of what’s going on with the rest of the application + this function will not change any data within the app either.

*Long answer* - Pure functions are a way to ensure peace of mind, in a sense. If you call a pure function, you truly know that the output is independent of the application state and hence can focus on purely debugging that function rather than worrying about implications that the application state can have on the function.

Implication(s) the function can have on the application state.
Notice point 2, this is another feature of such “pure” functions, they must not alter the application’s state OR to put it more formally, they do not have side effects.

1. The application state can have on the function.
2. Implication(s) the function can have on the application state.

Notice point 2, this is another feature of such "pure" functions, they must not alter application's state OR to put it more formally, they do not have side effects.

## What are "side-effects"

Side effects in this context are similar to the side effects we have when we take medicines! We all wish that medicine just does the job it's intended to do and doesn't mess around with anything else in our body, but most likely, it'll have unintended side effects that are printed on the label as warnings. Similarly, we want our functions to just "work" and not alter anything in the system OR have behaviour that is dependent on certain variable external to the function (eg, return a random value, have some operation that relies on the exact time, etc)

## Welcome back to reality

Just like medicines, it's unrealistic to expect all functions to be pure - in fact we want certain functions to behave in a manner dictated by the system state. However, it's best to actually decouple logic that can be considered pure into a separate function.
This way, we have one less thing to worry about while we debug some intermittent issue plus we can now re-use this at other places with the peace of mind that the function will just do what it's intended to do and nothing else.
