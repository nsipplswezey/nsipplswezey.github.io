---
layout: post
title: Blockstack And The Path To Mobile Pt. 2
---

## Goal

Blockstack registration needs to be easier, simpler and less distracting. The developer authentication and storage flow are downstream problems from signup… so let’s see how signup could be made easier.

The user story I’d like to see fulfilled:
“As a user, signup might be so easy, that a user might be able to accidentally sign up for Blockstack”

Needless to say that’s a lot more work than the challenge warrants. 
But I wanted to get started on that problem, because it’s the problem that I can’t disprove in my experience so far.

Most human beings on the planet don’t have computers, and increasingly may never own a desktop or laptop computer. 
Every one has a phone.
So to get to a world where signup might be so easy that a user might somehow be able to easily accidentally sign up for block stack, a user needs to use Blockstack without having a desktop/laptop computer.

The first step towards that world is that specifically the signup flow needs to work on a phone. Not authentication, or sign in. But Signup.

The simplest path to signup given the current state of Blockstack is to tunnel localhost:8888 in a mobile webview. We did that in our last post.

Now we're going to dig deeper and start looking into a more robust and thoughtful solution. A Blockstack browser for Android and iOS raises some interesting challenges. How do I link or share relevant user identity information between my mobile device and my desktop? Can I run a Blockstack dapp exclusively on a mobile device? Are Blockstack dapp developers even creating responsive or progressive web apps?

These are all great questions, and highlight –to me at least- why the mobile story for Blockstack needs to be aggressively fleshed out. In essence, any large platform is a contract offer extended from platform owners(Blockstack) to platform consumers(dapp developers). When developers choose to use a platform, they're entering into a contract with the platform creators, and the quality of their working partnership depends on the quality of that contract. Furthermore, it's up to the platform developers(Blockstack) to create and maintain the best possible contract with their consumers(dapp developers)..  If the contract is incomplete, irregularly maintained or frequently violated, rationale developers will stay away. A missing piece of the Blockstack contract –again just to me– is mobile.

## Scope: What needs to be done?

The first step in the mobile story of Blockstack was seeing if the current Blockstack browser setup could run on a mobile device. It can, although it's unclear at this time where user configuation data is being stored after signup. I assume it's on the machine that's hosting the Blockstack REST API, and serving up the dapp UI at localhost:8888.

The next step is to figure out how many of the concerns of the Blockstack browser can be natively ported to a mobile device.

Can the Blockstack mobile interface store all the necessary config environment with good security, availability and recoverability?

Can the Blockstack mobile interface reliably communicate with all the other Blockstack services(gaia, atlas, bitcoin, others)?

Can dapps run in a simple webview?

Is the UX sufficient?

I want to add a short note here, which is that this project has personal value to me. I would love to replace every mobile app I use with a Blockstack version.

### Up and Running

## Notes

## Reference Links

## **Thanks** 
Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for working with me on this. You should hire him if you can.


