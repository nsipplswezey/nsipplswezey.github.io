---
layout: post
title: VoltAGE The Camera
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

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

First off, the jetpac examples –which are ~4 years old– are actually based on Apple's 4 year old examples of camera apps, which use a `require(condition, bail)` pattern for error handling in three instances. This throws a `use of undeclared identifer 'bail'` error in modern Xcode. 

```
// utility routine used after taking a still image to write the resulting image to the camera roll
- (BOOL)writeCGImageToCameraRoll:(CGImageRef)cgImage withMetadata:(NSDictionary *)metadata
{
	CFMutableDataRef destinationData = CFDataCreateMutable(kCFAllocatorDefault, 0);
	CGImageDestinationRef destination = CGImageDestinationCreateWithData(destinationData, 
																		 CFSTR("public.jpeg"), 
																		 1, 
																		 NULL);
	BOOL success = (destination != NULL);
	require(success, bail);

	const float JPEGCompQuality = 0.85f; // JPEGHigherQuality
	CFMutableDictionaryRef optionsDict = NULL;
	CFNumberRef qualityNum = NULL;
	
	qualityNum = CFNumberCreate(0, kCFNumberFloatType, &JPEGCompQuality);    
	if ( qualityNum ) {
		optionsDict = CFDictionaryCreateMutable(0, 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
		if ( optionsDict )
			CFDictionarySetValue(optionsDict, kCGImageDestinationLossyCompressionQuality, qualityNum);
		CFRelease( qualityNum );
	}
	
	CGImageDestinationAddImage( destination, cgImage, optionsDict );
	success = CGImageDestinationFinalize( destination );

	if ( optionsDict )
		CFRelease(optionsDict);
	
	require(success, bail);
	
	CFRetain(destinationData);
	ALAssetsLibrary *library = [ALAssetsLibrary new];
	[library writeImageDataToSavedPhotosAlbum:(id)destinationData metadata:metadata completionBlock:^(NSURL *assetURL, NSError *error) {
		if (destinationData)
			CFRelease(destinationData);
	}];
	[library release];


bail:
	if (destinationData)
		CFRelease(destinationData);
	if (destination)
		CFRelease(destination);
	return success;
}
```

We can comment this out, since we don't need to take pictures right now. And if we did later, we could reference this approach in implementing a new one.

The other instance of the `require` `bail` pattern seems to be replaceable by an `if` statement for now.

Additionally, Xcode by default turns on bitcode, which causes an error. So we need to go turn off bitcode in build settings.

Otherwise, it looks like this 4 year old example app builds just fine in modern Xcode, and runs just fine on a modern iOS operating system.

## Up And Running

Looks like we've got the CNN up and running! Next up we integrate the CNN into the camera in our `captureOutput` method.

![voltage2](https://user-images.githubusercontent.com/7946707/32752258-5eda001a-c87d-11e7-99d8-d4698613221f.gif)

## Next Post: [The Camera Part 2](https://nsipplswezey.github.io/2017/11/14/VoltAGE-The-Camera-Part-2.html)

## **Thanks** 
Brian Cline [@standardhuman](https://github.com/standardhuman) and Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for comments and feedback on this post.
