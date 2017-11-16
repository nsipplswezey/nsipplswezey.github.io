---
layout: post
title: VoltAGE The Camera Pt. 3
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

At this point, our react-native-camera component can get frame data from the obj-c camera layer. Now we want to integrate our neural net for image classification.

That means we need to require in the jetpac DeepBeliefSDK for iOS, as well as it's accompanying classes and methods.

## Up And Running




