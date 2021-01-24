---
layout: post
title: Using Network Interceptors with Javascript
---
![image](https://user-images.githubusercontent.com/25403969/105626412-65517380-5e55-11eb-89c8-d179310f5ce3.png)
More often than not, people approach error handling in calls using the approach we see above

While this works well for smaller applications with just a couple of calls, as your application scales, you have multiple network calls & each unexpected response to such call has to be dealt in a specified fashion.

Rewriting the handling code in the catch block introduces unnecessary code redundancy which makes it even more difficult to manage errors as your application scales.
In such a case, using a network interceptor should be the way to go. The fetch-intercept monkey patches the global fetch method and allows you the usage in Browser, Node and Webworker environments. In this demo, I’ll be using fetch-intercept as an example, but similar interceptors are available for axios too.
We start by installing our network interceptor, which can be done by using either

```cmd
npm install fetch-intercept — save
yarn add fetch-intercept
```

Next up, we create a file intercept.ts at the root of our application or under the directory where we introduce utilities,

```javascript
fetchIntercept.register({
  response: function (response) {
    if ([502, 503, 504].includes(response.status)) {
      history.push(SERVICE_UNAVAILABLE_URL);
    }

    if (response.status === 401) {
      //Perform actions for log out
    }

    return response;
  },
});
```

In this example, we are handling a 401 error by redirecting the user to a component which then dispatches the logout actions.
Quick sidenote — if you are using Reactjs, introducing react-redux bindings inside the interceptor may not be the best practice. Your store & interceptor should be separate entities.
Now, all we have to do is, load this file at a top level component, say index.ts. Once that’s done, all the network requests throughout the application pass through your network interceptor, hence you have one point of contact to handle every possible error.
You can learn more about fetch-intercept by checking <https://github.com/werk85/fetch-intercept>
