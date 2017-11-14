---
layout: post
title: VoltAGE The Camera Pt. 2
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

We've got our fork of the react-native camera up and running, modified to console log something every time it processes a frame. In a separate app, we've got a neural net processing a frame every 300ms.

There are two next steps. One step is to start brining the image classifier into the react-native-camera. Another step is to pass a callback as a prop of the react-camera component, that is invoked at the objective-C layer with some data that can be made accessible at the react component level.

Let's tackle that second item first.

## Up And Running




