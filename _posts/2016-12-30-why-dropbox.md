---
slug: "2016-12-30-why-dropbox"
title:  "Why Dropbox?"
date:   2016-12-30 19:42 +1100
categories: "blog"
---

# Why Dropbox?

Well, it is official. I am killing the backend for my contact sync [app](https://play.google.com/store/apps/details?id=com.noni.embryio&hl=en). When I first built it, in early 2014, I was teaching myself to code both in Java and in Python.
I had also just applied to get into an accelerator - unfortunately which I got rejected from. I then applied to be a part of Microsoft's [Bizspark](https://bizspark.com) program, and it kinda worked! I got in.
That was pretty rad because I got a lot of Windows Azure credit. Life was awesome. Unfortunately, and I didn't realise at that time but I was taking on too much. I was trying to learn stuff,
I was working full time and I was building stuff that probably needed someone to work full time on. Eventually, I kinda ended up changing careers and taking quite a bit of a paycut so unfortunately I cannot keep funding Azure compute as it is a sizeable cost every month.

The main aim of the app was to give users total control of their contact data and also giving them a delete button to remove all traces of data that was being backed up - so eventually it only really
made sense to use a backup services provider that was:

- free.

- offered oauth2.

- let the user delete app data and revoke app permissions whenever they wanted.

- popular. So, fewer users actually have to register for a new account.

And, so today, as I have just finished building the Address book sync Dropbox integration, I figured I'd try and provide some clarification around why the changes.These changes are pretty transparent and there is actually no change in functionality of the app.
I am also kinda relieved because patching and updating servers, libraries and ensuring that Azure connectivity does not die is kind of a full-time DevOps role and I
am very short on time. The app however is and will always remain free. Thanks for your understanding.