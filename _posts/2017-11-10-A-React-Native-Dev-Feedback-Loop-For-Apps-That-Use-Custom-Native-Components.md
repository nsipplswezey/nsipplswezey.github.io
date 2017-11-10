---
layout: post
title: VoltAGE-A-React-Native-Dev-Feedback-Loop-For-Apps-That-Use-Custom-Native-Components
---
## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

This project is more involved than what's often projected as the use-case for react-native, because it involves a component –a customized CNN driven camera– that exists outside of the @expo SDK. Additionally, because the camera is involved in a UI/UX capacity, testing the CNN needs to actually happen on the device itself. Which means regularly building the app and running it on the phone. My current build-to-phone step seems to take in excess of 1-minute to bundle and deploy. This makes for a somewhat slow feedback loop for code changes. That slow loop is tolerable for building out the custom camera component. There's no alternative anyway. But for building out user interface components, 1-minute between changing some CSS, and seeing the new CSS update is a problem. We need a tighter feedback loop for UI work.



## The Simplest Thing Possible



## Up And Running





