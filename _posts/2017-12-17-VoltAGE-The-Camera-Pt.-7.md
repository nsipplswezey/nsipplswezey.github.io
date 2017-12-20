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

This is a good place to refer to the [commit to react-native-camera that highlights all these changes and additions](https://github.com/nsipplswezey/react-native-camera/commit/620a3a77ca6d9f9177afa6ad9af406ef9443b494). You'll notice that there's an image called `lena.png` included in the commit, which is a traditional computer vision image target for testing.

## Next Steps: Generating and Classifying Bitmaps

Our DeepBelief network operates on bitmaps. In essence we need to pass the relevant data including height, width, pixel count, bytesPerPixel, byteCount and a ByteBuffer of the image pixels a from the image we're classifying to a helper that creates an appropriate image buffer for our classifying method. So the java code here is just generating appropriate inputs, and then working with the outputs from `JPCNNLibrary.INSTANCE.jpcnn_create_image_buffer_from_uint8_dat`and `JPCNNLibrary.INSTANCE.jpcnn_predict`.  We also use pointers and variables by reference as arguments to `jpcnn_classify_image` which then point to results of the classification, and are passed as arguments to our `jpcnn_predict` call, which is the result we're looking for. You can see the DeepBeliefSDK docs for more info about [`jpcnn_classify_image`](https://github.com/jetpacapp/DeepBeliefSDK#jpcnn_classify_image) and [`jpcnn_predict`](https://github.com/jetpacapp/DeepBeliefSDK#jpcnn_predict) and their arguments. In general terms its image bitmap in, generate classifier inpputs from image, classify image, generate prediction value from classification and model, output value between 0-1 for how similar the input image is to the image that generated the prediction model. We also wrap this all in a simple start-stop speed test to see how long it takes for the operation to run on the device.

```
    public static void classifyBitmap(Bitmap bitmap) {
        final int width = bitmap.getWidth();
        final int height = bitmap.getHeight();
        final int pixelCount = (width * height);
        final int bytesPerPixel = 4;
        final int byteCount = (pixelCount * bytesPerPixel);
        ByteBuffer buffer = ByteBuffer.allocate(byteCount);
        bitmap.copyPixelsToBuffer(buffer);
        byte[] pixels = buffer.array();
        Pointer imageHandle = JPCNNLibrary.INSTANCE.jpcnn_create_image_buffer_from_uint8_data(pixels, width, height, 4, (4 * width), 0, 1);

        PointerByReference predictionsValuesRef = new PointerByReference();
        IntByReference predictionsLengthRef = new IntByReference();
        PointerByReference predictionsNamesRef = new PointerByReference();
        IntByReference predictionsNamesLengthRef = new IntByReference();
        long startT = System.currentTimeMillis();
        JPCNNLibrary.INSTANCE.jpcnn_classify_image(
                networkHandle,
                imageHandle,
                0,
                -2,
                predictionsValuesRef,
                predictionsLengthRef,
                predictionsNamesRef,
                predictionsNamesLengthRef);

        JPCNNLibrary.INSTANCE.jpcnn_destroy_image_buffer(imageHandle);

        Pointer predictionsValuesPointer = predictionsValuesRef.getValue();
        final int predictionsLength = predictionsLengthRef.getValue();
        
        //Start trained model prediction
        float trainedPredictionValue = JPCNNLibrary.INSTANCE.jpcnn_predict(predictorHandle, predictionsValuesPointer, predictionsLength);
        android.util.Log.d("ReactNative", "jpcnn_predict() value is " + trainedPredictionValue + ".");
        //End trained model prediction

        long stopT = System.currentTimeMillis();
        float duration = (float) (stopT - startT) / 1000.0f;
        android.util.Log.d("ReactNative", "jpcnn_classify_image() + predict() took " + duration + " seconds.");
        android.util.Log.d("ReactNative", String.format("predictionsLength = %d", predictionsLength));
    }
```

At this point, to prove that we're up and running, we're going to run our classifer and it's prediction method on Lena.png. To do that, we need a method to convert the Lena.png asset into a bitmap, and then pass it as an argument to an invocation of `classifyBitmap`. Our expectation is to see output of a value between 0-1 for some similar `Lena.png` is to our VoltAGE target. We'd expect that `Lena.png` won't score very well, because it's very different than our VoltAGE target.

Here's our method for getting a bitmap from an asset, with some error handling. It's a common utility operation:

```
    public static Bitmap getBitmapFromAsset(AssetManager mgr, String path) {
        InputStream is = null;
        Bitmap bitmap = null;
        try {
            is = mgr.open(path);
            bitmap = BitmapFactory.decodeStream(is);
        } catch (final IOException e) {
            bitmap = null;
            android.util.Log.d("ReactNative", "error in creating bitmap from asset" + e.getMessage());
            android.util.Log.d("ReactNative",  android.util.Log.getStackTraceString(e));

        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException ignored) {

                }
            }
        }
        return bitmap;
    }
```    

With all this in place we'll add the following to our `initDeepBelief` method...

```
        Bitmap lenaBitmap = getBitmapFromAsset(am,"lena.png");

        if(lenaBitmap != null){
            android.util.Log.d("ReactNative", "lena bitmap is not null");
            classifyBitmap(lenaBitmap);
        }
```

...and we'll give it a run and expect to see some logged output.

And great news! Here's our logged output:

```
12-19 21:26:09.085  3773  3773 D DeepBelief: Init deep belief
12-19 21:26:10.893  3773  3814 D RCTCameraModule: setCNNModel called
12-19 21:26:13.207  3773  3773 D ReactNative: networkFile: /data/user/0/com.nextvoltage/files/jetpac.ntwk
12-19 21:26:13.615  3773  3773 D ReactNative: lena bitmap is not null
12-19 21:26:14.871  3773  3773 D ReactNative: jpcnn_predict() value is 0.09134697.
12-19 21:26:14.871  3773  3773 D ReactNative: jpcnn_classify_image() + predict() took 1.226 seconds.
12-19 21:26:14.871  3773  3773 D ReactNative: predictionsLength = 4096
```

With a `jpcnn_predict()` result value of 0.09134697(which is very low) this looks like we're in pretty good shape! `Lena.png` doesn't look much like our `VoltAGE` targets! And it looks like it took a little over a second.

## Next Steps: Frames from the Camera and An Async Task

This is our last section. We now need to run `classifyBitmap` on as many frames as we can, without slowing VoltAGE down to a grinding halt. `react-native-camera` already includes a smart approach to this in their barcode scanner. We can follow that pattern:

```
    public void onPreviewFrame(byte[] data, Camera camera) {
    
        if (RCTCamera.getInstance().isBarcodeScannerEnabled() && !RCTCameraViewFinder.barcodeScannerTaskLock) {
            RCTCameraViewFinder.barcodeScannerTaskLock = true;
            new ReaderAsyncTask(camera, data).execute();
        }
    }
```

The barcode reader checks if the scanner is enabled(which is determined in the ReactNativeJS layer) and checks whether the `barcodeScannerTaskLock` property is true. If the scanner is enabled, and the task isn't locked, we lock the task, so that we only process one frame at a time, and then we pass the relveant data to a new instance of `ReaderAsyncTask` and execute.

Then what's `ReaderAsyncTask` look like? It's actually a bit more complicated than is useful. Essentially it just tries to read if there's a bardcode in the frame. Then it sets `RCTCameraViewFinder.barcodeScannerTaskLock = false;`, which frees us up to grab a new frame and check if it includes a barcode.

We're essentially going to do the same, except we're going to invoke `RCTDeepBelief.classifyBitmap(bmp);` instead of whatever the barcode checker method is. Otherwise everything else is about the same.

Here's our `CNNAsyncTask`:

```
    private class CNNAsyncTask extends AsyncTask<Void, Void, Void> {
        private byte[] imageData;
        private final Camera camera;

        CNNAsyncTask(Camera camera, byte[] imageData) {
            this.camera = camera;
            this.imageData = imageData;
        }

        @Override
        protected Void doInBackground(Void... ignored) {
            if (isCancelled()) {
                return null;
            }

            Size previewSize = camera.getParameters().getPreviewSize();
            YuvImage yuvimage=new YuvImage(imageData, ImageFormat.NV21, previewSize.width, previewSize.height, null);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            yuvimage.compressToJpeg(new Rect(0, 0, previewSize.width, previewSize.height), 80, baos);
            byte[] jdata = baos.toByteArray();
            Bitmap bmp = BitmapFactory.decodeByteArray(jdata, 0, jdata.length);
            android.util.Log.d("ReactNative", "Classifying image placeholder - byte");


            try {
                DeepBelief.classifyBitmap(bmp);
            } catch (Exception e) {
                // meh
                android.util.Log.d("ReactNative", "Error classifying bitmap");
                android.util.Log.d("ReactNative",  android.util.Log.getStackTraceString(e));

            } finally {
                RCTCameraViewFinder.CNNDetectorTaskLock = false;
                return null;
            }
        }
    }
```

With our own CNNDetectorTaskLock...

And here's it's instantiation and execution in the `onPreviewFrame` method:

```
    public void onPreviewFrame(byte[] data, Camera camera) {

        if (!RCTCameraViewFinder.CNNDetectorTaskLock){
            RCTCameraViewFinder.CNNDetectorTaskLock = true;
            new CNNAsyncTask(camera, data).execute();
        }

        if (RCTCamera.getInstance().isBarcodeScannerEnabled() && !RCTCameraViewFinder.barcodeScannerTaskLock) {
            RCTCameraViewFinder.barcodeScannerTaskLock = true;
            new ReaderAsyncTask(camera, data).execute();
        }
    }
```


We also need to add our definition of our TaskLock as a public static property of our RCTCameraFinder...

```
class RCTCameraViewFinder extends TextureView implements TextureView.SurfaceTextureListener, Camera.PreviewCallback {

...

    // CNN Setup
    // concurrency lock for CNN detector to avoid flooding the runtime
    public static volatile boolean CNNDetectorTaskLock = false;

...

}
```

Now our expectation is to have a steady stream of prediction scores logging out of our Android version of VoltAGE!

And here's our [final commit for now](https://github.com/nsipplswezey/react-native-camera/commit/30d5a982d05505a9a83d7ace4bd8704f117084b1) to our fork of react-native-camera!

There are a couple additions needed from here. One is to load our models from out `fetch` in the ReactNativeJS layer. Another is to gracefully handle teardown and set-up when the app goes into background. Right now on both Android and iOS the detection is on by default, so we might want to set it as toggleable. And on Android and iOS we need to implement an event trigger for a positive detection, and figure out a smart algorithm for converting our prediction scores into a reliable UX. All of that are next steps. This should give us a solid technical foundation to get there.

## Finally Up And Running!

At first glance it looks like everything is working great. Our classification and prediction step takes about 2 seconds on a modern Android device. That means in about 30 seconds, we can get predictions on whether the phone is looking at the training target! I've noticed this drift up to 3-4 seconds as the app continues running for a couple minutes. On the iOS side, performance is about 3-4x better, with a classification finishing in about 0.4-0.6 seconds. This is due to using an optimized linnear algebra library that Apple provides called Accelerate. Presumably similar improvements could be made by reworking some of the linnear algebra function calls in the DeepBelief source code with [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) but that's another optimization for later. This is good enough for now! See if you can notice in the log output below when it was that I put the VoltAGE target infront of the camera!

```
12-19 21:48:50.582  5568  5568 D DeepBelief: Init deep belief
12-19 21:48:51.606  5568  5606 D RCTCameraModule: setCNNModel called
12-19 21:48:52.623  5568  5568 D ReactNative: networkFile: /data/user/0/com.nextvoltage/files/jetpac.ntwk
12-19 21:48:52.844  5568  5568 D ReactNative: lena bitmap is not null
12-19 21:48:53.441  5568  5568 D ReactNative: jpcnn_predict() value is 0.09136026.
12-19 21:48:53.441  5568  5568 D ReactNative: jpcnn_classify_image() + predict() took 0.586 seconds.
12-19 21:48:56.056  5568  5589 D ReactNative: jpcnn_predict() value is 0.013354045.
12-19 21:48:56.056  5568  5589 D ReactNative: jpcnn_classify_image() + predict() took 1.567 seconds.
12-19 21:48:59.177  5568  5601 D ReactNative: jpcnn_predict() value is 0.009720933.
12-19 21:48:59.177  5568  5601 D ReactNative: jpcnn_classify_image() + predict() took 2.499 seconds.
12-19 21:49:02.272  5568  5614 D ReactNative: jpcnn_predict() value is 0.01081123.
12-19 21:49:02.272  5568  5614 D ReactNative: jpcnn_classify_image() + predict() took 2.463 seconds.

...

12-19 21:50:33.407  5568  5585 D ReactNative: jpcnn_predict() value is 0.01094358.
12-19 21:50:33.407  5568  5585 D ReactNative: jpcnn_classify_image() + predict() took 2.351 seconds.
12-19 21:50:36.479  5568  5589 D ReactNative: jpcnn_predict() value is 0.010450518.
12-19 21:50:36.479  5568  5589 D ReactNative: jpcnn_classify_image() + predict() took 2.417 seconds.
12-19 21:50:39.600  5568  5601 D ReactNative: jpcnn_predict() value is 0.060899194.
12-19 21:50:39.600  5568  5601 D ReactNative: jpcnn_classify_image() + predict() took 2.481 seconds.
12-19 21:50:42.176  5568  5614 D ReactNative: jpcnn_predict() value is 0.04774032.
12-19 21:50:42.176  5568  5614 D ReactNative: jpcnn_classify_image() + predict() took 2.028 seconds.
12-19 21:50:44.625  5568  5585 D ReactNative: jpcnn_predict() value is 0.049463954.
12-19 21:50:44.626  5568  5585 D ReactNative: jpcnn_classify_image() + predict() took 1.959 seconds.
12-19 21:50:47.647  5568  5589 D ReactNative: jpcnn_predict() value is 0.009187927.
12-19 21:50:47.647  5568  5589 D ReactNative: jpcnn_classify_image() + predict() took 2.462 seconds.
12-19 21:50:50.331  5568  5601 D ReactNative: jpcnn_predict() value is 0.18211325.
12-19 21:50:50.331  5568  5601 D ReactNative: jpcnn_classify_image() + predict() took 2.193 seconds.
12-19 21:50:52.868  5568  5614 D ReactNative: jpcnn_predict() value is 0.039335493.
12-19 21:50:52.868  5568  5614 D ReactNative: jpcnn_classify_image() + predict() took 2.03 seconds.
12-19 21:50:55.237  5568  5585 D ReactNative: jpcnn_predict() value is 0.831542.
12-19 21:50:55.237  5568  5585 D ReactNative: jpcnn_classify_image() + predict() took 1.939 seconds.
12-19 21:50:57.641  5568  5589 D ReactNative: jpcnn_predict() value is 0.8141885.
12-19 21:50:57.641  5568  5589 D ReactNative: jpcnn_classify_image() + predict() took 1.985 seconds.
12-19 21:51:00.292  5568  5601 D ReactNative: jpcnn_predict() value is 0.73880213.
12-19 21:51:00.292  5568  5601 D ReactNative: jpcnn_classify_image() + predict() took 2.181 seconds.
12-19 21:51:02.813  5568  5614 D ReactNative: jpcnn_predict() value is 0.7692477.
12-19 21:51:02.813  5568  5614 D ReactNative: jpcnn_classify_image() + predict() took 2.073 seconds.
12-19 21:51:05.167  5568  5585 D ReactNative: jpcnn_predict() value is 0.7586093.
12-19 21:51:05.167  5568  5585 D ReactNative: jpcnn_classify_image() + predict() took 1.93 seconds.
12-19 21:51:07.415  5568  5589 D ReactNative: jpcnn_predict() value is 0.817426.
12-19 21:51:07.415  5568  5589 D ReactNative: jpcnn_classify_image() + predict() took 1.772 seconds.
12-19 21:51:10.069  5568  5601 D ReactNative: jpcnn_predict() value is 0.0034205674.
12-19 21:51:10.069  5568  5601 D ReactNative: jpcnn_classify_image() + predict() took 2.167 seconds.

```




