---
layout: post
title: From the eyes of a junior developer - Electron IPC!
---
![ipc](https://user-images.githubusercontent.com/25403969/107105148-da0bb100-684a-11eb-9e39-9161d6f36271.png)

IPC (Inter Process Communication) is something most of us have heard about in undergrad but it was all in the context of fancy OS level calls, something which we all understood from a theoretical perspective but pretty much never came across a scenario where we truly had to work with it.

If you were an undergrad in anytime after late 2014-early 2015, you must have heard how "easy" it is to port any web app into a desktop app, almost making it sound as if it is a piece of cake.
Well, that statement is true but also decietful.

You see, after the inital setup, you can technically have a desktop app for your website in just one line of code -

```javascript
  await this.appWindow.loadURL(YOUR_WEBSITE_URL)
```

However, if your desktop app is just a website being rendered inside chromium, it gives little incentive to the user to actually use it over running it via the browser!
Most firms would like to add certain nice to haves which arent accessible via the browser. Now, electron does offer these "nice to haves" but how would your website access them?
Think about your website like a driver and your goal is to turn the wheels of your car. Now you can't directly go out of the car and turn the wheels manually, you need some interface to do this for you - that's what electron does - it acts as the steering wheel, so the driver (your website) accesses the steering wheel, and the steering wheel then moves the wheels in the direction of your choice (wheels === actual functionality on the OS).

Now, your website technically is still sitting inside a container, inside electron. In order to signal things to electron, it uses IPC (inter process communication) where it notifies electron "hey,move the wheel to the right by 10 degrees" and electron keeps listening for the any wheel turning inputs, the moment it recieves such a message, it turns the wheels (or makes underlying OS calls).

Hold up, though. Isnt IPC === *Inter process* communication, implying that there should be atleast processeses? And the answer to that is, yes there are - Electron divides the application into two conceptual processes "main" & "renderer" where if we go back to the driver and car analogy, "main" === car and "renderer" === driver. The renderer process is your website, it is what "Renders" on top of the main, electron process which handles communications with the underlying OS.

The way we go about doing this is by using two functions, ipcMain and ipcRenderer. As you might have guessed the renderer uses ipcRenderer and invokes calls which the ipcMain then handles.

Say you wish to make your app run whenever the operating system starts up. Surely you can't achieve this without the use of electron's underlying APIs

So what you'd end up doing is, in your web app you write a function to invoke a specific channel which the main (electron) process is listening to/expects you to invoke

```javascript
const { ipcRenderer } = window.electron;
 await ipcRenderer.invoke("SET_STARTUP", true);
```

This what you'd write in your web app and it would invoke this - the message would travel over on to electron, where you need to write some code which listens for this ipc invoke

```javascript
ipcMain.handle("SET_STARTUP", (event, start: boolean) => {
  app.setLoginItemSettings({
    openAtLogin: start,
  });
});
```

Alright, lets break down this code
ipcMain/ipcRenderer - As I explained these are the interfaces available on the respective processes to perform invokes/handle invokes
"SET_STARTUP" - this is the channel, in our car analogy, consider this to be the steering wheel. A car might have multiple such interfaces, like the gas pedal, the brakes etc. You, the driver, invoke these interfaces.
start - Just as we move the wheel by a set amount of "Degrees" we need to provide some meta data to these channels - this isnt always necessary, you may write some channels which don't need this metadata at all while some might need it.
app.setLoginItemSettings ({...}) - think of this as the part of the steering wheel that connects to the axle. This is the last abstraction layer available on electron which then directly speaks with the underlying OS to set the app to start at default on login.

## What have I learnt over time working with IPC's then?

### It's better to have a single place which handles all invokes

This might seem like common knowledge, but trust me, it isn't. The web is riddled with bad examples and especially with things like IPC which aren't extremely well adopted in the community just yet, we see this happen a lot.
Having all your invokes in  one place would mean you can handle if a invoke doesn't have an underlying handler for it.
You can also log how many times you invoke a particular channel over the lifetime of the app run and then use that data to perform optimisations over it.
While both of these can be achieved even if your invokes are all spaced out over multiple files, this would just increase the amount of code duplication over the codebase

### Use IPC

I have mentioned this in a full blown article in the past, but re iterating it - don't use window.remote even if its the easy way out!

### Good IPC architecture would mean all your business logic can sit in the web app

Electron updates are a pain to handle, and even once you have a streamlined system configured, each update ships with a fullblown browser which means update sizes are enourmous. In most professional scenarios, updates should be avoided as frequent 100mb+ updates aren't fun to have. As a result, if most of your business logic is handled by your web client, it would make life much easier as a simple refresh of the app would mean your clients are on the latest business logic. This wouldnt be possible without having a streamlined ping-pong of IPC calls between main and renderer process!

...and that's about it! This should help you get up and running at a basic level with your IPC setup, for more information, electron docs are the place to be!
