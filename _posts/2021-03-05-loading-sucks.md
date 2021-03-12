---
layout: post
title: From the eyes of a junior developer - Adding Loading fields on Redux sucks and why you should use async mutex!
---
We have all run into this at some point or other, we wish to perform a whole bunch of tasks, and at the same time prevent some other tasks to happen until we have a response come back from server.

## The EASIEST way to tackle this - Add Loading field

Well, the easiest way to approach this would be to set a flag like "loading" to true in your reducer whenever you run an action which would later require some sort of a server response.

Here's the deal - this is technically the lowest hanging solution, and on the surface there is no such apparent draw back with this - but if you forget to manipulate this loading state at one place OR when a new developer joins the team and they add a new reducer and forget to make the necessary manipulations on this loading field, guess what happens? Your store keeps telling you that you are loading (if you throw a loader on the frontend based on this loading field then you'll keep seeing the spinner right in front of your eyes) when your content has seemingly loaded right!

Debugging this can be an absolute nightmare of a task, even more so when this loading state has not been updated correctly on multiple reducers staggered across stores!

Although, this may not always be a bad thing to do - as one of my collegues rightly pointed out while proof reading this "Using a loading field to control the loading spinner is not the problem itself… If we do need to show a spinner, we need to have a different data to represent this behavior… When we are loading some data into a specific field, some people just use null or undefined on this field to represent the loading state, and use an empty array to represent a not loaded state, but sometimes we need the empty array to represent the ‘no data’ state, or if it is some data that is loaded only after some user action, we need the null or undefined to represent the not-loading state before this user action, so in these cases, it does make sense to have the loading field state"

Furthermore, the loading field might not directly contribute to future problems, but it may be one of the factors, again as rightly described by Leonardo Carrerio,
"The forever loading state issue is not caused by using the loading field, I see two reasons:

1. Not handling exception cases. If we have an HTTP call to load the data, that can have all the code in the same function/scope using await, we just need to add a try-catch and put on the catch block the code to stop the loading state and show that an error has occurred. But on websockets that we expects a response message from server, then it’s harder, because we may send a message and not get a response at all. So we need to handle that in some way. It can be a timeout, but it’s hard to handle, because we never know if the problem is the delay (the message can arrive after our timeout). Or if it will never arrive. We can handle that by adding options to the user, for example: “Taking too long? cancel or try again“.

2. Using the loading field to control the logic. The logic cannot depend on this loading field. The logic changes, and then the use of this loading will change and is very likely that we will forget to handle some edge case that will make this field to have the wrong value."

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
