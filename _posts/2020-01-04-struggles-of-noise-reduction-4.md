---
layout: post
title: Struggles of noise reduction in RTC — Part 4
---

## The Epilogue

Finally — after going through the deprecated/non-optimal ways of approaching Noise reduction, let us use the ideal way now! AudioWorklets, it is!
Just as in Part 3, we will use RNNoise as our template for this walkthrough — however, this time around it’ll be running inside a worklet!
Remember the super high latency we faced in the scriptProcessor implementation? At a bufferSize of 16k for the scriptProcessor we were facing a latency of around a second or so! However, using the AudioWorklet API, the buffer size can go down to 128 frames, i.e. 2.667ms. We need to be, however, mindful of the fact that RNNoise’s buffer is fixed at 480 frames, thus the actual latency is 640 frames, i.e. 13.333ms.

Quick tip — Always, have your processor file, wasm, inside the public folder of your JS application. Especially if you check out a lot of the open-source solutions for this, they expect you to simply boot up a server and serve it over npx, and are generally not large-sized applications/introducing other complexities like React into the mixture. As a result straight up following’em, would be as good as entering the wild, wild west.
A general good rule of thumb would be to have three files in the structure of a processor, a runtime, and an actual wasm whenever you are interacting with wasm based features.
Folks over at Jitsi have developed a wasm port for rnnoise <https://github.com/jitsi/rnnoise-wasm>, along with a good third part port developed by <https://github.com/wegylexy/rnnoise_wasm>. You can’t go wrong with either of these

Another Quick Tip — ALWAYS have a fallback. AudioWorklet is an amazing thing to have in your arsenal but it's not supported for a large demographic of people, namely people hooked onto the apple ecosystem. Safari enforces its own set of laws governing audio contexts + does not support the use of audio contexts (as of writing this post) at all. So users from Safari (both iOS and Mac) won’t be able to use your app at ALL if you don’t have back up. I suggest using AudioWorklet-Polyfill as a fall back however in certain instances you might want to consider writing your own scriptProcessor node-based implementation. Why? Because the polyfill library will resort to a browser decided default for the buffer value of the scriptProcessor node (because it passes undefined inside the buffer value of scriptProcessor) as a result, these values may even default to as low as 256 which is a sure way of getting broken audio. Tread with caution.
Coming back to our audio worklet, you might also want to configure headers for your wasm otherwise you’ll run into issues where your server returns the .wasm with a header different than what the call expected.

With this, your implementation should now be up and running (note a lot of this would fall back to implementation specifics of your own app as to how to handle the media stream)
Here is a short but real-life like demo of our current app, notice, I didn’t exaggerate background noises (although even if I did so, the filters would remove them) but kept a normal state where the background noise was mainly caused due to the fan running (which too can only be heard if you wear headphones) <https://www.loom.com/share/57855f4108b04dec8d0153784875f2ba>
But why take my word for it? Try it out yourself for a quick demo at <https://app.voice-ping.com/demo>
Check out our app at voice-ping.com
