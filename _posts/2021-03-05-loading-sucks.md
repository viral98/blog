---
layout: post
title: From the eyes of a junior developer - Adding Loading fields on Redux sucks and why you should use async mutex!
---
We have all run into this at some point or other, we wish to perform a whole bunch of tasks, and at the same time prevent some other tasks to happen until we have a response come back from server.

## The EASIEST way to tackle this - Add Loading field

Well, the easiest way to approach this would be to set a flag like "loading" to true in your reducer whenever you run an action which would later require some sort of a server response.

Here's the deal - this is technically the lowest hanging solution, and on the surface there is no such apparent draw back with this - but if you forget to manipulate this loading state at one place OR when a new developer joins the team and they add a new reducer and forget to make the necessary manipulations on this loading field, guess what happens? Your store keeps telling you that you are loading (if you throw a loader on the frontend based on this loading field then you'll keep seeing the spinner right in front of your eyes) when your content has seemingly loaded right!

Debugging this can be an absolute nightmare of a task, even more so when this loading state has not been updated correctly on multiple reducers staggered across stores!

## A not so easy way to tackle this - P-Queue

A P-Queue here refers to a queue which returns a promise hence P-queue (promise-queue)

Lets get back to our original example, we have a bunch of redux actions which we dont wish to run all at the same time... now, we can use the async-await syntax and load functions on top of p-queue and as they return a promise upon completion, we can "Await" for them to be done!

On top of this you can also set the level of concurrency, so a concurrency of 1 would be strictly all actions execute in a strictly linear fashion with no parallel processing happening whatsoever.

Another thing to note here is, p-queue considers failed tasks to be "run" too, and returns a promise for them, while its not relevant for the use case I am referring to here, it might be helpful to keep in mind!

```javascript
yarn add p-queue
OR
npm install p-queue
```

Setup is as simple as it gets!

```javascript
export function getNewQueueRunner() {
  const queue = new PQueue({ concurrency: 1 });
  return async (description: string, task: () => Promise<void>) => {
    await queue.add(async () => {
      //Task started to run
      await task();
     //Task finished Running
    });
  };
}
```

Now in our code we use this function

```javascript
const runOnQueue = getNewQueueRunner();
.
.
.
.

export function someAction(): AppThunk {
  return async (dispatch, getState) => {
    await runOnQueue( async () => {
        //Code
    })
  }
}

export function someOtherAction(): AppThunk {
  return async (dispatch, getState) => {
    await runOnQueue( async () => {
        //Code
    })
  }
}
```

Now, upon dispatching _someAction_ and then dispatching _someOtherAction_ would mean _someOtherAction_ would only run after _someAction_ finishes off execution.
And that's it! We don't need to rely on a loading flag + now new changes wouldn't affect previously functioning systems.
