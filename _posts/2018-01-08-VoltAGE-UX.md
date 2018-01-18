---
layout: post
title: VoltAGE The UX
---


## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera. 

In simpler terms, you point your mobile device camera at an object you want VoltAGE to recognize. With your camera pointed at the object, the stream of images your phone is capturing get sent to a machine learning algorithm. A bunch of computation happens based on the images your camera is feeding to the algorithm, and after about 20 seconds of pointing your camera, that algorithm outputs about 1.2mb of data. We can then pass that 1.2mb of data to another machine learning algorithm, and then again start sending the stream of images from your mobile camera to *that* algorithm, which uses the 1.2mb data the first algorithm generated to tell you on a confidence scale of 0.00 to 1.00 how much the thing that your camera is pointed might be the thing it was pointed at during the 20 seconds spent creating the 1.2mb of data. 

The **model parameters** are the 1.2mb of data. The **convolutional-neural-net(CNN)** is the algorithm that combines the images from your front facing camera and that 1.2mb of data and outputs a number between 0.00 and 1.00. That number between 0.00 and 1.00 can be used as a **detection score** for the physical object using the front-facing camera.

The idea is to use all this to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

Our previous post [The Camera Part 8](https://nsipplswezey.github.io/2018/01/08/VoltAGE-The-Camera-Pt.-8.html) capped off development of the camera component for now. We've reached our milestone of classifying frames with a CNN that is paramaterized based on data fetched from our server on both Android and iOS.

Essentially we've reached VoltAGE 0.0.2. You can visit the [VoltAGE README.md](https://github.com/nsipplswezey/VoltAGE/blob/master/README.md) to see what was entailed in that milestone.

Here's where we're headed next with VoltAGE 0.0.3:

- [ ] gift interaction between android and iOS device

- [ ] beta channel and app store deployment

- [ ] model training workflow for new objects

- [ ] clean coherent UX

This blog post series is going to focus on that 4th item: clean and coherent UX.

"clean and coherent UX" is actually a relatively tall –and vague– task item. It's also a task item that spans outside of engineering. User experience is a design problem, that interfaces with engineering implementation.

What does "clean and coherent" even mean? In this case I'm going to use these words as a rubric. Clean means nothing is included besides what's necessary. Coherent means that each part of the experience is attached in an intentional way, that a user might be able to notice if prompted. And lastly, I think it's important to state that we want to get something good up and running, rather than something perfect that never actually exists.

We actually already have an existing UI on iOS. A positive detection programatically triggers a "swipe right" gesture through a carousel.

![voltage](https://user-images.githubusercontent.com/7946707/32696574-70902de6-c730-11e7-8027-b840e8bc7860.gif)

Earlier this week I had a chance to show a friend the existing UX, and took a second to sketch out ideas around a more "clean" and "coherent" iteration.

A couple insights stood out from the conversation, and we can reference the above video for context.

First, my friend pointed out that that initial bouncing icon is a great intuitive call to action. They mentioned as well that a "lock-key" or "home button" style "click" metaphore would be coherent with the bouncing call to action.

Second, they highlighted that 5 screens of instructions are way to many. I agree wholeheartedly.

Third, they pointed out that a "swipe" metaphor is inconsistent with what amounts to a z-axis physical alignment. Essentially, the physical VoltAGE item is aligned with the camera, which is aligned with the image overlays. If we're using alignment of 3 layers –reality, camera, icon– then coherence would suggest maintaining alignment based UX.

I think that's exactly enough for the next UX iteration. Keep the bouncing call to action. Embelish it with a "click" if possible. Reduce the number of screens. Replace the y-axis transitions with z-axis transitions.

Let's give it a shot!

Of those task items, the click is a nuance change, using fewer screens is a reduction, and the bouncing call to action is a keep. Replacing the y-axis transition with a z-axis transition seems to be the most important to get started with.

While we're at it, we want a solution that works both of Android and iOS. And right now our UI doesn't work for Android. Because our implementation is based on extending an activity, which at the time was the simplest thing to do, but now up further investigation, interupts the main app activity.

So we need to quick refactor. Previous we had our `DeepBelief` class extending the Android `Activity` class. This was because our base code example was built as an `Acitivity` and we were able to get our app context from an `Activity` method for fetching and copying CNN model and network files. This approach breaks our Android app, because it interrupts the ract-native main activity. It turns out react-native provides it's own app context, which our camera exposes in a method called `RCTCameraModule.getReactContextSingleton();` which we can use for our file copying needs.

Previously we used `Activity` `onCreate` event handler to assign our app context to the `ctx` variable before initializing our `DeepBelief` classifier.

```
     @Override
     public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         ctx = this;
    
         initDeepBelief();
     }
 ```
 
 We can get rid of this whole method, and just assign `ctx` to the result of `RCTCameraModule.getReactContextSingleton();` in `initDeepBelief` and do all our file copying.
 
 ```
     static void initDeepBelief() {

        android.util.Log.d("DeepBelief", "Init deep belief");

        ReactContext reactContext = RCTCameraModule.getReactContextSingleton();
        ctx = reactContext;

        AssetManager am = ctx.getAssets();
        String baseFileName = "jetpac.ntwk";
        String dataDir = ctx.getFilesDir().getAbsolutePath();
        String networkFile = dataDir + "/" + baseFileName;
        copyAsset(am, baseFileName, networkFile);
 ```

Check out the commit here: [https://github.com/nsipplswezey/react-native-camera/commit/9d357fbc6c920d57dee0a415f2d727712e4fbc18](https://github.com/nsipplswezey/react-native-camera/commit/9d357fbc6c920d57dee0a415f2d727712e4fbc18)

That's enough to get our Animation working on Android.

Additionally, I also made changs to our `TargetOverlay` component which holds our boncing VoltAGE target call to action as in the gif above.

There's a series of relevant quirks here, and it's unclear if work-arounds are necessary for all of them. What I do know is that it works as it is currently, and I left my self comments for where the guidance around the fixes came from, which I'll deal with later, probably while referencing this blog post.

The quirks are as follows:

1. On Android the `backgroundColor` style property for an `Animated.Image` needs to be transparent.

2. Using native drivers seems to work better, so the `useNativeDriver` prop for our `Animated.Image` component should be set to `true`.

3. If modifying the `transform` style property, which we are to create that bouncing effect, the `perspective` style property of our `transform` should be set to 1000.

4. Additionally, initial values of `0` for animated properties can cause issue, so starting at `0.01` is ideal.

5. Finally, to loop our animation infinitely, we previously were chaining our animation as a callback to be run after each animation cycle finished. We've refactored that to use the Animated APIs `.loop` method.

Here's what the component looks like now, with a couple extraneous bits for handling new CNN detetion values, and waking from background state, commented out for now:

```
export default class TargetOverlay extends Component{
  constructor(props: any){
    super(props);
    this.state = {
      bounceValue: new Animated.Value(1.5), //https://stackoverflow.com/questions/47278781/react-native-animation-not-working-properly-on-android
      wiggleValue: new Animated.Value(0.01), //https://stackoverflow.com/questions/47278781/react-native-animation-not-working-properly-on-android
      bouncing: false,
    };
  }

  render(){
    console.log('using native driver')
    return(
	    <Animated.Image
	      source={require('../Assets/Target.png')}
        useNativeDriver= {true}
	      style={
	        {flex: 0,
		       width: 500,
		       height: 500,
           backgroundColor: "transparent", //https://github.com/facebook/react-native/issues/13550
		       transform: [
		         {scale: this.state.bounceValue},
             {perspective: 1000}, //https://facebook.github.io/react-native/docs/animations.html#bear-in-mind
		       ]}
	      }
	      resizeMode={'contain'}/>
    )}

  componentDidMount() {
    //this._bounce();

    Animated.loop(
      Animated.spring(
        this.state.bounceValue,
        {
          toValue: 0.8,
  	      friction: 1,
        }
      )).start();

    //AppState.addEventListener('change',this._handleIOSstateChange.bind(this));
    //NativeAppEventEmitter.addListener('CameraCNNUpdate', this._handleCNNUpdate.bind(this));
  }
  
  ...
```



See here: [https://github.com/nsipplswezey/VoltAGE/commit/cbff06e28437b3721f8d18f01377590ed455d7c3](https://github.com/nsipplswezey/VoltAGE/commit/cbff06e28437b3721f8d18f01377590ed455d7c3)

### Up and Running

It appears we're now up and running with a single stack UI on Android and iOS. Next up we'll add transitions to the animations.


## Reference Links


## **Thanks** 
Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) and [@christopher-hague](https://github.com/christopher-hague) for comments and feedback on this post.


