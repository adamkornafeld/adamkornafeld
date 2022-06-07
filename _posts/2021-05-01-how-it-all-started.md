---
layout: article
title: The one that tells the tale of how it all started
tags: pascal clock UI Dalí Salvador
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    src: https://media.npr.org/assets/img/2015/02/04/spiral-bbf98c485003ba03cc036a52dd4a4b068e96e965-s1600-c85.webp
---

When I was looking for a cover image for the [very first post](https://kornafeld.com/2021/04/10/on-naming.html) I almost instinctively reached out to one of my favorite paintings: [The Persistence of Memory](https://en.wikipedia.org/wiki/The_P ersistence_of_Memory) by Dalí. You might know it by the name: The Melting Clocks. It's only now, a couple weeks later that it dawned on me why. Read on to find out...


🎥 How it all started
---------

The story says Dalí got his inspiration for the painting from the surreal way he once saw a piece of runny Camambert cheese melting in the Sun. The melting clocks represent the _omnipresence_ of time, and identify its _mastery_ over human beings. I feel lucky to have been able to witness this painting in real life at the [MoMa](https://www.moma.org/) in New York City. 

I am fascintated by the concept of time. Don't really know (yet) why, but I always has been. Back when I was a freshmen at the university - Budapest University of Technology and Economics or [BME](https://www.bme.hu/?language=en) that is - the gateway drug language into software engineering they taught us was _Pascal_. I know, righ?! I feel old. During the first semester everyone had to pick an idea and implement it as their take home assignment over the course of those few months. When people think about Pascal, command line applications usually first pop in mind. Not for me though. Excited to get my hands dirty with _real_ software engineering - after having been hacking at it on my own well before university - I wanted to jump in the deep. 

So I picked a _graphics_ problem. An idea that I had for some time that found its roots in Dalí's painting and that I lacked the know-how on how to execute it up until that time. See, mechanical clocks, watches and timepieces have a major limitation in my eyes. The shape of them are mainly influenced by the lenght of their hour, minute and second hands. The longest hand sets the minimum of the radius the clock face can have. Anything smaller and the face would not be able to complete a full circle without crashing in the wall of the clock face. Or worse run off of the clock face. Sure, you can find rectangular clock faces every now and then. But that's usually the farthest the imagination of the creator would go. Enter Dalí. He was first to break down this barrier raised by the physical limitations presented by watch hands in his painting. However, he was cheating or more like lucky in a sense that through his painting he was able to freeze time so he didn't have to worry about the length of the clock hands.

⏰ Free-style clock face
---------

For the take home assignment my idea was to take Dalí's concept one notch futher leveraging the creative freedom provided to us by computers. Here's the brief algorithm for the code: 
- Step 1 Draw an arbitrary shape of a closed loop.
- Step 2 Point somehwere inside the shape to pin the center of the clock
- Step 3 Draw the hands of the clock starting from the center point according to the current time
- Step 4 Dyncamically update the length of the clock hands as they sweep around showing the current time to adjust for the given radius between the center point created in Step 2 and the outline created in Step 1

🔘 Connecting the dots
---------

It is indeed funny how life gives you dots and its up to you to connect them or not. This is me having a _great time_ realizing how I subconsciously used Dalí's painting as the cover of the first post of my blog in which I focus _mainly_ on software engineering. Only to realize a couple weeks later that how that very painting ties back into the early days of my software [endeavors](https://ndvr.com). Also fascinating that it seems to me that this idea has not been leveraged much yet with the dawn of all the smart watches out there that has the necessary screens built in to support dyncamically shaped watch faces. A missed opportunity?!

This post ended up being a bit unusual in that we haven't done much coding. But I will find some time to recreate that original dynamic clock of mine and write about it in a next post. Heck, I might even have the original Pascal code laying around somehere. And that wraps our _non-coding_ session for today. Happy coding!
