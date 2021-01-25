---
layout: post
title: Struggles of noise reduction in RTC — Part 2
---

![image](https://user-images.githubusercontent.com/25403969/105566179-c2afcc80-5d50-11eb-8c9f-74d320f06d94.png)

## The simple approach

I’ll be going through noise reduction in these blogs in the same way in which I personally learned and implemented noise reduction at p2p.inc
Some might think the first thing we’ll talk about will be filters and how to hook’em up together and pass your stream, but before that one really important point- the filters given with your getUserMedia API call are pretty cool and are truly the lowest hanging fruit! Use them to your advantage.

```javascript
const constraints = {
    audio: {
        deviceId,
        echoCancellation: true,
        echoCancellationType: { ideal: " system " },
        channelCount: 1,
        sampleRate: { ideal: AUDIO_SAMPLE_RATE },
        noiseSuppression: false,
        autoGainControl: true,
        googEchoCancellation: true,
        googAutoGainControl: true,
        googExperimentalAutoGainControl: true,
        googNoiseSuppression: true,
        googExperimentalNoiseSuppression: true,
        googHighpassFilter: true,
        googTypingNoiseDetection: true,
        googBeamforming: false,
        googArrayGeometry: false,
        googAudioMirroring: true,
        googNoiseReduction: true,
        mozNoiseSuppression: true,
        mozAutoGainControl: false,
        latency: 0.01,
    },
    video: false,
};
const stream = await navigator.mediaDevices.getUserMedia(constraints);
```

### Pay special attention to

```javascript
echoCancellationType: { ideal: " system " }
```

In RTC scenarios, your noise reduction algorithm might (and most likely will) introduce a delay, thus, users using speakers and a mic might possibly have an echo or a loopback, this parameter tries to prevent that from happening
Moving on to the actual filter based setups — these are quite famous with most podcasters and are fairly easy to implement in JavaScript
WebAudioAPI — Our Saviour
While the options I listed above are easily available, anyone considering RTC seriously would not just vouch on these parameters. The WebAudioAPI thus provides us with a diverse set of nodes over which we can perform our straight forward filter based noise reduction.
Before we start with these filters, we need to set our expectations straight — these filters won’t do good in reducing acoustic noise, i.e. the noise which we generally associate with the word “noise” i.e. a dog barking/drilling/the sweet sound of cherry MX switches hitting someone’s mechanical keyboard — these filters help reduce the static which we often hear in low quality or early-stage RTC applications, as users use a wide variety of mics and underlying OS’s we cannot rely on the underlying implementation to provide us with a static-free stream.
We start off with the one thing without which none of this would be possible i.e. our AudioContext. Define your AudioContext and pull a stream inside a variable
Quick Flashback to third-year undergrad — The number of data points in a sound file depends on its sample rate. You might have seen this number before; the typical sample rate for mp3 files is 44.1 kHz. This means that, for every second of audio, there are 44,100 individual data points.[1]

```javascript
const AudioContext = (window as any).AudioContext || (window as any).webkitAudioContext
const context: AudioContext = new AudioContext({ sampleRate: YOUR_AUDIO_SAMPLE_RATE });
const input = context.createMediaStreamSource(stream); // We defined stream in the last code block
```

The WebAudioAPI provides us with a BiquadFilterNode, instances of which, we will use to generate a whole host of filters
Simply speaking the Biquad Filter filters each channel of the input signal with the specified biquadratic infinite impulse response (IIR) filter. When you specify the filter coefficients and type, the node implements static filters with fixed coefficients.
Here is where specificity comes into the picture — you can slam in a high pass filter at around 100Hz which will take away much of your static, but would also take away the depth from male voices from your target demographic. Having a slightly conservative value here would be a better idea. A very high value will make your voice sound TINNY

```javascript
const highpass = audioContext.createBiquadFilter();
highpass.type = "highpass";
highpass.frequency.value = YOUR_VALUE;
highpass.Q.value = YOUR_VALUE;
```

Another thing to note here would be the fact that Web Audio filters can be configured with three parameters: gain, frequency, and a quality factor (also known as Q). These parameters all affect the frequency response graph differently, and while I am not going over each one of these, feel free to play around with those options for your filters!
Next up we hook up a lowpass filter, we generally keep its value well below the 8kHz range, pushing this to be aggressive will make your voice sound Flat/Muffled
const lowpass = audioContext.createBiquadFilter();

```javascript
lowpass.type = "lowpass";
lowpass.frequency.value = YOUR_VALUE;
lowpass.Q.value = YOUR_VALUE;
```

This filter passes all the frequencies which are above its cut off frequency and blocks all the frequencies below the cut off frequency (vice versa for lowpass).
You can check out the other varieties offered by the BiquadFilter here <https://developer.mozilla.org/en-US/docs/Web/API/BiquadFilterNode>
Meanwhile, let us connect all of these nodes together so our audio passes through

```javascript
source.connect(lowpass);
lowpass.connect(highpass);
highpass.connect(destination);
```

Here destination is of type MediaStreamAudioDestinationNode which we can then use for RTC

## Wait, what’s the struggle then?

The struggle here is the fact that the NR offered by this solution is “Weak” to put it mildly. This gets rid of static and that’s about it, everything else, passes right through!

## Up next…

We’ll discuss a hundred different reasons why NOT to use scriptProcessor nodes even if you find some code in the crevices of the web which directly solves your problem using it, along with the solution to scriptProcessor i.e. audioWorklets. We’ll be making use of these worklets to then perform NR using an actual Machine Learning model to clean up our input stream
