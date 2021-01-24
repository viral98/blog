---
layout: post
title: Struggles of noise reduction in RTC — Part 3
---

![image](https://user-images.githubusercontent.com/25403969/105566149-9431f180-5d50-11eb-9d46-53c28fc5b948.png)

Moving on from the barebones filter-based implementation we checked in Part 2 which you can check out here, any firm that has a business model based around RTC would wish to implement a more robust, all-encompassing NR module.
The best example to take for this series would be the current industry standard, RNNoise. If we check out their demo, inspecting the page, we’d find dumps to their files for using RNNoise in the most non-BS, straightforward way if you know basic web audio API standards and how to pass in your input stream.
Here’s where things become super tricky. As this is a demo and not a part of their official repo, there’s little to no documentation in case you run into errors, plus they have mentioned that they (i.e. the rnnoise team) have introduced a delay in the live demo on purpose so that the users can hear feedback of their voice.
The latter can be easily disproven by checking out <https://jmvalin.ca/demo/rnnoise/record.js>, they make us of…. you guessed it, a scriptProcessorNode

```javascript
var bufferSize = 16384;
var sampleRate = audioContext.sampleRate;
var processingNode = audioContext.createScriptProcessor(bufferSize, 1, 1);
var noiseNode = audioContext.createScriptProcessor(bufferSize, 1, 1);
```

On top of this, looking closely, we see that they opt for a bufferSize of 16k. Keep this in mind, as we move forward.
Checking out the documentation for scriptProcessorNode, we get greeted with this big red wall of text

![image](https://user-images.githubusercontent.com/25403969/105566158-a4e26780-5d50-11eb-9dcf-571de4c2f02d.png)

While it is deprecated, startups might just wish to use it because of the fact that there is a whole lot of code available for it and not a whole lot of code/documentation for its successor, the audioWorklet. Here’s where we start running into problems
ScriptProcessorNode, is essentially a zero-sum API, in the sense, something has to be lost in order to achieve gains somewhere else. In the case of scriptProcessorNode, you got to pay a high computational & latency cost for good processing.
This tradeoff is dictated using the bufferSize, which varies from 512 (least computation & latency cost, worst audio output, a lot of audio breakages) to 16k (highest computational cost, the latency of at least a second if you are doing anything heavy but comparatively, decent processing). This is why the demo of rnnoise has some “delay” it hasn’t been introduced by code directly but introduced because of the usage of scriptProcessorNode.
This is mainly due to the architecture of the node, it runs on your main thread!!! This means all processing comes in the way of any other rendering that is happening in your app.
The scriptProcessorNode’s audio processing function i.e. the one that does all the heavy lifting for us, runs on the main thread (all the other web audio API stuff, runs on their own thread)
Now, this processor, first performs whatever computations it has to perform on the MAIN thread, then conveys the result back to the WebAudioRender thread, and that then relays that data over onto any other node(s) that follow the scriptProcessor. As you can imagine, things get real ugly, real quick.
Lets try to make this work (to some extent)
Going back over onto the code from rnnoise demo,
We can get rid off their noise processing node, it serves no purpose to us, as we don’t want to introduce more noise!

```javascript
noiseNode.onaudioprocess = function (e) {
    var input = e.inputBuffer.getChannelData(0);
    var output = e.outputBuffer.getChannelData(0);
    for (let i = 0; i < input.length; i++) {
      if (addNoise) {
        output[i] = input[i] + (Math.random() / 100);  
      } else {
        output[i] = input[i];
      }
    }
  };
//Get rid off this and all the declarations aroun this
```

Next, we got to lower the bufferSize if we wish to achieve half-decent latency, and not fry the CPU’s of our customers using apple devices with “amazing” fan curves (pun intended)

```javascript
var bufferSize = 4096;
```

This should be fine if your focus is latency over the strength of noise reduction, but still keep in mind, if your room peers increase you will most likely face audio breakage.
In the following part, we’ll be going over how to overcome the challenge thrown to us by the scriptProcessorNode
