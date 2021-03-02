---
layout: post
title: From the eyes of a junior developer - Adding Loading fields on Redux sucks and why you should use async mutex!
---
We have all run into this at some point or other, we wish to perform a whole bunch of tasks, and at the same time prevent some other tasks to happen until we have a response come back from server.

## The EASIEST way to tackle this - Add Loading field

Well, the easiest way to approach this would be to set a flag like "loading" to true in your reducer whenever you run an action which would later require some sort of a server response.
