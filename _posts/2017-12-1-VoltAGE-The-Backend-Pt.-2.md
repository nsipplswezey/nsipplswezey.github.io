---
layout: post
title: VoltAGE The Backend Pt. 2
---

## Goal?
VoltAGE is a react-native app that uses a client-side-optimized convolutional-neural-net(CNN) to detect, and generate specific model parameters for detecting specific target physical objects using the front facing camera.

The idea is to create a class of small handheld physical objects that are generally compelling or interesting to real people, and use the mobile app to generate parameters for a CNN based detection model for positively identifying an instance of that class of objects, so that if a user is in possession of one of those objects, they can use the app to facilitate an interaction with the object. The specific interaction, is a gifting of the object from one person to another.

I give someone the object, and indicate in the app that I have gifted it. They download the app, and point their phone at the object, and the phone recognizes that they now hold the object. In combination with the gifter's indication of gifting, the  giftees positive visual identification that an instance of the item is in the hands of the gift-recipient is sufficient verification of the transfer of the gift. It's a fun idea.

## Scope: What needs to be done?

We need a way to store, update and distribute the training output from our VoltAGE detection learning sessions. If you take a look at [The Camera Pt. 3](https://nsipplswezey.github.io/2017/11/15/VoltAGE-The-Camera-Part-3.html) post, we have a client that generates a ~1.2mb .txt file based on ~20 seconds of interaction between our camera with one of our VoltAGE targets. We also have a client that can import any 1.2mb .txt file and start giving using classification scores based on camera interactions with the object.

It turns out storing and fetching these models as strings works fine, and the client can write the string to a tmp .txt file which worked great in [The Camera Pt. 6](https://nsipplswezey.github.io/2017/11/15/VoltAGE-The-Camera-Part-6.html).

Now that we're able to load a model from the API, let's add a new model to the backend.

To keep things simple, we'll just reseed our API database with a new model. We can handle POSTing logic later. And to keep our backend maintainable, we'll use the provided `tasks` abstraction from nodal.js.

A task is simply a container for any repeated or repeatable work that our API server might need to do that can be run with `nodal task <task_name>`. Task running can also be automated as a cron job, though we don't need that yet either.

All we need is a task that reads through a directory of our predictor models like `VoltAGE_1_predictor.txt` and `VoltAGE_2_predictor.txt` and then writes them into our API servers `seed.json` file.

Our `update_seed.js` task looks something like:

```
class UpdateSeedTask {

  exec(args, callback) {

    console.log('Executing update seed');

    //Get file names
    let fileNames = fs.readdirSync(__dirname + '/../predictors')
    console.log('Adding ' + fileNames + " to seed.")

    //Read each file, read the seed, create predictor seed data object, add to see
    fileNames.forEach((name,index,collection) => {

      //Read each file
      let path = __dirname + '/../predictors/' + name 
      let model = fs.readFileSync(path, 'utf8')

      //Read the seed.json and parse it into an object 
      let seed = fs.readFileSync(__dirname + '/../config/seed.json', 'utf8')
      let seedObject = JSON.parse(seed)

      //Create a predictor seed data object
      let predictor = {}
      predictor.history = {} 
      predictor.parameters = model 
      
      //Add it to the seed
      seedObject.development.Detector.push(predictor)
      seedObject.production.Detector.push(predictor)

      //Stringify and write the seed                                                                                                                             
      let seedString = JSON.stringify(seedObject,null,2)
      console.log('Current seed object shape')
      console.log(seedObject)
      fs.writeFileSync(__dirname + '/../config/seed.json', seedString, 'utf8') 
      console.log('Seed updated');
        
    })
    //callback();

  }

}
```

And we can run it with `nodal task update_seed`.

The shape of our simple seed is:

```
Current seed object shape
{ development: { Detector: [ [Object], [Object] ] },
  test: { Detector: [] },
  production: { Detector: [ [Object], [Object] ] } }
```

Having written the updated seed, we can deploy it with a commit, followed by a quick:

```
git push heroku master
heroku run nodal db:compose
```

## Up And Running

Now a request to `https://evening-falls-93288.herokuapp.com/v1/detectors` responds with two distinct models which we can fetch and deploy to our CNN.

The first and second models were trained on two similar, but different VoltAGE targets. And with two models we can start to get a sense of what kind of sensitivity our CNN has to being trained on two similar, but different targets.

We can also continue developing a workflow for steadily and more rapidly adding and updating models for new VoltAGE targets.

```
{
    "meta": {
        "total": 2,
        "count": 2,
        "offset": 0,
        "error": null
    },
    "data": [
        {
            "id": 1,
            "history": {},
            "parameters": "svm_type c_svc\nkernel_type rbf\ngamma 0.000244141\nnr_class 2\ntotal_sv 97\nrho 0.467251\nlabel 1 0\nprobA -5.34389\nprobB -0.156837\nnr_sv 46 51\nSV\n0.3278981828007359 5:0.93250179 8:2.0136087 11:2.8867192 14:0.57158649 16:1.1969955 19:1.2844154 20:0.097958535 21:2.7672584 29:0.085689664 33:2.4410281 35:0.048395634 40:0.5292545 43:0.026218385 47:1.4601691 48:1.3122737 54:4.3637943 57:0.70868182 58:3.1910131 60:0.62172848 ...",
            "created_at": "2017-12-02T05:39:17.924Z",
            "updated_at": "2017-12-02T05:39:17.938Z"
        },
        {
            "id": 2,
            "history": {},
            "parameters": "svm_type c_svc\r\nkernel_type rbf\r\ngamma 0.000244141\r\nnr_class 2\r\ntotal_sv 80\r\nrho 0.623993\r\nlabel 1 0\r\nprobA -5.25186\r\nprobB -0.0825233\r\nnr_sv 37 43\r\nSV\r\n0.07813609292967102 10:0.22175172 11:1.5537319 12:0.14385414 15:1.6191089 16:1.3075602 18:0.86073238 19:1.8425725 21:0.033965662 23:0.86468035 27:5.5633793 29:2.0578187 30:0.62867892 54:1.3694928 68:1.1352413 73:0.17410821 77:0.23199695 89:2.9675913 ...",
            "created_at": "2017-12-02T05:39:17.925Z",
            "updated_at": "2017-12-02T05:39:18.203Z"
        }
    ]
}
```

## Previous Post: [The Camera](https://nsipplswezey.github.io/2017/11/13/VoltAGE-The-Camera.html) Next Post: [The Camera Part 3](https://nsipplswezey.github.io/2017/11/14/VoltAGE-The-Camera-Part-3.html)
