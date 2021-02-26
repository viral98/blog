---
layout: post
title: Know how to set your boundaries.....in React!
---
![wsod](https://user-images.githubusercontent.com/25403969/108991398-0f732400-76be-11eb-9e01-ddc5b0ae0393.png)
We all have heard about the dreaded blue screen of death on windows but there's a new "white screen of death" in town introduced in React 16, where now, if you run across an unhandled error all your customer sees... is a white screen!
What throws developers off guard is the fact that on development builds we are shown the error so we just try to work towards fixing but for any off-case which we didnt handle, react would not keep the last rendered state of the app! So even if one child component is misbehaving your entire app goes white!

I'll be using <https://www.npmjs.com/package/react-error-boundary> for setting our boundaries!

Our first order of business is encapsulating our app with a component which we show in case we run into errors... think of it like a catch block but for your components!

```javascript
<ErrorBoundary FallbackComponent={ErrorBoundaryWithFallback}>
        <App />
</ErrorBoundary>
```

Now anything that goes south in app, will be handled by this new component of yours call "ErroBoundaryWithFallBack"
There is one thing to remember though, this will just handle errors in your compoments, anything going south inside your action creators will still result in a WSOD (White screen of death)

Once this is done, open up your ErrorBoundaryWithFallback function and add a screen to be displayed to the user in case they run into this...
Bonus points - have different boundaries for different components of your dashboard! So now, you can display different error pages based on the component that is misbehaving and perhaps offer a comprehensive fallback instead of plain and simple "refresh".

But there's still one unanswered question - how do we have a blanket catch for other errors? Stay tuned, I'll be covering it in upcoming blogs!
