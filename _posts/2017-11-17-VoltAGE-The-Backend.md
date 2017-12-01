---
layout: post
title: VoltAGE The Backend
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

We need a way to store, update and distribute the training output from our VoltAGE detection learning sessions. If you take a look at [The Camera Pt. 3](https://nsipplswezey.github.io/2017/11/15/VoltAGE-The-Camera-Part-3.html) post, we have a client that generates a ~1.2mb .txt file based on ~20 seconds of interaction between our camera with one of our VoltAGE targets. We also have a client that can import any 1.2mb .txt file and start giving using classification scores based on camera interactions with the object.

Now we can build a minimal backend to start handling the concerns of storing, updating and distributing these 1.2mb .txt files.

I'm weighing two different attempts at addressing this. 1.2mb of text, I'd argue, is small enough that all those characters could arguable be stored as a long string in Postgres. To give that a shot I spun up a nodal server, built a simple model for our 1.2mb of txt, gave it an associated jsonb field as a holder for any relevant, and then wrote the 1.2mb of .txt into the seed file for the model. Postgres had no problem with this. And our server didn't either. But since our seed is specified as a JSON file, the 1.2mb of text hit VIMs character limit when I tried to open it and visually inspect it. On top of that... the API call with a single model takes about 2.8 seconds to respond completely, and that's running locally.

![image](https://user-images.githubusercontent.com/7946707/32971885-3dc1d11c-cba4-11e7-8421-55c62dc3d0d4.png)

To compare, I stored the same .txt file on github, essentially as as static file hosting. We can access our first VoltAGE model right now at https://nsipplswezey.github.io/VoltAGE-models/VoltAGE_1_predictor.txt. And that request took about 9 seconds.

![image](https://user-images.githubusercontent.com/7946707/32971989-bd628dc6-cba4-11e7-8924-9ac2bb0d7f0b.png)

These two approaches present some interesting tradeoffs. And either would work, truthfully. If we stored our models as strings in our DB, rather than as references to github URLs, we'd likely want two endpoints involved in requesting detector models. The first endpoint could exclude the actual model strings, which would allow us to query based on the json data associated with the model. Once we knew which model we wanted, another endpoint could use data from the first request to specifically request the model we're looking for. Essentially this tradeoff boils down to "do you want to store your .txt as a static file on some other service? Or do you want to store it in your own database?"

The only other thing I want to check is if there's a difference between live response times for these two approaches.

And it seems like there is. Approximately the same data coming from a deployed heroku API server arrives in about 3.5 seconds.

![image](https://user-images.githubusercontent.com/7946707/32971241-1dab66ac-cba1-11e7-92ed-84a3c35a7da3.png)


While there's more to do to get to the bottom of this, I'm left unsure about which approach to take. They both have tradeoffs. Static file hosting seems like a high level set of abstractions, but maybe slower. Storing 1.2mb strings in Postgres seems like lower level abstraction, perhaps a bit more complicated, and a bit faster over the wire. In either case, both of these tests seem a bit slow for 1.2mb of content. God news most gifs are 10x that size.

## Up And Running

I can get the same roughly the same data from [https://nsipplswezey.github.io/VoltAGE-models/VoltAGE_1_predictor.txt](https://nsipplswezey.github.io/VoltAGE-models/VoltAGE_1_predictor.txt) as I can from [https://evening-falls-93288.herokuapp.com/v1/detectors/1](https://evening-falls-93288.herokuapp.com/v1/detectors/1)

Now that I'm up and running with both, I can build the client against either, and see what the tradeoffs exist in terms of client integration.

My initial hunch has been that the static file hosting will be better, because the native iOS layer of VoltAGE expects a .txt file. Having said that, it's likely that the supported data type for passing data from our react layer, which will have done the fetching of our predictor model, is just a string anyway. And that we'll have to use an obj-C class to convert that string into a .txt for importing into our CNN. We'll certainly be able to find out now.

## **Thanks** 
Brian Cline [@standardhuman](https://github.com/standardhuman) and Lawrence Stiers [@tandcsurf](https://github.com/tandcsurf) for comments and feedback on this post.
