---
layout: post
title: Blockstack And The Path To Mobile Pt. 1
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

The simplest path to signup given the current state of Blockstack is to tunnel localhost:8888 in a mobile webview. “But wait! That’s just linked to a computer! That’s not mobile!” To which I would say “yeah. But rewriting the user registration for mobile is way more than a coding challenge. This is the appropriate scope for taking a first pass at this problem to understand how it might actually be solved relatively soon.


## Scope: What needs to be done?

The fastest way to build simple mobile applications right now is Expo.io.
The fastest way to use the existing registration flow is to install Blockstack browser and tunnel to localhost:8888.
The fastest way to get access to that mobile app is through an expo account.

To share expo apps on iOS you have to use a shared account[1]
On android, you can still just scan a QR code and be up and running.

Download the Expo client on [android](https://play.google.com/store/apps/details?id=host.exp.exponent&hl=en_US) or [ios](https://itunes.apple.com/us/app/expo-client/id982107779?mt=8):

Login with username: nicks_blockstack

Login with password: blockstack

Under the projects tab, you should see a project named `blockstackv1`.

Open the project.

The project is simple a WebView that tunnels to my localhost:8888 through ngrok. By the time you're reading this, it's likely non-functional, because I'm tunneling to my laptop, and depending on where you are in the world, talking to my laptop from wherever you are might be very very slow.

It's slow to begin with. It takes about 5 minutes to load on my device that's colocated with my laptop. I have some hunches about why… Your most likely outcome in follwing these steps is an empty white screen. That's ok, it'll get better, and in the mean time I've attached screenshots to support the claim that this works.

After the 5min loading period, you should see the typical Blockstack browser experience on mobile.

If you haven’t set up Blockstack on your computer, the signup workflow modal will sign you up.

If you have signed up on Blockstack, it appears to prompt you for your storage solution. Choose default.

You should now have your Blockstack browser working on your mobile device.

Done!


### Up and Running

This might seem unimpressive.

All we’re doing is tunneling from our phone to our desktop.

And all we did was wrap the Desktop browser WebView in a mobile WebView.

And we didn’t actually get the signup experience onto a mobile device, because we’re still linked to the Desktop.

And none of the Blockstack apps work, which makes it more strikingly obvious that Blockstack apps, even though the UI indicates they’re like mobile iOS or macOS apps… while in fact, they don't behave anything like either of those things.

And yet… seeing the Blockstack browser on mobile might generate evidence to support the hypothesis that the Browser needs to exist on a mobile device, and the signup workflow also needs to operate independently on a mobile device. No more computers. The phone is the new secure container.

## Notes
[1] This is recent, and it’s irritating because the solution to sharing apps easily involves either sharing your personal developer credentials with people you’re sharing the app with, or generating a new email and expo account for each project you want to share and sharing those project credentials with anyone who wants to see what you made. You can read more about it here: https://blog.expo.io/upcoming-limitations-to-ios-expo-client-8076d01aee1a

## Reference Links


## **Thanks** 
Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for working with me on this. You should hire him if you can.

