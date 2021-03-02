---
layout: post
title: From the eyes of a junior developer - Adding Loading fields on Redux sucks and why you should use async mutex!
---
We have all run into this at some point or other, we wish to perform a whole bunch of tasks, and at the same time prevent some other tasks to happen until we have a response come back from server.

## The EASIEST way to tackle this - Add Loading field

Well, the easiest way to approach this would be to set a flag like "loading" to true in your reducer whenever you run an action which would later require some sort of a server response.

Here's the deal - this is technically the lowest hanging solution, and on the surface there is no such apparent draw back with this - but if you forget to manipulate this loading state at one place OR when a new developer joins the team and they add a new reducer and forget to make the necessary manipulations on this loading field, guess what happens? Your store keeps telling you that you are loading (if you throw a loader on the frontend based on this loading field then you'll keep seeing the spinner right in front of your eyes) when your content has seemingly loaded right!

Debugging this can be an absolute nightmare of a task, even more so when this loading state has not been updated correctly on multiple reducers staggered across stores!

## A not so easy way to tackle this - Async Mutex

Async mutex in essence achieves the same task which you were trying to achieve from this loading state, i.e. wait for something to happen and then once its done then allow/perform subsequent actions.

The only drawback here would be some extra code which you'd have write as opposed to adding a single field on your store, but as you will see, there are clear upsides to using this as opposed to the loading field.

Mutex is something which most CS undergrads might have heard but here's a short recap -
"mutex" usually refers to a data structure used to synchronize concurrent processes running on different threads.  This is guaranteed to block the thread until no other thread holds a lock on the mutex and thus enforces exclusive access to the resource.

While we are recapping, lets quickly brush up on semaphores too - A semaphore while being similar to a mutex has this exception of allowing multiple threads to acquire it at a given time, this can come in handy when you want to run a specified number of tasks in parallel.

As with most libraries, we start off with

```javascript
yarn add async-mutex
OR
npm install async-mutex
```
