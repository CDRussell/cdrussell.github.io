---
layout: post
title: "Code Confessions"
date: 2019-07-26T00:16:12+01:00
description: Code Confessions; openly confessing your coding sins to your team. Involve your team and absolve yourself.
keywords:
  - code confessions
  - team work
  - collaboration
  - programming
  - shared responsibility
  - shared ownership

# featured_image is the hero image at the top of the page and the thumbnail preview from the home page
# images typically go into /static/images dir
list_image: /images/confession.jpg

permalink: 2019/07/code-confessions/

---

_This post describes code confessions; openly confessing your coding sins to your team._

I want to tell you about something I do regularly which I call a `Code Confession`. It's how I unburden myself from the guilt of having written bad code, bring it to the attention of the team to improve it and seek absolution for my coding crimes.

<img alt="sign saying next in line for confession" src="/images/confession.jpg" />

Let's say I've written some code but there is something I just don't like about it. There is a smell. The code solves the core problem though, and I know that if I had to, I could live with it. But yet I know that there is still something bad about it and, despite my own best efforts, it's the best I've come up with.

The smell could be anything; it could be as simple as a method I can't quite name well, or it could be more of an architectural decision, an added dependency on a library I didn't want to add, a hacky workaround etc... There are a million ways to write smelly code, and I've probably done them all at one time or another.

We all write bad code sometimes; there is no shame in it. Sometimes it's because we don't know any better. Sometimes it's because we have to take shortcuts to get something shipped. Sometimes it's because we're tired. Sometimes it's because we're trying to juggle too many tasks at the same time. Sometimes it's because we're just too close to the problem we're trying to solve.

No matter the particular reason for the bad code, the thing which often helps is to borrow a second pair of eyes on the problem. And so I grab a teammate on a call...

_"I have a confession to make..."_ is my opening gambit. My teammates know what's coming next; they've heard my confessions many times before.

- _"I can't think of a way to improve this..."_
- _"You're going to hate me but I've had to ...."_
- _"I've spent the morning cleaning up vomit from my desk as a result of the code I've just written, but..."_

And I get it all out in the open. You see, the worst thing you can do with this is to bury it. You can try to hide it in the codebase and hope no one notices in the code review. You can leave it festering in there and hope you're never held accountable for it. But you are accountable. Everyone on the team is accountable for it. And this is why `Code Confessions` work.

At a Code Confession, I'm telling the team "this code I've just written sucks", and highlighting that it's a problem for me. But I'm also warning them that this sucky solution, which only exists in my local branch currently, will become their problem soon. Once it is in the main codebase, it's everyone's responsibility to look after it.

The `Code Confession` is the time for the team to either come up with a better solution or to admit defeat as a group, and humbly accept that it's the best we can do. 

Sometimes during a `Code Confession` I'll hear that actually the solution isn't all that bad, and that can be nice to hear. But more often than not, we'll work together to find a better solution. One that smells less and one that the team is going to be happier with long term. By highlighting code I don't like but can't improve on my own, and confessing it upfront to the team, we come together to improve it or we agree to live with it.

The team feels satisfied as they got to help solve a problem, I feel better that I'm not carrying this problem alone, and the codebase gets better because everyone feels shared responsibility.

Give `Code Confessions` a try the next time you're left a little unsatisfied with some code you've written, and see how good it feels. Involve your team and absolve yourself.

_Forgive me teammates, for I have sinned..._

---
_Image Attribution: Photo by Shalone Cason on Unsplash_
<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@shalone86?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Shalone Cason"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Shalone Cason</span></a>