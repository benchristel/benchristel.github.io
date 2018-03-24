---
layout: post
title: Whose Error?
---

# Whose Error?

What's this?

<div style="font-family:'Times New Roman', serif;font-size:24px;background-color:#fff;padding:18px;"><strong>404 Not Found</strong></div>

> That looks like an error page.

Right. And *whose* error is it?

> I'm not sure I understand.

Computers don't make mistakes. Well, sometimes a cosmic ray flips a bit, but let's forget about that for now. When there's an error, it's almost always because some *person* goofed. Errors happen because *to err* is human. So, *whose* error is a 404 page?

> Well, generally, when I get a 404 page it's because I typed a URL wrong. So this is probably the user's error.

Okay. How about this one?

<div style="font-family:'Times New Roman', serif;font-size:24px;background-color:#fff;padding:18px;"><strong>500 Internal Server Error</strong></div>

> Hmm... a 500 error is when there's an uncaught exception or something in the server code, right? So this is probably the programmer's error.

Good. Now, what if you're using an image-editing app and you see something like this?

<div style="font-family:'Arial', 'Helvetica', sans-serif;background-color:#f88;color:#800;padding:6px;margin:18px;">
  Could not fetch image from the CDN
</div>

> Well, just from that, it's hard to say. Maybe the CDN is down? Or the app isn't configured properly? I'm going to guess programmer error.

Well, look, if the CDN is down, is the app programmer really responsible for that?

> No, but the CDN maintainers are. So it's still programmer error.

Okay. Now what if I told you that the user of this app didn't know before they saw this error that a CDN was being used? What if they don't even know [what a CDN *is*](https://en.wikipedia.org/wiki/Content_delivery_network)?

> Well, that's unfortunate for them but I don't see how that changes things.

It does change things. If the user isn't aware of the CDN until they see the error, then this app is violating the [balanced abstraction principle](https://codurance.com/2015/01/27/balanced-abstraction-principle/).

> What? I thought the balanced abstraction principle was about how to structure code. How can you say an *app* is violating the balanced abstraction principle when you don't know anything about its code?

The principle is not just about code. It's about all systems, including whole applications. If the CDN is not part of the user's mental model of the app, and then they see an error message about it, the app's implementation details are leaking through its interface. And that's bad. If the user spends most of their time in the app manipulating images or whatever, they shouldn't be confronted with an error message about a CDN, because then they end up having to understand both the interface of the app and how it's implemented behind the scenes.

> Okay, I think I understand what you're saying. Basically, the app developer should catch the error from the CDN and display their own message that doesn't reveal implementation details. "Could not fetch images" or something.

Actually, no.

I'm saying the app developer shouldn't be relying a CDN *at all*.

> What?

You heard me right. The app should not depend on a CDN.

> That seems a bit extreme.

Not everything that seems extreme is bad.

> Okay, I'll play along. Why should the app not depend on a CDN? What if that's the most efficient way to get the content to the user?

Doesn't matter. Let's go back to the discussion about responsibility for errors. If the CDN is down, is the user responsible?

> Um... no?

And is the app developer responsible?

> I guess we already said no, so I'll say no.

And neither the user nor the app developer can do anything to fix the error?

> I suppose they could bother the maintainers of the CDN, but other than that, no, I guess not.

Well, there you go. From the perspective of both the user and the developers, the app is just plain *broken* until the CDN comes back. They're both at the mercy of the CDN maintainers. But in some sense, the app developers are *still* responsible for causing this error, *even though they can't fix it*, because they were the ones who chose to use the CDN in the first place.

> Well, you know what they say about power and responsibility.

I know what they *should* say: that if you take on responsibility while abdicating power over the thing you're responsible for, you're in for a bad time.

> You're saying that's what these hypothetical developers did. They're responsible for making an app that serves images, but they have no power to fix this particular issue where images aren't getting served.

Yes. This is why it seems that, despite all the engineering progress we've made in the last decade, we still can't have nice things. Our programs depend on things that neither the programmers nor the users are responsible for or able to fix. When something goes wrong, we have to delve through layers we didn't even know existed to find the person who can fix our problem. The end result is systems that are confusing, hard to maintain, and where most bugs seem to lead down an endless rabbit-hole of obscure root causes.

> But how does it make anything better if they don't use the CDN? Are they supposed to use the app server to serve the images? Sure, now they can fix the problem when it happens, but the problem is more likely to happen in the first place. Actually, it's worse: if the server gets too busy serving images, it will slow down the whole app for everyone, whether they're on a page that needs images or not.

That might be fine. Users get that sometimes apps are slow. It's not great when they're slow, but it's certainly not *unexpected*. And [*not surprising people*](https://www.joelonsoftware.com/2001/10/24/user-interface-design-for-programmers/) is a pretty important tenet of software design.

Let's go back to the question of what error message to show the user when the images don't load: from a user-experience perspective, it really doesn't matter if the error mentions the CDN or not. The error is inexplicable in terms of the user's mental model of the system, which is likely to be that their browser is communicating with a single server that's running the whole application. Thus, it's a violation of the balanced abstraction principle regardless of what the message says.

This happens within our code, too. Think of NPM packages that have deep dependency trees. You probably never think about all those transitive dependencies until something goes wrong. But when it does go wrong... hoo boy. Have you ever tried to debug an `npm install` issue where the error is coming from some package you've never heard of?

> Yes... but, look, we're not going back to the days when people didn't have package managers and maintained their own servers on actual hardware. That was awful! We had to spend so much time monitoring and deploying and installing things and patching libraries that we barely had time to write any actual code! CDNs and AWS and stuff aren't perfect, but they work great most of the time, and they're way more reliable than anything I could build myself. And NPM is the same; sometimes things break, but 99% of the time it makes everything way faster.

We don't have to go back to the bad old days. We just have to be mindful of the fact that dependencies outside the user's mental model of the system create user-experience issues when they fail. From our users' perspective, a error in one of our dependencies is no different from an error in our code, and they will make us responsible for fixing it.

And don't forget, programmers are also users—of each others' code.
The user of the library you write today may well be your coworker or your future self, so even if you don't have "end users", it pays to think about user experience. And the more dependencies you have, the more likely you are to surprise your users with bizarre errors from systems they've never heard of.

> I'm still not sure how to address the problems you've described without making everything worse.

I agree, it's a hard problem. I think we're only just beginning to be aware of the degree to which it *is* a problem.
For the last decade or so, we've steadily added more and more dependencies to our software stacks, and it's enabled us to create a lot of stuff really really fast. Now, I think we're starting to pay the price. We may have hit "peak reliability", where the compounded failure probabilities of our mountain of dependencies start to outweigh the gains from abstraction and code reuse.

If we want to keep creating really *dependable* software—and that includes apps, APIs, libraries, everything—we have to take a good hard look at our *dependencies* and make sure we're okay with taking on some of the responsibility for keeping them stable. And if they're not absolutely necessary, we should cut them out.

Concretely, that means:

- Work full-stack. Structure your team to be full-stack if that's your role. This encompasses both devops and application-level programming.
- If you must depend on something, depend on open source.
- Understand your dependencies well enough to submit a fix to their maintainers if you find a bug.
- Take responsibility for fixing problems in your dependencies and communicate to your supervisors that this *is* your responsibility as a software engineer.
- Ensure your dependencies have shallow dependency graphs of their own.
- Be completely transparent about what your code depends on. If it looks like you depend on too many things, depend on fewer things.
- Don't require users to understand your implementation before they can understand the behavior at the interface they're consuming.
- Seek simple interfaces, even if you have to make the implementation slightly more complex. But don't oversimplify to the detriment of reliability, and beware of "magic" interfaces that seem simple but have complex failure modes.
- Don't use the network if an offline experience could be just as good.
- Degrade the experience gracefully if some component fails. In the CDN example, the app could prefer to serve images from the CDN but get them from the app server if the CDN is down.
- Don't create a distributed system until and unless you really need to.
