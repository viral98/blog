---
layout: post
title: From the eyes of a junior developer - Code Reviews!
---
![image](https://user-images.githubusercontent.com/25403969/106691080-e6003480-65f8-11eb-8378-716f6728020e.png)

Entering industry as full time SDE, I came in with the idea that the fact that I could write code during my internships well would translate well when I start writing code as a full time SDE - and then Code reviews happened.

To this date, I remember my first ever PR had 50+ comments with multiple changes requested, this seems extremely negative at first, because a lot of the changes requested seem extremely pedantic at the moment, but looking at it in hindsight I really feel, code reviews hands down are the best thing to happen to a junior developer - they might just provide you more value than the "online courses" or the "Tricky interview questions for X language" videos.
Note, this article does assume the fact that your senior developers are not toxic and are genuinely providing good reviews. Good reviews !== Easy reviews. Good reviews are the ones where the senior developer encapsulates  his thought process behind requesting changes.

This what a good code review look like - mistakes pointed out clearly, with links to the bits which are actually wrong along with the expectations of the developer
![image](https://user-images.githubusercontent.com/25403969/106691589-c584aa00-65f9-11eb-8338-d82faa7c3496.png)
After this review, I clearly know that action creators must never have any side effects in them, and that remains at the back of my head. Think about this while interviewing for react - Candidate A describes action creators, whereas candiadate B not only describes them but also describes how they must be used. Who do you think would be selected all else being equal?

Instead of looking at reviews with a negative light or with a mindset of "Oh no, here we go again" - think about it this way, when a developer with years of experience reviews your code, he is essentially giving away best practices that he has learnt over the course of his career in a short condensed way + now you have a way to implement those "best practices" in real time as opposed to just hear someone blab about them on a 15 minute youtube video and then forget about it the next day.

At the end, your own progression in tech is also dependent on how cleanly you can articulate your ideas into actual code, that not only acomplishes the task but is also clean & maintainable. Just about anyone can write hack fixes to finish up an issue, but the fact that you take the time to actually articulate code in a way that you are internally sure that this won't break under any known circumstance, is what sets apart good developers from the rest! The later comes with experience writing code - there is no shortcut to it, no course/youtube video can robustly encompass "how" to exactly write clean code in your specific circumstance but code reviews serve as an internal knowledgebase you can build upon, so once you encounter a situation and get requested to change the way you approached it, you now know for a fact how to approach the situation in the cleanest way possible.
