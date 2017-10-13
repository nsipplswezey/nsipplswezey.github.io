---
layout: post
title: Liquid Vote Berkeley Day 2
---

## Goal?

Build [liquid.vote/berkeley]("https://liquid.vote/berkeley").

What's [liquid.vote]("https://liquid.vote/")?

See [liquid.vote/sf]("https://liquid.vote/sf").

Oh [liquid.vote/sf]("https://liquid.vote/sf") is broken? They're rebuilding the new version of liquid.vote? 

And it's ![under construction](https://motherboard-images.vice.com/content-images/contentimage/26327/1444070256569233.gif)


## Dev and Prod

I have git access to some version of the source for this `api.liquid.vote` server. It's backed by RethinkDB. I'd have to ask David for the keys. Or I'd have to generate my own. Redeploy it. Then scrape data, and write it into the db.

I also have access to the backing legislative data for San Francisco. So I know the schema of the input data. David's backend handles the other things like users, votes and delegation. Mostly I care about handling the legislative data. That's the right place to start.

So let's get a handle on endpoints. What does the client request? What fails? What does it intend to render?

Both production API endpoints are broken.

`pure-api.liquid.vote` tells me:

``` json
{"message":"You found a bug! Let us know at bugs@liquid.vote and we'll buy you coffee.","status":500}
```

And `api.liquid.vote` says:

![parsing](https://files.slack.com/files-pri/T2X7DMCD6-F7C2H1Z9N/screen_shot_2017-10-03_at_9.52.08_am.png)

But [https://github.com/liquidvote/sf-legislation](https://github.com/liquidvote/sf-legislation) says:

``` js
module.exports =
[ { itemNumber: 1,
    id: 161015,
    title: 'Appropriation - Real Property Transfer Tax for Funding Community College Fund - FY2016-2017 - $9,000,000',
    sponsors: [ 'Kim', 'Campos, Avalos, Yee, Mar and Peskin' ],
    text: 'Ordinance appropriating a total of $9,000,000 of Real Property Transfer Tax in FY2016-2017 for funding the Community College Fund to support students attending the City College of San Francisco and placing funds on Controller’s Reserve pending the outcome of General Fund tax revenue measures in the November 8, 2016, Election.',
    fiscalImpact: true,
    statusLog: [ { date: '12/13/2016', status: 'PASSED ON FIRST READING.' } ],
    question: 'Question: Shall this Ordinance be FINALLY PASSED?',
    date: '2017-01-10',
    source_doc: 'bag011017_agenda.pdf',
    uid: '2017-01-10-161015' },
  { itemNumber: 2,
    id: 161094,
    title: 'Contract Amendment - Dominion Voting Systems, Inc. - Voting System - Not to Exceed $21,980,691.25',
    sponsors: undefined,
    text: 'Ordinance authorizing the Department of Elections to enter into the Fourth Amendment to an agreement with Dominion Voting Systems, Inc., to extend the term of the agreement through December 31, 2018, and increase the total not-to-exceed amount by $2,289,758 from $19,690,933.25 to $21,980,691.25. (Elections Department)',
    fiscalImpact: true,
    statusLog: [ { date: '12/13/2016', status: 'PASSED ON FIRST READING.' } ],
    question: 'Question: Shall this Ordinance be FINALLY PASSED?',
    date: '2017-01-10',
    source_doc: 'bag011017_agenda.pdf',
    uid: '2017-01-10-161094' },
  { itemNumber: 3,
    id: 161108,
    title: 'Administrative Code - Library Fines and Fees',
    sponsors: [ 'Mayor', 'Mar' ],
    text: 'Ordinance amending the Administrative Code to modify the fines and fees of the Public Library.',
    fiscalImpact: false,
    statusLog: [ { date: '12/13/2016', status: 'PASSED ON FIRST READING.' } ],
    question: 'Question: Shall this Ordinance be FINALLY PASSED?',
    date: '2017-01-10',
    source_doc: 'bag011017_agenda.pdf',
    uid: '2017-01-10-161108' },
```

And I expect something like that to come back in a response to a GET request to `api.liquid.vote/bills`

## A Decision

We learn what to do, from doing something to begin with, and then reassessing.

Initially, the goal was to create `liquid.vote/berkeley`. Berkeley releases their agendas in PDF, apparently stored on DropBox available [here](https://www.dropbox.com/s/v710j2yhwj545ld/2017-10-03%20Agenda%20Packet%20%28WEB%29.pdf?dl=0). Download the PDF and parse it. Just like SF. Though as I was reading through the PDF, I took a look to see if I could pick out any legislation tracking service behind the PDF. A lot of municipalities use some 3rd party service to manage their legislation tracking. And I couldn't find one for Berkeley. Additionally, some of the motivation for Berkeley, was because someone in my social network was purportedly running for office in Berkeley. I assumed it'd be city council. But they're actually running for a state assembly seat. I've thought about a Liquid California, and now that there's a liquid setup for federal legislation, there's probably a reasonable path to it. But I want to keep the first one as simple and clean as possible.

In the past, I've looked at New York as a good candidate. And on further inspection, it looks like New York has a nice clear consistent format for New York City Council meetings. Bigger potential audience too. Plus Larry lives in Brooklyn, so the content might be relevant to him as a user, and he's working on this as well. See his thoughtful Day 1 blog post here: [https://tandcsurf.github.io/Dealing-With-A-Big-New-Scary-Codebase/](https://tandcsurf.github.io/Dealing-With-A-Big-New-Scary-Codebase/). 

And Brian, who's also working on this, lives in Berkeley. So maybe after we knock out New York, Brian can take on Berkeley.

Decision: Drop Berkeley. Do New York.

## A Clone

Start simple. Stick exactly to the SF schema. Look at the NYC data. Then make adjustments as necessary.

The SF Schema, from [https://github.com/liquidvote/sf-legislation/blob/master/json/011017.js](https://github.com/liquidvote/sf-legislation/blob/master/json/011017.js):

``` js
  { itemNumber: 2,
    id: 161094,
    title: 'Contract Amendment - Dominion Voting Systems, Inc. - Voting System - Not to Exceed $21,980,691.25',
    sponsors: undefined,
    text: 'Ordinance authorizing the Department of Elections to enter into the Fourth Amendment to an agreement with Dominion Voting Systems, Inc., to extend the term of the agreement through December 31, 2018, and increase the total not-to-exceed amount by $2,289,758 from $19,690,933.25 to $21,980,691.25. (Elections Department)',
    fiscalImpact: true,
    statusLog: [ { date: '12/13/2016', status: 'PASSED ON FIRST READING.' } ],
    question: 'Question: Shall this Ordinance be FINALLY PASSED?',
    date: '2017-01-10',
    source_doc: 'bag011017_agenda.pdf',
    uid: '2017-01-10-161094' }
```

A single NYC agenda item: [http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=563540&GUID=26A3BD82-86F3-47FE-A13E-AAC05219B54E&Search=](http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=563540&GUID=26A3BD82-86F3-47FE-A13E-AAC05219B54E&Search=)

And the schema, populated with the data:

``` json
    {
        "billid": "M 0545-2017",
        "item_number": 1,
        "title": "Zoning, Establishing a Special Harlem River Waterfront District, Bronx (C170311ZMX, C170314PPX and 170315ZSX)",
        "sponsors": ["Greenfield"],
        "text": "By the Chair of the Land Use Committee Council Member Greenfield: Pursuant to Rule 11.20(c) of the Council Rules and Section 197-d(b)(3) of the New York City Charter, the Council hereby resolves that the actions of the City Planning Commission on Uniform Land Use Review Procedure application nos. C 170314 PPX and C 170315 ZSX shall be subject to Council review. These items are related to application nos. C 170311 ZMX and N 170314 PPX which are subject to Council review pursuant to Section 197-d(b)(1) of the New York City Charter",
        "fiscal_impact": "None",
        "status_log": [{
            "date": "9/7/2017",
            "status": "Approved, by Council"
        }],
        "question": "A motion was made that this Land Use Call-Up be Approved, by Council approved by consent Roll Call.",
        "date": "2017-09-07",
        "source_doc": "http://legistar.council.nyc.gov/LegislationDetail.aspx?ID=3147881&GUID=76BCCFBB-77F8-4E6D-9743-492CBBE48107",
        "uid": "2017-09-07-M 0545-2017"
      }
```

Not bad. Start simple. Let's get this data into a live api. Spin up a Nodal API server.

```
nodal new
cd nyc-api
nodal g:model bill billid:string item_number:int title:string sponsor:json text:string fiscal_impact:string status_log:json question:string date:string source_doc:string uid:string
nodal g:controller --for bills
nodal s
```

And we're up and running. Then seed the development server with data:



``` json
{
  "development": {
    "Bill" : [
      {
        "billid": 161015,
        "item_number": 1,
        "title": "Appropriation - Real Property Transfer Tax for Funding Community College Fund - FY2016-2017 - $9,000,000",
        "sponsors": ["Kim", "Campos, Avalos, Yee, Mar and Peskin"],
        "text": "Ordinance appropriating a total of $9,000,000 of Real Propery Transfer Tax in FY2016-2017 for funding the COmmunity College Fund to support students attending the City Callege of San Francisco and placing funds on Controller's Reserve pending the outcome of General Funx tax revenue measures in the November 7, 2016, Election.",
        "fiscal_impact": "true",
        "status_log": [{
            "date": "12/13/2-016",
            "status": "PASSED ON FIRST READING"
        }],
        "question": "Question: Shall this Ordinance be FINALLY PASSED?",
        "date": "2017-01-10",
        "source_doc": "bag011017_agenda.pdf",
        "uid": "2017-01-10-161015"
      }
    ]
  },

  "test": {},

  "production": {}
```

Then bootsrap the server to build the db, clear the db, run db migrations, and seed the db.

``` json
{
  "development": {
    "Bill" : [
      {
        "billid": 161015,
        "item_number": 1,
        "title": "Appropriation - Real Property Transfer Tax for Funding Community College Fund - FY2016-2017 - $9,000,000",
        "sponsors": ["Kim", "Campos, Avalos, Yee, Mar and Peskin"],
        "text": "Ordinance appropriating a total of $9,000,000 of Real Propery Transfer Tax in FY2016-2017 for funding the COmmunity College Fund to support students attending the City Callege of San Francisco and placing funds on Controller's Reserve pending the outcome of General Funx tax revenue measures in the November 7, 2016, Election.",
        "fiscal_impact": "true",
        "status_log": [{
            "date": "12/13/2-016",
            "status": "PASSED ON FIRST READING"
        }],
        "question": "Question: Shall this Ordinance be FINALLY PASSED?",
        "date": "2017-01-10",
        "source_doc": "bag011017_agenda.pdf",
        "uid": "2017-01-10-161015"
      }
    ]
  },

  "test": {},

  "production": {
```

Then make a requestion to `localhost:3000/bills` and we're up and running.

``` json
{
  "meta": {
    "total": 1,
    "count": 1,
    "offset": 0,
    "error": null
  },
  "data": [
    {
      "id": 1,
      "billid": null,
      "item_number": null,
      "title": "Appropriation - Real Property Transfer Tax for Funding Community College Fund - FY2016-2017 - $9,000,000",
      "sponsors": [
        "Kim",
        "Campos, Avalos, Yee, Mar and Peskin"
      ],
      "text": "Ordinance appropriating a total of $9,000,000 of Real Propery Transfer Tax in FY2016-2017 for funding the COmmunity College Fund to support students attending the City Callege of San Francisco and placing funds on Controller's Reserve pending the outcome of General Funx tax revenue measures in the November 7, 2016, Election.",
      "fiscal_impact": null,
      "status_log": null,
      "question": "Question: Shall this Ordinance be FINALLY PASSED?",
      "date": "2017-01-10",
      "created_at": "2017-10-03T19:22:06.114Z",
      "updated_at": "2017-10-03T19:22:06.131Z"
    }
  ]
}
```

And then deploy!

```
$ heroku login
$ git init
$ heroku create
$ heroku git:remote -a <your-heroku-app-name> (usually something like hidden-harbor-12345)

$ git add .
$ git commit -am "Up and running"
$ git push heroku master

$ git push heroku master
$ heroku addons:create heroku-postgresql:hobby-dev //create db step
$ heroku run nodal db:prepare //prepare for migrations
$ heroku run nodal db:migrate //create my data models
$ heroku open
```

And make a request to the live server.

## A Clone Is Born

Oops. No data. Gotta seed production. A clone is *being* born.
