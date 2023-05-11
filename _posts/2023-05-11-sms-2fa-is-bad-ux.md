---
title: "SMS 2FA is Bad UX"
categories:
  - Rant
tags:
  - UX
---

I know next to nothing about UX design but I know what I don't like.

Requiring you to enter a username / password then sending a code by plain old SMS has become a common practice for many sites. For example, here is a step in the login process to access Australian federal government services:

![myGov SMS screen](/assets/images/mygov-sms.png)

Even the happy path here is problematic: unless I have messages coming to the device I'm using to log in (which is a subject for another rant), I have to read a code from one device and type it into another. If you aren't sitting at a computer with your phone in front of you, that is annoying. If your sight isn't perfect and you need glasses for one device but not the other, it's suddenly a far more difficult task than intended.

But one unhappy case is awful. Imagine your phone number changes. Going further, imagine that changing your phone number was a last resort to stop receiving unwanted calls or messages. Now you can no longer log in to *any service that assumes it can send a message to your old phone number*.

Today, changing your phone number invalidates a large fraction of your online accounts. That means every interaction with banks, telcos, and governments reminds you of a traumatic event that led to the change of phone number in the first place.

We can do better. Stop making assumptions that phone numbers uniquely and permanently identify someone. Provide convenient and secure fallbacks. Have empathy for people who aren't using your site every day and may have other things going on in their lives.

Further reading:
	[https://github.com/google/libphonenumber/blob/master/FALSEHOODS.md](https://github.com/google/libphonenumber/blob/master/FALSEHOODS.md)