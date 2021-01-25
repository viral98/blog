---
layout: post
title: Stop using window.remote() now!
---
![image](https://user-images.githubusercontent.com/25403969/105672660-cdae5c80-5f0a-11eb-97cd-32ece63c1ab6.png)

As of now, if you start looking up code to act as a boilerplate for an application that uses electron + any other JS Framework, you'd be bombarded with preload.js files attaching the remote module on the window object and then these projects use this remote module over on their frontend framework

## But, why?

Because it's convinient! You see, remote provides access to pretty much most of the APIs your application would need access to throughout its life time. As a result simply calling them like window.remote.SOME_API() is much much easier than the alternative!

## Why is remote bad, then?

### Argument 1 (And probably the most important one, in my opinion) - Main process cannot block waiting for renderer to return result

The alternative to remote, i.e. ipc calls, allow us to call them via .handle() and .invoke() both of which allow us to use async-await strategies which is a key feature in a lot of these remote API usage! Think about scenarios where you need results before proceeding further! Qouting from another great article on this issue "When you pass a function as a callback to a remote method, then calling that callback from the main process will always return undefined" <https://medium.com/@nornagon/electrons-remote-module-considered-harmful-70d69500f31#d978>

### Argument 2 - Remote Objects !== Regular Objects

We can think of these remote objects which we have overloaded on window as proxies to the underlying APIs. Remote in this case tries to abstract the underlying APIs but this abstraction, as one can imagine isnt fool-proof and leads to edge case breakages which are about impossible to replicate -> resolve.

#### Counter Point to Argument 2 - Setup Event listners for call backs to avoid race conditions

Quoting from this github comment <https://github.com/electron/electron/issues/21408#issuecomment-589463204> - "Setting up callback listeners before execution eliminates this class of problems. I attended the Covalence 2020 Conference where Jeremy gave a show and tell of this. It was funny, but I was not impressed. Such a race condition will not happen with proper sequencing, setting up callback listeners before issuing the call."

This is a good counter point - however, it doesn't help when your app relies heavily on ipc calls - you now need to make sure you setup multiple event listeners waiting for callbacks + write clean up code to kill those listeners in case things go south and you don't get a response! That is just more code which you need write, test and maintain!

### Arugment 3 - Performance

Taking off from point 2, as remote essentially acts as a proxy for underlying APIs, this whole routing takes some toll. How big of a toll? About x10,000 slower than actually accessing those objects via IPC! How big of a difference is this in terms of perception? 0, atleast in my case. We shifted our whole app to use IPC only, and the difference isn't that huge, in terms of performance atleast.

## Conclusion

![image](https://user-images.githubusercontent.com/25403969/105778382-56bca680-5f92-11eb-9e80-63af8b45c16b.png)

Use IPC!! No, seriously - in an ideal world the communication between your main and renderer should be minimised, but we realise that that's difficult to do when building apps that do infact rely on those underlying APIs, in such a case, using IPC would be a great way to counter this issue. Sure, its more work than using window.remote, but its much much safer and to do so and sooner or later, remote is going to be deprecated, its better to spend a few days to move infra away from remote and to IPC!
Specifically, ipc.invoke would be the best choice if you are running an electron version above 7 as it would allow the use of async await, hence circumventing the unexpected issues caused by the remote module at times + remove the need for adding listeners, hence making your code cleaner, infact!
