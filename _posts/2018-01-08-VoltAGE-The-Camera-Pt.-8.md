---
layout: post
title: VoltAGE The Camera Pt. 8
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera. 

In simpler terms, you point your mobile device camera at an object you want VoltAGE to recognize. With your camera pointed at the object, the stream of images your phone is capturing get sent to a machine learning algorithm. A bunch of computation happens based on the images your camera is feeding to the algorithm, and after about 20 seconds of pointing your camera, that algorithm outputs about 1.2mb of data. We can then pass that 1.2mb of data to another machine learning algorithm, and then again start sending the stream of images from your mobile camera to *that* algorithm, which uses the 1.2mb data the first algorithm generated to tell you on a confidence scale of 0.00 to 1.00 how much the thing that your camera is pointed might be the thing it was pointed at during the 20 seconds spent creating the 1.2mb of data. 

The **model parameters** are the 1.2mb of data. The **convolutional-neural-net(CNN)** is the algorithm that combines the images from your front facing camera and that 1.2mb of data and outputs a number between 0.00 and 1.00. That number between 0.00 and 1.00 can be used as a **detection score** for the physical object using the front-facing camera.

The idea is to use all this to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

In our previoues post [The Camera Part 7](https://nsipplswezey.github.io/2017/12/17/VoltAGE-The-Camera-Pt.-7.html), we built out the Android version of our image detector which processes frames from the front facing camera. It's about 4-5x slower than iOS, but it's workable, processing an classifying a frame ever 2-3 seconds.

The iOS side is still jsut a touch ahead of the Android side in one way. The prediction model on iOS is fetched from the server. We want to make sure the same is true for our Android react-native component.

We already have a component method that accepts the model string fetched from the server and makes it available in the Android version of our camera module.

```
    @ReactMethod
    public void setCNNModel(final String model) {
        Log.d(TAG, "setCNNModel called");
    }
```

Currently in our `DeepBelief` class, there's a couple steps to handle the loading of our predictor file, and the application of our predictor to our on device image classification.

It helps to look at this process in reverse from the end point.

Ultimately we're trying to create a `trainedPredictionValue` which is the output of an invocation of `jpcnn_predict`:

```
float trainedPredictionValue = JPCNNLibrary.INSTANCE.jpcnn_predict(predictorHandle, predictionsValuesPointer, predictionsLength);
```

The important piece here is that we pass a `predictorHandle` to this function invocation. `predictorHandle` is a pointer result of the `jpcnn_load_predictor` method.

```
static Pointer predictorHandle = null;
predictorHandle = JPCNNLibrary.INSTANCE.jpcnn_load_predictor(predictorFile);
```

And the way we make sure our `predictorFile` string is actually a real file path that `jpcnn_load_predictor` can use to load a correctly formated predictor, is with a helper method called `copyAsset`.

```
        String predictorFileName = "VoltAGE_2_predictor.txt";
        String predictorFile = dataDir + "/" + predictorFileName;
        copyAsset(am, predictorFileName, predictorFile);
```

`copyAsset` essentially copies a bundled file `Voltage_2_predictor.txt` to our apps `data` directory.

So what we need to do is take the string that contains the data for model that's fetched from our API server, write it to the data directory, pass a valid path to that file to `jpcnn_load_predictor`, and assign the result of `jpcnn_load_predictor` to our `predictorHandle` pointer, which is then passed as an argument to `jpcnn_predict`.

And for now, we want to do all that inside of our `setCNNModel` method.

This happened in two steps.

The first step was realizing that the Android APIs for file writing are dependent on a `context` object. And that the react-native component code appears to have a different `context` than the one we need. To handle this, we'll use a public static property on our `DeepBelief` class to maintain a reference to the necessary context that we need to interact with in our `setCNNModel` method.

Essentially that looks like defining that public static property, and then assigning it to the correct context when we create our `DeepBelief` activity.

That's `public static Context ctx = null;` followed by `ctx = this;` which then allows us to reference `DeepBelief.ctx` in any of our other Android classes.

```
public class DeepBelief extends Activity {

    public static final String TAG = "DeepBelief";
    public static Context ctx = null;
    static Pointer networkHandle = null;
    static Pointer predictorHandle = null;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ctx = this;

        initDeepBelief();
    }
```

The necessary references to `DeepBelief.ctx` in our `setCNNModel` method, where we use the Android file system API to write our string to a .txt file look like:

```
    @ReactMethod
    public void setCNNModel(final String model) {
        Log.d(TAG, "setCNNModel called");

        //write string to file
        String predictorFileName = "VoltAGE_predictor_from_api.txt"; //define the file name
        FileOutputStream outputStream; //create an outputStream

        try {
          outputStream = DeepBelief.ctx.openFileOutput(predictorFileName, DeepBelief.ctx.MODE_PRIVATE); //open a file output
          outputStream.write(model.getBytes()); //get the model string (fetched from server) as bytes and write them
          outputStream.close(); //close the stream
        } catch (Exception e) {
          e.printStackTrace(); //handle errors
        }

        //get file path
        String dataDir = DeepBelief.ctx.getFilesDir().getAbsolutePath(); //get path to android files
        Log.d(TAG, dataDir);
        String predictorFilePath = dataDir + "/" + predictorFileName; //append "VoltAGE_predictor_from_api.txt" to the path
        Log.d(TAG, predictorFilePath);

        //if file exists, DeepBelief.setPredictor(predictorFile);
        File file = new File(predictorFilePath);
        if(file.exists()){
          Log.d(TAG, "File exists. Setting predictor");
          DeepBelief.setPredictor(predictorFilePath);  //set predictor

        }

    }
```

At the end of this method, is an invocation of a `DeepBelief.setPredictor` method which is a simple wrapper around `jpcnn_load_predictor`.

```
    public static void setPredictor(String predictorFilePath){
      predictorHandle = JPCNNLibrary.INSTANCE.jpcnn_load_predictor(predictorFilePath);
    }
```


### Up and Running

Currently this seems to work. Though it looks like there's an occasional crash on first load from the bundle. Which leaves me considering whether there's potentially some useful error handling to be done around waiting for the model to be fetched from the server. We'll tackle that next.

Check out the [commit](https://github.com/nsipplswezey/react-native-camera/commit/30c01f797f460a20d762ba83c5d03bc2b7f2e2f8) to the fork of `react-native-camera` with these changes!

## Previous Post: [The Camera Part 7](https://nsipplswezey.github.io/2017/12/17/VoltAGE-The-Camera-Pt.-7.html) 

## Reference Links

[https://developer.android.com/training/data-storage/files.html](https://developer.android.com/training/data-storage/files.html)

## **Thanks** 
Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for comments and feedback on this post.


