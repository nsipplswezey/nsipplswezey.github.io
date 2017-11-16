---
layout: post
title: VoltAGE The Camera Pt. 3
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convoluted-neural-net(CNN) to detect, and generate specific model parameters for specific target objects.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate CNN parameters for detecting an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, and the giftees positive visual identification that an instance of the item is in the hands of the gift-recipient, is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

At this point, our react-native-camera component can get frame data from the obj-c camera layer. Now we want to integrate our neural net for image classification.

That means we need to require in the jetpac DeepBeliefSDK for iOS, as well as it's accompanying classes and methods into our fork of react-native-camera. We also want to keep on eye on functionality. Does VoltAGE need to be able to learn? Or just positively classify? At this stage, an appropriate goal is probably just to get the react layer a steady flow of classification values based on an existing model. 

Our model output looks like the snippet in the notes section below. Except it runs on for another couple thousand lines. That big blob of data is enough to positively detect one of our compelling hand-held gifts. The string containing the paramaterized model is about 15797 characters. As we move forward, we're going to need some thoughtful way to store, organize, update and distribute that model, and other strings like it, on an API server. We'll start by storing it as plaintext `VoltAGE_1_predictor.txt`.

Let's see if we can import this model into an untrained instance of our CNN. The DeepBeliefSDK comes with an example just for this, with a model trained for a winebottle. If we replace the winebottle model with our VoltAGE item model, that should give us a start.

This step is just adding our model to the project as a .txt, and changing two lines of code.
```
  NSString* predictorPath = [[NSBundle mainBundle] pathForResource:@"wine_bottle_predictor" ofType:@"txt"];
  ```
Becomes:
```
  NSString* predictorPath = [[NSBundle mainBundle] pathForResource:@"VoltAGE_1_predictor" ofType:@"txt"];
 ```

And

```
  NSMutableDictionary* values = [NSMutableDictionary
    dictionaryWithObject: [NSNumber numberWithFloat: predictionValue]
    forKey: @"Wine Bottle"];

  dispatch_async(dispatch_get_main_queue(), ^(void) {
    [self setPredictionValues: values];
  });
  ```
  
Becomes:

```
  NSMutableDictionary* values = [NSMutableDictionary
    dictionaryWithObject: [NSNumber numberWithFloat: predictionValue]
    forKey: @"VoltAGE"];

  dispatch_async(dispatch_get_main_queue(), ^(void) {
    [self setPredictionValues: values];
  });
  ```

## Up And Running

This might seem a bit redundant, as a past blogpost had a similar visual outcome. The distinction here is that this is the first example of a model being exported from one app, and imported by another.

We ca host those models on an API server. Then the react client can fetch them as needed. Additionally, we can start adding functionality to our react-native-camera fork for importing, loading and running the external models. We'll pick that up next post. 

![voltage3](https://user-images.githubusercontent.com/7946707/32876285-f7e6ee56-ca50-11e7-90bd-ce43bc6c72f2.gif)


## Notes
```
optimization finished, #iter = 156
nu = 0.231006
obj = -19.637936, rho = 0.493341
nSV = 86, nBSV = 11
Total nSV = 86
2017-11-15 20:46:00.308180-0800 LearningExample[379:85274] Finalizing CVPixelBuffer 0x1c01384c0 while lock count is 1.
*.*
optimization finished, #iter = 162
nu = 0.229572
obj = -19.648198, rho = 0.450914
nSV = 88, nBSV = 11
Total nSV = 88
2017-11-15 20:46:00.523770-0800 LearningExample[379:85274] Finalizing CVPixelBuffer 0x1c4135fe0 while lock count is 1.
*.*
optimization finished, #iter = 189
nu = 0.216142
obj = -18.153928, rho = 0.433361
nSV = 87, nBSV = 6
Total nSV = 87
2017-11-15 20:46:00.732986-0800 LearningExample[379:85274] Finalizing CVPixelBuffer 0x1c0137840 while lock count is 1.
*
optimization finished, #iter = 147
nu = 0.228993
obj = -19.201451, rho = 0.479462
nSV = 80, nBSV = 7
Total nSV = 80
2017-11-15 20:46:00.943809-0800 LearningExample[379:85274] Finalizing CVPixelBuffer 0x1c0137840 while lock count is 1.
*
optimization finished, #iter = 159
nu = 0.220403
obj = -18.274342, rho = 0.397512
nSV = 85, nBSV = 9
Total nSV = 85
2017-11-15 20:46:01.164452-0800 LearningExample[379:85274] Finalizing CVPixelBuffer 0x1c0137840 while lock count is 1.
*
optimization finished, #iter = 187
nu = 0.197262
obj = -20.802525, rho = 0.467251
nSV = 97, nBSV = 9
Total nSV = 97
------------- SVM File output - copy lines below ------------
svm_type c_svc
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
0.3278981828007359 5:0.93250179 8:2.0136087 11:2.8867192 14:0.57158649 16:1.1969955 19:1.2844154 20:0.097958535 21:2.7672584 29:0.085689664 33:2.4410281 35:0.048395634 40:0.5292545 43:0.026218385 47:1.4601691 48:1.3122737 54:4.3637943 57:0.70868182 58:3.1910131 60:0.62172848 61:3.1783092 68:0.87323213 71:0.65454847 72:1.202968 73:2.2565515 90:0.49647731 93:1.0256643 95:1.7748675 98:1.6373122 103:0.018649071 108:1.2171577 113:0.19387662 115:0.96137989 123:3.2428927 125:3.1465654 127:0.59765375 130:2.65716 131:0.68586147 141:0.20154279 143:2.0557554 146:1.9896706 147:1.5091631 150:0.11507498 155:2.5605545 156:0.34060448 157:1.2211245 163:0.91133463 164:0.44356358 166:1.0564358 172:1.1373464 175:0.11959201 176:0.26447207 188:0.60830659 190:1.3575654 191:0.11130513 194:0.099838197 195:0.30228135 199:4.0016022 200:2.998297 204:3.5373354 205:2.3839386 211:1.6306676 218:1.0142212 221:4.6005177 223:1.8114175 224:0.070331901 229:1.8780099 239:1.9300385 240:1.0186868 242:2.5137208 244:1.5101008 258:3.7197943 260:1.2916348 262:1.8612854 263:2.3549609 267:3.1112001 269:3.0864401 271:0.13793087 282:2.2634714 284:0.020304203 288:1.5332193 289:1.7379861 290:0.37557745 292:2.2115147 295:1.0060397 296:2.0972331 301:0.42301372 306:1.9411511 315:0.24874607 318:0.22158197 325:1.2470878 329:0.1070272 333:0.8519215 340:2.6277456 342:5.040627 345:1.6160069 347:3.772033 350:2.7830403 351:3.2149861 354:1.1121056 365:1.4016166 366:0.85322189 371:0.82046497 373:1.9478617 379:1.9218774 385:0.69728923 386:2.9333973 389:1.8318629 391:2.1314566 392:0.055717424 403:0.24177086 409:1.1199064 425:1.4380273 427:1.3152776 432:0.83104354 442:1.5515572 444:1.1739322 446:1.8855252 449:0.20995483 459:0.37906009 460:1.1458588 461:2.1588883 462:1.1541297 463:0.044884503 464:0.13238111 466:0.77286404 468:0.467884 469:1.9557354 490:1.3159249 491:2.4852862 494:3.1088672 495:1.5404123 498:1.7595756 499:1.1419648 500:3.5296984 504:3.5051508 505:1.2020793 507:0.02798672 509:0.24818861 517:0.4410409 533:0.75104988 534:0.62724829 536:0.61931175 540:1.0300145 545:1.3722963 546:1.7150553 554:0.22005296 565:2.367743 566:1.2113823 575:0.9224509 578:1.520849 581:0.24457663 583:3.0799172 586:3.140301 590:0.64574522 592:4.4730573 608:1.389248 615:0.24143478 616:4.2038975 617:0.090583086 619:0.81566751 620:0.042307258 625:1.7474378 629:1.0508316 630:1.1158239 635:2.8924367 638:0.2352922 645:0.79167908 651:1.0629879 661:1.7337872 662:0.60505772 675:0.75396395 677:1.1066585 682:0.61797774 684:1.8865715 687:0.052626982 692:0.96160561 694:1.4504572 697:1.4796872 704:0.84839892 705:4.175529 706:1.4315436 713:2.4939961 723:1.980742 724:3.6851401 725:1.758352 728:0.4404332 735:0.29251719 737:1.286465 738:0.43251619 751:2.1009939 752:0.35525551 753:1.0229774 757:1.1738899 759:0.052788645 768:0.44397825 772:0.78633499 773:1.0923136 775:0.97636366 776:1.4872847 778:1.4455309 780:2.6493824 782:4.7303839 785:1.7340153 789:1.6812123 794:3.3977933 795:0.7384423 796:1.0428138 797:1.0856593 798:1.0469124 799:2.8789129 802:0.99995661 804:0.99808049 805:2.927669 807:0.77596855 813:0.2019237 819:0.45250994 820:0.67282629 827:0.4576965 831:4.3604703 834:0.15740916 837:0.80070978 845:1.06007 848:2.4981017 849:3.0529633 851:1.5792134 853:2.0699663 856:1.3782606 859:0.46876228 865:2.1167917 866:1.3867819 867:0.98039812 868:1.8851459 869:2.2848072 871:1.622507 872:0.29811466 874:0.063984141 882:2.5118458 895:2.8546085 902:1.1606362 904:0.63620651 908:2.0349612 909:3.4053209 913:2.2536125 915:3.1697955 919:3.1941397 921:1.7987825 924:1.5415939 927:0.73135757 938:3.325299 940:0.014504194 945:3.4744089 951:0.55080843 965:1.5116761 966:0.72898692 969:2.0839338 970:0.4541176 972:2.3704522 974:2.3378217 989:2.3793929 991:0.13695414 996:2.8151464 997:0.61762935 999:2.0928445 1001:4.6727128 1004:0.48947999 1009:0.95344126 1014:2.5220916 1015:0.73272622 1016:2.3089459 1019:0.039600432 1022:3.2746284 1024:0.058181774 1026:3.4770689 1027:3.8120832 1031:1.449358 1038:1.3549274 1039:0.52864254 1042:1.3389148 1049:4.8162603 1053:1.5233648 1059:1.9357234 1063:1.6540618 1073:1.0620714 1074:0.56042266 1075:0.63990605 1076:0.90002698 1077:0.092239201 1084:2.9027348 1095:2.7064321 1100:3.072298 1110:1.1404141 1111:1.9371063 1118:4.9129381 1121:1.8845919 1122:1.3361499 1125:2.1062546 1128:1.478018 1129:0.11581144 1133:4.5204082 1135:0.9110024 1137:2.8116584 1140:2.1161184 1147:2.6436279 1153:0.045432806 1158:2.1332352 1160:1.8699874 1180:2.6233046 1181:3.5841405 1188:2.565794 1193:3.6454761 1195:0.66092831 1198:1.4127823 1199:4.2364507 1204:3.5225871 1206:4.9381504 1217:1.627171 1221:4.3989682 1224:1.2258146 1227:0.32586515 1228:7.1483727 1229:1.275996 1234:2.1411433 
```
