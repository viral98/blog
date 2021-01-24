---
layout: post
title: Struggles of noise reduction in RTC — Part 1
---

![image](https://user-images.githubusercontent.com/25403969/105566197-da875080-5d50-11eb-8b62-8b542c86fcbd.png)

## Prologue

In just about all the apps we use for real-time communications i.e. calling, there is some level of noise reduction offered. This is due to the fact, the internet is filled with extremely heterogeneous devices, with microphones ranging from a whole variety of configurations. Thus all of these introduce a different level of unwanted frequencies and noises into the webRTC stream these apps use.
Thus, noise reduction at the bare bones would be simply chopping off these unwanted frequencies and at the highest level would be just keeping the intended user’s voice in the stream and removing just about everything else.
The heterogeneity in the web doesn’t end being a problem — the fact that your application can be used by folks with a $500 laptop or an all maxed out MacBook Pro, doesn’t help!
Most applications now are left with two options — hope that your users don’t use a potato for video/voice calls OR throw some good deal of money at the problem, and perform noise reduction on their stream while the stream is in transit
The Expensive Route
The latter looks like an enticing option, in fact, it is being used by Google meet. When you’re on a Google Meet call, your voice is sent from your device to a data center, where it goes through the machine learning model on the TPU, gets re-encrypted, and the newly modified data is injected back in the stream. (Media is always encrypted during transport, even when moving within Google’s own networks, computers, and data centers. There are two exceptions: when you call in on a traditional phone, and when a meeting is recorded.) One can easily imagine that this method can go south real quick by introducing additional latency if not implemented correctly or cause a good deal of expense behind the TPU’s[1]
While Google has all the resources, both monetary and in terms of engineering to throw at this problem, a startup at its early phases of development most likely won’t.

## The Pragmatic Route

This now leaves us with option 2, the likes of which are used by companies like krisp.ai and, voice-ping.com along with most other known RTC applications — they use the user’s local resources to perform noise reduction and most of them have a filter aggressiveness scalability option based on whether they are using a potato or a MacBook Pro. These filter aggressiveness can be a wide variety of things, from simply increasing the buffer size over which we perform batch computations or at a very basic level simply the variety of filter nodes we attach.

## Up next

In the coming parts, we will go through the implementation methodologies, both primal and fairly advanced ways of achieving this along with ways in which we can (to some extent) tackle the challenges I just mentioned above
