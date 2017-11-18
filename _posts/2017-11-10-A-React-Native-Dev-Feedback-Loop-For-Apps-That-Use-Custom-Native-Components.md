---
layout: post
title: VoltAGE A React Native Dev Feedback Loop For Apps That Use Custom Native Components
---



## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

This project is more involved than what's often projected as the use-case for react-native, because it involves a component –a customized CNN driven camera– that exists outside of the @expo SDK. Additionally, because the camera is involved in a UI/UX capacity, testing the CNN needs to actually happen on the device itself. Which means regularly building the app and running it on the phone. rather than just sending a javascript bundle over the wire to the Expo app. My current build-to-phone step seems to take in excess of 1-minute to bundle and deploy. This makes for a somewhat slow feedback loop. That slow loop is tolerable for building out the custom camera component. There's no alternative anyway. But for building out user interface components, 1-minute between changing some CSS, and seeing the new CSS update is a problem. We need a tighter feedback loop for UI work.

### A Two Speed react-native Dev Environment
The UI development environment can be fast, and use the current react-native expo tooling, along with a mock camera component, where the CNN detection events are triggered.

The CNN camera environment with it's 1-2 minute long builds, can run separately. This makes sense anyway, since really the CNN camera is a fork of the existing react-native-camera, and should be developed, managed and source controlled independent of VoltAGE.

## The Simplest Thing Possible

Camera environment in Xcode.

UI environment in expo.

Integration environment in Xcode. One approach to this would be to build a workflow where we eject our UI expo environment, and then integrate the CNN-camera. A simpler flow, is just to make sure we can copy-paste expo into our non-expo react-native app. 

## Up And Running

I have a VoltAGE code base I wrote a while back, before expo even existed. It contained a working UI, and a working CNN-enhanced-react-native-camera in an older version of react-native. The fastest "up-and-running" here is to create a fresh non-expo react-native project, and get as much of that old codebase working as possible. 

Essentially then, the task is to split that working project into a UI portion and put it into an expo UI development environment, with mocked CNN camera events, and make sure it runs in expo, and that changes made in that environment run in the actual deployed app.

Then based on that UI, continue to develop out the CNN enhanced camera as needed. 

Update: it looks like the CNN fork of react-native-camera will have to be rebuilt from scratch. But the previous UI is alive.

![voltage](https://user-images.githubusercontent.com/7946707/32696574-70902de6-c730-11e7-8027-b840e8bc7860.gif)


