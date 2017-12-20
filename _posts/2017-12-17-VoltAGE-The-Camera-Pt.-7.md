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
This is going to be a longer post(or series of posts) in that what was covered for iOS in 5-6 smaller chunks is going to get a faster and looser treatment here. Additionally, the Android side of react-native-camera is factored into more classes, and things appear to be a bit more verbose on the Java side. All this is to say, if you're reading this closely to follow along in technical detail, you're more likely to be getting a quick gist of it all by scanning the code snippets in the posts. I'm going to throw in links to the actual commits to our fork of react-native-camera and VoltAGE, as a more enduring reference technical reference in context. Additionally, you can assume that I won't be linking any breaking commits. Additionally, I think it's worth noting that these blog posts represent a cleaned up more linnear approach to building out these features, which emerged from a much more typical non-linnear discovery, coding and debugging process. 


## Our First Interaction With Android
Since we haven't touched the Android side of our application, let's get it running with a fresh clone of VoltAGE. At this point in time, VoltAGE is at [commit-40](https://github.com/nsipplswezey/VoltAGE/tree/2e7b2763bd5c3bb52fc90305c65b98e59b911ab6) and our fork of react-native-camera is at [commit-443](https://github.com/nsipplswezey/react-native-camera/tree/ecd0a232e33108b1211d0dff5fe128a039db06d4). 

```
git clone https://github.com/nsipplswezey/VoltAGE.git
cd VoltAGE/nextVoltAGE
npm install
react-native run-android
```

The first error we run into is that our fork of react-native-camera doesn't have a `setCNNModel` method, so it throws an error on `      this.camera.setCNNModel(detectorParamString);`.

This is a nice easy start into working our way from our react-native app down to our react-native-camera android module layer. The central class in our Android module is `RCTCameraModule`. Our Android classes extend and implement other classes. Extending and implementing essentially means that the class your working with gets access to specific methods and properties based on what classes it implements and extends.

```
public class RCTCameraModule extends ReactContextBaseJavaModule
    implements MediaRecorder.OnInfoListener, MediaRecorder.OnErrorListener, LifecycleEventListener { 
    
    ...a bunch of methods...
    
    }
```

In implementing `ReactContextBaseJavaModule` the `RCTCameraModule` class allows us to implement react-native methods that we can call from the JS layer of our application using a `@ReactMethod` annotation. Which means something like the below:

```
public class RCTCameraModule extends ReactContextBaseJavaModule
    implements MediaRecorder.OnInfoListener, MediaRecorder.OnErrorListener, LifecycleEventListener { 
    
    ...a bunch of methods...
    
    @ReactMethod
    public void setCNNModel(final String model) {
        Log.d(TAG, "setCNNModel called");
    }
    
    ...a bunch more methods...
    
}
```

Allows us to call `this.camera.setCNNModel(detectorParamString);` in 

```
  setCNNModel() {
    fetch('https://evening-falls-93288.herokuapp.com/v1/detectors/1')
    .then((response) => response.json())
    .then((responseJson) => {
      let detectorParamString = responseJson.data[0].parameters;
      this.camera.setCNNModel(detectorParamString);
    })
    .catch((error) => {
      console.error(error);
    })
  }
```

without any error. 

We won't worry about using fetched models until later.

## Adding a Class

Next up, we're going to add a class to the Android side of react-native-camera to encapsulate our convolutional neural network processing. That means that our `RCTCameraModule` or any other existing classes in react-native-camera will just call methods from our `DeepBelief` class. 

At it's simplest it looks like:

```
package com.lwansbrough.RCTCamera;

/**
 * Created by nsipplswezey on 12/19/17.
 */

public class DeepBelief {
    public static final String TAG = "DeepBelief";

}
```

Which we can interact with in our other classes. Ultimately some method in our `DeepBelief` class is going to operate on frames taken from the front facing camera. Those frames are going to come from the `RCTCameraViewFinder` class, specifically from the [`onPreviewFrame`](https://github.com/nsipplswezey/react-native-camera/blob/ecd0a232e33108b1211d0dff5fe128a039db06d4/android/src/main/java/com/lwansbrough/RCTCamera/RCTCameraViewFinder.java#L287) method.

Let's make sure our `DeepBelief` class can interact with `onPreviewFrame` in a way that seems reasonable by logging our `DeepBelief` static `TAG` string variable on every preview frame. Notice that react-native-camera uses this same `onPreviewFrame` method to do some work related to a barcode scanner.

```
    public void onPreviewFrame(byte[] data, Camera camera) {
    
    android.util.Log.v("RCTCamera", "DeepBelief class tag " + DeepBelief.TAG );
    
        if (RCTCamera.getInstance().isBarcodeScannerEnabled() && !RCTCameraViewFinder.barcodeScannerTaskLock) {
            RCTCameraViewFinder.barcodeScannerTaskLock = true;
            new ReaderAsyncTask(camera, data).execute();
        }
    }
```

And sure enough we get a stream of logs

```
12-19 10:56:52.144 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.170 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.236 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.284 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.346 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.383 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.461 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.527 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
12-19 10:56:52.611 14879 14879 D RCTCamera: DeepBelief class tag DeepBelief
```

## Logging on Android

To see our log output, we use `logcat` from the Android Debug Bridge `adb` command line utility. Essentially when code is running on our Android device, the device generates a ton of logs, all with different tags –like `RCTCameraModule` or `ReactNative`– and all with different log levels, like `e` for error, or `d` for debug. We can use `logcat` to filter out only the logs we want to see, in which case we want to see log messages at the debug `d` level for `RCTCameraModule` `ReactNative` `RCTCamera` and `ReactNativeJS` which is our javascript layer `console.log`. We also want to silence all other tags which is where the `*:S` comes from.

```
adb logcat RCTCameraModule:D ReactNative:D RCTCamera:D ReactNativeJS:D *:S
```

## Next Steps: Our DeepBelief Class

This would typically be a good stopping point. We defined our scope, we got up and running with android, we implemented a simple react-native method in the Android layer of our application, we created a new class, we found the right method to hook into for processing frames, we successfully logged. Having said that, I'm going to press on to our next steps towards getting our client side DeepBelief network running in Android. From here things are likely to get more complex, and perhaps raise questions. That's ok. One of the benefits of focusing on getting something up and running, is that it helps you understand what questions to ask.

Our DeepBelief class is going to extend the Android Activity class. Activities are typically containers for UI interactions like modals or screens or buttons or lists. What we're going to do here is build our DeepBelief class as an activity with no display. When our camera starts, we'll fire up our DeepBelief activity, and start feeding it frames. But because there's no display for the activity, we'll still see the front facing camera in our preview layer in our App view. This aproach could likely be improved upon, but it's ok for now.

Android Activities provide `onCreate` `onPause` and `onResume` life methods that we can override. That is when the activity is created, we can run some code. Same for when it's paused or resumed.

Let's implement `onCreate` in our DeepBelief class. There's going to be some initial setup necessary for our DeepBelief network, so let's have our `onCreate` method invoke a `initDeepBelief` method, and have it log a message for now.

```
public class DeepBelief extends Activity {
    public static final String TAG = "DeepBelief";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        initDeepBelief();
        
    }

    static void initDeepBelief() {

        android.util.Log.d("DeepBelief", "Initialized DeepBelief");

    }

}
```

Our `DeepBelief` class now extends Activity, and overrides the provided `onCreate` method. If you want to learn more about `super.onCreate(savedInstanceState);` you can check out [this stack overflow post](https://stackoverflow.com/questions/14671897/super-oncreatesavedinstancestate) for a quick rundown.

Now that we have our activity, we need to initialize our activity.

This piece also leapfrogs over some complexity. But essentially in our RCTCameraViewFinder constructor we can programatically trigger our `DeepBelief` activity using the following pattern, with the rest of react-native-camera RCTCameraViewFinder provided for context:

```
    public RCTCameraViewFinder(Context context, int type) {
        super(context);
        this.setSurfaceTextureListener(this);
        this._cameraType = type;
        this.initBarcodeReader(RCTCamera.getInstance().getBarCodeTypes());

        //start Deep Belief CNN activity here...
        Intent deepBeliefIntent = new Intent(context, DeepBelief.class);
        context.startActivity(deepBeliefIntent);

    }
```

Whenever extending an existing module, it helps to follow the patterns of the existing codebase, as much as possible. This is a case where the initilization of our DeepBelief activity happens in the same place that the initialization of the react-native-camera bar code reader also occurs. The bar code reader, however, isn't implemented as a separate class, nor is it implemented as an activity. Refactoring DeepBelief to more consistently follow the pattern of the bar code reader might be worthwile to do later.

Given these additions to the code, we exect the DeepBelief activity to be started upon the invocation of the `RCTCameraViewFinder` constructor, which should then invoke the `onCreate` method of our `DeepBelief` activity, which should then invoke the `initDeepBelief` method of our `DeepBelief` class, which should then log "Initialized DeepBelief" to our console.

Let's give it a try!

We get an error!

![image](https://user-images.githubusercontent.com/7946707/34178208-320bc352-e4bb-11e7-91fa-3cc8c404d24d.png)

On Android devices, activities need to be declared in the AndroidManifest.xml file. This is where we'll indicate that the DeepBelief activity shouldn't display any UI. This is also a place where the way we implemented DeepBelief in the Android side of react-native-camera begins to have some consequences. The AndroidManifest of VoltAGE itself now needs to be edited to work with react-native-camera. This highlights a perk to refactoring our implementation using another approach that isn't built on extending Activity, especially if I were to want to create an upstream pull request to react-native-camera. All of this, however, is fine for now.

The tag we need to add to our manifest looks like 
```
<activity android:name="com.lwansbrough.RCTCamera.DeepBelief" android:theme="@android:style/Theme.NoDisplay" />
```

And here it is in context of the VoltAGE AndroidManifest.xml.

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.nextvoltage"
    android:versionCode="1"
    android:versionName="1.0">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>

    <uses-sdk
        android:minSdkVersion="16"
        android:targetSdkVersion="22" />

    <application
      android:name=".MainApplication"
      android:allowBackup="true"
      android:label="@string/app_name"
      android:icon="@mipmap/ic_launcher"
      android:theme="@style/AppTheme">
      <activity
        android:name=".MainActivity"
        android:label="@string/app_name"
        android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
        android:windowSoftInputMode="adjustResize">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
      </activity>
        
      <activity android:name="com.lwansbrough.RCTCamera.DeepBelief" android:theme="@android:style/Theme.NoDisplay" />
      <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
        

    </application>

</manifest>
```

## Up And Running

And here's our log as proof that it worked with our "Initilized DeepBelief" from our DeepBelief class, followed by the log that fires on each invocation of `onPreviewFrame`  and our log from our react-native `setCNNModel` function call!

```
12-19 12:58:23.606 18672 18708 I ReactNativeJS: Running application "nextVoltAGE" with appParams: {"rootTag":1}. __DEV__ === true, development-level warning are ON, performance optimizations are OFF
12-19 12:58:24.591 18672 18672 D DeepBelief: Initialized DeepBelief
12-19 12:58:25.262 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.304 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.337 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.392 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.501 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.566 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.624 18672 18709 D RCTCameraModule: setCNNModel called
12-19 12:58:25.700 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.766 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.844 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.915 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:25.982 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:26.044 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
12-19 12:58:26.114 18672 18672 D RCTCamera: DeepBelief class tag DeepBelief
```

[VoltAGE commit-41](https://github.com/nsipplswezey/VoltAGE/commit/374700bdbc5685856c3fb82eab99a0abd0bfefe9) and [react-native-camera commit-445](https://github.com/nsipplswezey/react-native-camera/commit/64212b74e3d814de24136522c120a76f7a120212) represent the working state of VoltAGE up to this point.

## Next Steps: Assets and C libraries

This might also be a natural pause or conclusion point. We extended our DeepBelief class into an activity, created a placeholder method for initialization, and then programatically started our activity at what seems like a reasonable location in the existing react-native-camera codebase, and added our activity to the VoltAGE AndroidManifest, and everything works!

Let's keep going.

The DeepBelief SDK highlights quirks in the Android implementation. Mostly that at the time the SDK was released, they needed to implement some workarounds with regards to copying files in Android. So they implemented two helper methods for copying assets. One is called `copyAssets` and the other is `copyFiles`. Let's add those methods to our DeepBelief class. We won't go into these in detail, but generally they're just helpers to read from one directory in our bundle, and write to another. I could take a deeper look into this to figure out a way to potentially improve it, or assess if it's even necessary, but that can also wait for some later date.

```
package com.lwansbrough.RCTCamera;

import android.app.Activity;
import android.content.res.AssetManager;
import android.os.Bundle;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

/**
 * Created by nsipplswezey on 12/19/17.
 */

public class DeepBelief extends Activity {
    public static final String TAG = "DeepBelief";

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        initDeepBelief();
    }

    static void initDeepBelief() {

        android.util.Log.d("DeepBelief", "Init deef belief");

    }

    private static boolean copyAsset(AssetManager assetManager,
                                     String fromAssetPath, String toPath) {
        InputStream in = null;
        OutputStream out = null;
        try {
            in = assetManager.open(fromAssetPath);
            new File(toPath).createNewFile();
            out = new FileOutputStream(toPath);
            copyFile(in, out);
            in.close();
            in = null;
            out.flush();
            out.close();
            out = null;
            return true;
        } catch(Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    private static void copyFile(InputStream in, OutputStream out) throws IOException {
        byte[] buffer = new byte[1024];
        int read;
        while((read = in.read(buffer)) != -1){
            out.write(buffer, 0, read);
        }
    }



}

```

What matters more is how we use them. When we initialize DeepBelief, we have to copy our trained network, and for now, we also copy the .txt file that contains the parameters of the model we trained on our VoltAGE target. Then we need to pass the path of those files to our machine learning methods that can use them to classify frames from the front facing camera. Those files are `jetpac.ntwk` and `VoltAGE_2_predictor.txt`.

The two machine learning methods in our library that we need to set up on initialization are `JPCNNLibrary.INSTANCE.jpcnn_create_network(networkFile);` and `predictorHandle = JPCNNLibrary.INSTANCE.jpcnn_load_predictor(predictorFile);`. Additionally we need to use pointers to retain a references to these two function calls, so we also define to empty pointers for that purpose `static Pointer networkHandle = null;` and `static Pointer predictorHandle = null;`

Both of those methods exist in a pair of C libraries, `libjnidispatch.so` and `libjpcnn.so` which both need to be added to our fork of react-native-camera. Android studio helps with this, because external C libaries that interface with Android java applications need to be added to a specific directory called `jniLibs` and specifically into subdirectories called `armeabi` `armeabi-v7a` and `armeabi-x86` for what I understand to be different architectures of phones.

In addition to these `libjnidispatch.so` and `libjpcnn.so` we also need to add the `deepbelief.jar` to our project. Finally, we also create a context `static Context ctx;` variable, which allows us to access the assets associated with the project with `AssetManager am = ctx.getAssets();`.

To put all the pieces together so far, our `onCreate()` and `initDeepBelief()` method looks like the following:

```
import com.jetpac.deepbelief.DeepBelief.JPCNNLibrary;
import com.sun.jna.Pointer;

public class DeepBelief extends Activity {
    public static final String TAG = "DeepBelief";

    static Context ctx;
    static Pointer networkHandle = null;
    static Pointer predictorHandle = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ctx = this;

        initDeepBelief();
    }

    static void initDeepBelief() {

        android.util.Log.d("DeepBelief", "Init deef belief");


        AssetManager am = ctx.getAssets();
        String baseFileName = "jetpac.ntwk";
        String dataDir = ctx.getFilesDir().getAbsolutePath();
        String networkFile = dataDir + "/" + baseFileName;
        copyAsset(am, baseFileName, networkFile);
        android.util.Log.d("ReactNative", "networkFile: " + networkFile);
        networkHandle = JPCNNLibrary.INSTANCE.jpcnn_create_network(networkFile);

        //TODO: Load predictor and predict
        //Use the same technique to set of the predictor
        //Replace the classify image call with the predict call
        String predictorFileName = "VoltAGE_2_predictor.txt";
        String predictorFile = dataDir + "/" + predictorFileName;
        copyAsset(am, predictorFileName, predictorFile);
        predictorHandle = JPCNNLibrary.INSTANCE.jpcnn_load_predictor(predictorFile);

    }
```

This is a good place to refer to the commits that highlight all these changes in a working form.



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




