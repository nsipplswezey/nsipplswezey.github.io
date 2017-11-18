---
layout: post
title: VoltAGE The Camera Pt. 2
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

We've got our fork of the react-native camera up and running, modified to console log something every time it processes a frame. In a separate app, we've got a neural net processing a frame every 300ms.

There are two next steps. One step is to start brining the image classifier into the react-native-camera. Another step is to pass a callback as a prop of the react-camera component, that is invoked at the objective-C layer with some data that can be made accessible at the react component level.

Let's tackle that second item first. The existing react-native-camera component includes a barcode reader. A positive CNN classification is similar to a positive barcode read. We can use the existing barcode reader logic as a model for implementing on CNN classifier logic.

At it's simplest, the link between the objective-C layer and the react component layer is just an event emitted from the native layer, a listener on the react-native side for that event, and a react-native-camera component method that will be invoked with native data when the native event is emitted.

Removing other layers of complexity for handling different cases, essentially we have the following parts:

A objective-C method which sends the app event `CameraCNNDetect`, with some data `body:event` which in this case is just an objective-C dictionary(synonymous with a javascript object for most of our purpses). 

```
[self.bridge.eventDispatcher sendAppEventWithName:@"CameraCNNDetect" body:event];
```
And in context (leaving the commented code in for reference)

```
- (void)captureOutput:(AVCaptureOutput *)captureOutput didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection
{
    //CVPixelBufferRef pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);
    //[self runCNNOnFrame:pixelBuffer];
    //NSLog(@"Hello");
    
    
    NSDictionary *event = @{
                            @"data": @"Hello there"
                            };
                            
    
    [self.bridge.eventDispatcher sendAppEventWithName:@"CameraCNNDetect" body:event];
    
}
```

Then in our camera component we can call:

```
NativeAppEventEmitter.addListener('CameraCNNDetect', this._onCNNDetect)
```

And we can define a method `_onCNNDetect`

```
  _onCNNDetect = (data) => {
    if (this.props.onCNNDetect) {
      this.props.onCNNDetect(data)
    }
  };
 ```
 
 And we can pass a prop to our camera 
 ```
 onCNNDetect={(event) => {console.log(event.data)}}`
 ```
 
 In context:
 
 ```
  export default class BadInstagramCloneApp extends Component {
   render() {
     return (
       <View style={styles.container}>
         <Camera
           ref={(cam) => {
             this.camera = cam;
           }}
           style={styles.preview}
           aspect={Camera.constants.Aspect.fill}
           onCNNDetect={(event) => {console.log(event.data)}}
           >
           <Text style={styles.capture} onPress={this.takePicture.bind(this)}>[CAPTURE]</Text>
         </Camera>
       </View>
     );
   }
 ```

Something like that should log "Hello there" to our react-native camera app console. 

## Up And Running

The actual up-and-running detection event callback has an extra log in it:

```
onCNNDetect={(event) => {console.log("positive CNN classification"); console.log(event.data);}}
```

And the `NSLog(@"Hello");` is uncommented.

And here's a nice console log dump from Xcode, showing that our round trip works generally as expected. We can now access data from the obejective-C layer of our react-native camera component in the javascript methods of our camera component.

```
2017-11-14 11:43:19.350951-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.351 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.351266-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.382933-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.385 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.385161-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.385 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.386963-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.418143-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.419 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.419489-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.420 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.419785-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.450307-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.451 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.451525-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.452 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.451815-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.486588-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.489 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.488851-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.489 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.489149-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.516057-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.518 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.517931-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.518 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.518218-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.550753-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.553 [info][tid:com.facebook.react.JavaScript] positive CNN classification
2017-11-14 11:43:19.553483-0800 camera[2767:2110203] positive CNN classification
2017-11-14 11:43:19.554 [info][tid:com.facebook.react.JavaScript] Hello there
2017-11-14 11:43:19.553848-0800 camera[2767:2110203] Hello there
2017-11-14 11:43:19.582577-0800 camera[2767:2110857] Hello
2017-11-14 11:43:19.585 [info][tid:com.facebook.react.JavaScript] positive CNN classification
```

## **Thanks** 
Brian Cline [@standardhuman](https://github.com/standardhuman) and Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for comments and feedback on this post.


