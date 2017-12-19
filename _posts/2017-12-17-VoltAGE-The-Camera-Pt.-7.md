---
layout: post
title: VoltAGE The Camera Pt. 7
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera. 

In simpler terms, you point your mobile device camera at an object you want VoltAGE to recognize. With your camera pointed at the object, the stream of images your phone is capturing get sent to a machine learning algorithm. A bunch of computation happens based on the images your camera is feeding to the algorithm, and after about 20 seconds of pointing your camera, that algorithm outputs about 1.2mb of data. We can then pass that 1.2mb of data to another machine learning algorithm, and then again start sending the stream of images from your mobile camera to *that* algorithm, which uses the 1.2mb data the first algorithm generated to tell you on a confidence scale of 0.00 to 1.00 how much the thing that your camera is pointed might be the thing it was pointed at during the 20 seconds spent creating the 1.2mb of data. 

The **model parameters** are the 1.2mb of data. The **convolutional-neural-net(CNN)** is the algorithm that combines the images from your front facing camera and that 1.2mb of data and outputs a number between 0.00 and 1.00. That number between 0.00 and 1.00 can be used as a **detection score** for the physical object using the front-facing camera.

The idea is to use all this to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

These items are to be gifted. And the phone is to be used to facilitate the gifting. And to develop a clean ML driven gifting flow, we're going to need two phones that are interacting with our VoltAGE targets. With what we've done so far, I could go out and grab another iPhone, or resucitate one of my old ones. Or I could rebuild everything we just did for iOS, for Android, and use my second phone which is a Nexus 6p, and in-so doing enable VoltAGE for Android users.

What's involved here:

We're going to add a class to the Android side of our fork of react-native-camera to encapsulate all of our DeepBelief ML logic.

The implementation is going to be an Android activity, which is the android class that handles views that require user interaction like a modal. This approach is slightly orthogonal to the react-native component approach, because instead of exposing the DeepBelief methods directly as javascript methods to be called in the react layer where we might build our UI exclusively, we're instead triggering Android UI. Conceptually this adds some complexity, because our UI concerns are now potentially split between two different layers of our application. Having said that, our Android activity doesn't contain any direct UI. Essentially it's set to "display: none" and serves as a wrapper for our machine learning methods. It works for now. A refactor for later might be looking into some other way to get this working besides the Activity class. We'd have to find an alternative to the provided `onCreate` `onPause` and `onResume` methods that we're overriding.

Also since we're calling C from java, we're going to be adding two dependencies to our Android java project: `libjnidispatch.so` and `libjpcnn.so`.

There also exist some quirks in how resource and asset file paths work when bundling our files. Our ML tool depends on a network file that holds the trained version of our convolutional neural net. We also have two predictor .txt files as well that have been trained on our VoltAGE targets. There exists a need for what to me seems like excessive copying of resources. This would also be a good place for later refactor/optimization, but we'll leave it as is for now.

We're going to set those ideas aside, and just focus on getting up and running on Android. 

### Brief Note
This is going to be a longer post(or series of posts) in that what was covered for iOS in 5-6 smaller chunks is going to get a faster and looser treatment here. Additionally, the Android side of react-native-camera is factored into more classes, and things appear to be a bit more verbose on the Java side. All this is to say, if you're reading this closely to follow along in technical detail, you're more likely to be getting a quick gist of it all by scanning the code snippets in the posts. I'm going to throw in links to the actual commits to our fork of react-native-camera and VoltAGE, as a more enduring reference technical reference in context. Additionally, you can assume that I won't be linking any breaking commits. Additionally, I think it's worth noting that these blog posts represent a cleaned up more linnear approach to building out these features, which emerged from a much more typical non-linnear discovery, coding and debugging process which has been smoothed out into a more linnear flow. 


## Let's get to it
Since we haven't touched the Android side of our application, let's get it running with a fresh clone of VoltAGE. At this point in time, our most recent commit is https://github.com/nsipplswezey/VoltAGE/commit/2e7b2763bd5c3bb52fc90305c65b98e59b911ab6

```
git clone https://github.com/nsipplswezey/VoltAGE.git
cd VoltAGE/nextVoltAGE
npm install
react-native run-android
```




## Up And Running

There's room, it seems, for some optimization. Our classification and prediction step takes 4-5 seconds on a modern Android device. This could be improved with a Linnear Algebra library called [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) but that's another optimization for later.

```
12-18 11:06:54.452 31479 31497 D ReactNative: jpcnn_predict() value is 0.09783074.
12-18 11:06:54.452 31479 31497 D ReactNative: jpcnn_classify_image() + predict() took 5.302 seconds.
12-18 11:06:59.310 31479   399 D ReactNative: jpcnn_predict() value is 0.4637906.
12-18 11:06:59.310 31479   399 D ReactNative: jpcnn_classify_image() + predict() took 4.183 seconds.
12-18 11:07:04.974 31479 31522 D ReactNative: jpcnn_predict() value is 0.039335493.
12-18 11:07:04.974 31479 31522 D ReactNative: jpcnn_classify_image() + predict() took 4.988 seconds.
12-18 11:07:10.560 31479  1586 D ReactNative: jpcnn_predict() value is 0.7373725.
12-18 11:07:10.561 31479  1586 D ReactNative: jpcnn_classify_image() + predict() took 4.888 seconds.
12-18 11:07:15.750 31479 31497 D ReactNative: jpcnn_predict() value is 0.75858045.
12-18 11:07:15.750 31479 31497 D ReactNative: jpcnn_classify_image() + predict() took 4.482 seconds.
12-18 11:07:20.879 31479   399 D ReactNative: jpcnn_predict() value is 0.78222394.
12-18 11:07:20.879 31479   399 D ReactNative: jpcnn_classify_image() + predict() took 4.493 seconds.
12-18 11:07:25.807 31479 31522 D ReactNative: jpcnn_predict() value is 0.55595887.
12-18 11:07:25.807 31479 31522 D ReactNative: jpcnn_classify_image() + predict() took 4.222 seconds.
12-18 11:07:30.872 31479  1586 D ReactNative: jpcnn_predict() value is 0.1456414.
12-18 11:07:30.872 31479  1586 D ReactNative: jpcnn_classify_image() + predict() took 4.449 seconds.
12-18 11:07:35.610 31479 31497 D ReactNative: jpcnn_predict() value is 0.13940312.
12-18 11:07:35.611 31479 31497 D ReactNative: jpcnn_classify_image() + predict() took 4.156 seconds.
```




