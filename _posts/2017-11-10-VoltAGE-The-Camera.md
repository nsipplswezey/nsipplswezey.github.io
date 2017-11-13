---
layout: post
title: VoltAGE: The Camera
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

We need a react-native camera that can pipe frames to our image classifier. Our image classifier is [https://github.com/jetpacapp/DeepBeliefSDK/tree/gh-pages](https://github.com/jetpacapp/DeepBeliefSDK/tree/gh-pages) and it's about 3 or 4 years old. 

Our react component is a fork of [react-native-camera](https://github.com/nsipplswezey/react-native-camera). 

And our starting point is the `captureOutput` method of our RCTCameraManager, which is invoked whenever a sample buffer from the camera stream is available.

And ideally, this is where we can run our CNN on frames as they become available.

```
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection
{
    //CVPixelBufferRef pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
    //[self runCNNOnFrame:pixelBuffer];
    NSLog(@"Hello");
}
```

With the react component side sorted out, we need to get up and running with the machine learning library. So let's get some jetpacc examples[https://github.com/jetpacapp/DeepBeliefSDK/tree/gh-pages#examples](https://github.com/jetpacapp/DeepBeliefSDK/tree/gh-pages#examples) up and running.

## Up And Running




