---
layout: post
title: VoltAGE The Camera Pt. 4
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera. 

In simpler terms, you point your mobile device camera at an object you want VoltAGE to recognize. With your camera pointed at the object, the stream of images your phone is capturing get sent to a machine learning algorithm. A bunch of computation happens based on the images your camera is feeding to the algorithm, and after about 20 seconds of pointing your camera, that algorithm outputs about 1.2mb of data. We can then pass that 1.2mb of data to another machine learning algorithm, and then again start sending the stream of images from your mobile camera to *that* algorithm, which uses the 1.2mb data the first algorithm generated to tell you on a confidence scale of 0.00 to 1.00 how much the thing that your camera is pointed might be the thing it was pointed at during the 20 seconds spent creating the 1.2mb of data. 

The **model parameters** are the 1.2mb of data. The **convolutional-neural-net(CNN)** is the algorithm that combines the images from your front facing camera and that 1.2mb of data and outputs a number between 0.00 and 1.00. That number between 0.00 and 1.00 can be used as a **detection score** for the physical object using the front-facing camera.

The idea is to use all this to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

I'd like to be able to get the model data from the API and pass it into the obj-C camera layer. To do this we can create a method 

In our fork of react-native-camera we do the following in our obj-c camera manager:


```
//RCTCameraManager.h
- (void)setModel:(NSString *)model;
```


```
//RCTCameraManager.m
RCT_EXPORT_METHOD(setModel:(NSString *)model) {
    NSLog(@"%@", model);
}
```

And the following in our index.js file for our react-native-camera component.

```
//react-native-camera/index.js
export default class Camera extends Component {

...

  setModel(model) {
    CameraManager.setModel(model);
    return;
  }
 
...

}  
```

Then in the actual app that implements the camera:

```
export default class VoltAGE extends Component {

   render() {
     return (
       <View style={styles.container}>
         <Camera
           ref={(cam) => {
             this.camera = cam;
           }}
           style={styles.preview}
           aspect={Camera.constants.Aspect.fill}
           onCNNDetect={(event) => {console.log("positive CNN classification"); console.log(event.data);}}
           >
           <Text style={styles.capture} onPress={this.setModel.bind(this)}>[SET MODEL]</Text>
         </Camera>
       </View>
     );
   }

   setModel() {

     this.camera.setModel("Setting model")

    }
    
```

To extend that basic implementation with fetched data:

```
   setModel() {

     fetch('https://evening-falls-93288.herokuapp.com/v1/detectors/1')
       .then((response) => response.json())
       .then((responseJson) => {
         let detectorParamString = responseJson.data[0].parameters;
         this.camera.setModel(detectorParamString);
       })
       .catch((error) => {
         console.error(error);
       })

    }
```


## Up And Running

Sure enough, on the obj-c side we get out fetched data logged to the console! Scroll to the right in the text box below to see a sample of the large part of our data which is a series of space delimited key value pairs of integers and floating point numbers.

```
2017-11-20 19:51:38.969 [info][tid:com.facebook.react.JavaScript] Model fetched!
2017-11-20 19:51:38.968947-0800 camera[2808:1694897] Model fetched!
2017-11-20 19:51:39.085460-0800 camera[2808:1694545] svm_type c_svc
kernel_type rbf
gamma 0.000244141
nr_class 2
total_sv 97
rho 0.467251
label 1 0
probA -5.34389
probB -0.156837
nr_sv 46 51
SV
0.3278981828007359 5:0.93250179 8:2.0136087 11:2.8867192 14:0.57158649 16:1.1969955 19:1.2844154 20:0.097958535 21:2.7672584 29:0.085689664 33:2.4410281 35:0.048395634 40:0.5292545 43:0.026218385 47:1.4601691 48:1.3122737 54:4.3637943 57:0.70868182 58:3.1910131 60:0.62172848 61:3.1783092 68:0.87323213 71:0.65454847 72:1.202968 73:2.2565515 90:0.49647731 93:1.0256643 95:1.7748675 98:1.6373122 103:0.018649071 108:1.2171577 113:0.19387662 115:0.96137989 123:3.2428927 125:3.1465654 127:0.59765375 130:2.65716 131:0.68586147 141:0.20154279 143:2.0557554 146:1.9896706 147:1.5091631 150:0.11507498 155:2.5605545 156:0.34060448 157:1.2211245 163:0.91133463 164:0.44356358 166:1.0564358 172:1.1373464 175:0.11959201 176:0.26447207 188:0.60830659 190:1.3575654 191:0.11130513 194:0.099838197 195:0.30228135 199:4.0016022 200:2.998297 204:3.5373354 205:2.3839386 211:1.6306676 218:1.0142212 221:4.6005177 223:1.8114175 224:0.070331901 229:1.8780099 239:1.9300385 240:1.0186868 242:2.5137208 244:1.5101008 258:3.7197943 260:1.2916348 262:1.8612854 263:2.3549609 267:3.1112001 269:3.0864401 271:0.13793087 282:2.2634714 284:0.020304203 288:1.5332193 289:1.7379861 290:0.37557745 292:2.2115147 295:1.0060397 296:2.0972331 301:0.42301372 306:1.9411511 315:0.24874607 318:0.22158197 325:1.2470878 329:0.1070272 333:0.8519215 340:2.6277456 342:5.040627 345:1.6160069 347:3.772033 350:2.7830403 351:3.2149861 354:1.1121056 365:1.4016166 366:0.85322189 371:0.82046497 373:1.9478617 379:1.9218774 385:0.69728923 386:2.9333973 389:1.8318629 391:2.1314566 392:0.055717424 403:0.24177086 409:1.1199064 425:1.4380273 427:1.3152776 432:0.83104354 442:1.5515572 444:1.1739322 446:1.8855252 449:0.20995483 459:0.37906009 460:1.1458588 461:2.1588883 462:1.1541297 463:0.044884503 464:0.13238111 466:0.77286404 468:0.467884 469:1.9557354 490:1.3159249 491:2.4852862 494:3.1088672 495:1.5404123 498:1.7595756 499:1.1419648 500:3.5296984 504:3.5051508 505:1.2020793 507:0.02798672 509:0.24818861 517:0.4410409 533:0.75104988 534:0.62724829 536:0.61931175 540:1.0300145 545:1.3722963 546:1.7150553 554:0.22005296 565:2.367743 566:1.2113823 575:0.9224509 578:1.520849 581:0.24457663 583:3.0799172 586:3.140301 590:0.64574522 592:4.4730573 
```

The next step from here it to pass that data into our CNN. And to do that, we need to integrate our CNN objective-C classes into our fork of react-native-camera.

## **Thanks** 
Brian Cline [@standardhuman](https://github.com/standardhuman) and Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for comments and feedback on this post.
