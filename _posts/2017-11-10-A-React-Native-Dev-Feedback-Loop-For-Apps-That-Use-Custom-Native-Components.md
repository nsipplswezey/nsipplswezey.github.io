---
layout: post
title: VoltAGE A React Native Dev Feedback Loop For Apps That Use Custom Native Components
---



## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

This project is more involved than what's often projected as the use-case for react-native, because it involves a component –a customized CNN driven camera– that exists outside of the @expo SDK. Additionally, because the camera is involved in a UI/UX capacity, testing the CNN needs to actually happen on the device itself. Which means regularly building the app and running it on the phone. My current build-to-phone step seems to take in excess of 1-minute to bundle and deploy. This makes for a somewhat slow feedback loop for code changes. That slow loop is tolerable for building out the custom camera component. There's no alternative anyway. But for building out user interface components, 1-minute between changing some CSS, and seeing the new CSS update is a problem. We need a tighter feedback loop for UI work.

So what needs to be done, is to create a UI development environment where the camera functionality is mocked out. And have a seperate development environment for building the CNN enhanced react-native-camera component.

## The Simplest Thing Possible

Camera environment in Xcode.

UI environment in expo.

Integration environment in Xcode. One approach to this would be to build a workflow where we eject our UI expo environment, and then integrate the CNN-camera. A simple flow, is just to make sure we can copy-paste expo into our non-expo react-native app. 

## Up And Running

I have a VoltAGE code base I wrote a while back, before expo even existed. It contained a working UI, and a working CNN-enhanced-react-native-camera. So the fastest "up-and-running" here is to create a fresh non-expo react-native project, pull in that codebase and get it working. Essentially then, the task is to split that working project into a UI portion and put it into an expo UI development environment, with mocked CNN camera events, and make sure it runs in expo, and that changes made in that environment run in the actual deployed app.

Then based on that UI, continue to develop out the CNN enhanced camera as needed.



