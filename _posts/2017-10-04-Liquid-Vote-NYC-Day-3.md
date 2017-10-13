---
layout: post
title: Liquid Vote NYC Day 3
---

## Goal?

Build [liquid.vote/nyc]("https://liquid.vote/nyc").

What's [liquid.vote]("https://liquid.vote/")?

See [liquid.vote/sf]("https://liquid.vote/sf").

Oh [liquid.vote/sf]("https://liquid.vote/sf") is broken? They're rebuilding the new version of liquid.vote? 

And it's ![under construction](https://motherboard-images.vice.com/content-images/contentimage/26327/1444070256569233.gif)


## A Clone Is Born

Oops. No data. Gotta seed production. A clone is *being* born.

We scrape. We write JSON. We seed our production server with JSON.

We start simple. Scrape all the bill names on the page. Go to the agenda. Highlight a bill name. Highlight it's link. Get a css selector. Write a node script that does the same.

``` js
const fs = require("fs")

const jsdom = require("jsdom");
const { JSDOM } = jsdom;


const options = {};
JSDOM.fromURL("http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=563540&GUID=26A3BD82-86F3-47FE-A13E-AAC05219B54E",options)

.then(dom => {

  //get agenda date
  let dateSelector = "#ctl00_ContentPlaceHolder1_lblDate"
  let dateElement = dom.window.document.querySelector(dateSelector);
  let agendaDate = dateElement.textContent

  //select all times
  let tableSelector = "#ctl00_ContentPlaceHolder1_gridMain_ctl00 > tbody"
  let table = dom.window.document.querySelector(tableSelector)
  //iterate through all rows

  let rows = Array.from(table.children);

  let billData = rows
    .map((element) => {
      let atag = element.children[0].children[0]
      let billid = atag.text;
      let source_doc = atag.href;

      let bill = {
        billid:billid,
        source_doc:source_doc,
        date:agendaDate
        };

      return bill;
  })
    .filter((bill) => bill.source_doc )

  return billData
})
```

Not bad. `billid`, `source_doc` and `date`.

## Then What?

Love promises.

``` js
.then(bills => {

  //now that we have item URLs, we navigate to them
  //then we can generate a bunch of promises?
  //and chain process them

  let agendaDate = bills[0].date;

  //generate an array of promises wrapping up the DOMs of the legislation
  let options = {};
  //let billDetailPages = bills.slice(0,3).map(bill => JSDOM.fromURL(bill.source_doc,options)) 
  let billDetailPages = bills.map(bill => JSDOM.fromURL(bill.source_doc,options)) 


  //process that array of promises with appropriate selectors
  Promise.all(billDetailPages)
    .then((pages) => {

      let fullBills = pages.map((page,id) => {

        let billIdSpan = page.window.document.querySelector("#ctl00_ContentPlaceHolder1_lblFile2");
        let billIdText = billIdSpan.textContent;

        let titleSpan = page.window.document.querySelector("#ctl00_ContentPlaceHolder1_lblName2");
        let titleText = titleSpan.textContent

        let billTextSpan = page.window.document.querySelector("#ctl00_ContentPlaceHolder1_lblTitle2");
        let billText = billTextSpan.textContent

        let sponsorSpan = page.window.document.querySelector("#ctl00_ContentPlaceHolder1_lblSponsors2");
        let billSponsors = sponsorSpan ? Array.from(sponsorSpan.children).map(sponsorTag => sponsorTag.text) : null;

        let bill = {
          billid: billIdText,
          item_number:id,
          title:titleText,
          text: billText,
          sponsors:billSponsors,
          fiscal_impact: "None",
          status_log: [{}],
          question: "A motion was made that this Introduction be Approved by Council approved by Roll Call",
          date: agendaDate,
          source_doc: null,
          uid: agendaDate + billIdText
        }

        console.log(bill);
        return bill;
      });

      console.log(fullBills);

      fs.writeFileSync("./agenda.json", JSON.stringify(fullBills,null,2));
      
    })

})
```

Our output? By the time you read this blog, I suspect it might be a directory, but for now it's a single file at https://github.com/liquidvote/nyc-legislation/blob/master/agenda.json

``` json
  {
    "billid": "M 0542-2017",
    "item_number": 0,
    "title": "Communication from the Mayor – Returning Introduction 1648-A to Council pursuant to recall.",
    "text": "Communication from the Mayor – Returning Introduction 1648-A to Council pursuant to recall.",
    "sponsors": null,
    "fiscal_impact": "None",
    "status_log": [
      {}
    ],
    "question": "A motion was made that this Introduction be Approved by Council approved by Roll Call",
    "date": "9/7/2017",
    "source_doc": null,
    "uid": "9/7/2017M 0542-2017"
  },
  {
    "billid": "M 0543-2017",
    "item_number": 1,
    "title": "Withdrawing the nomination of Mario Gooden (M 538), Landmarks Preservation Commission.",
    "text": "Communication from the Mayor - Withdrawing the nomination of Mario Gooden (M 538) from the City Council for its advice and consent concerning his appointment to the Landmarks Preservation Commission.",
    "sponsors": null,
    "fiscal_impact": "None",
    "status_log": [
      {}
    ],
    "question": "A motion was made that this Introduction be Approved by Council approved by Roll Call",
    "date": "9/7/2017",
    "source_doc": null,
    "uid": "9/7/2017M 0543-2017"
  },
```

Quick and dirty. Lots to cleanup. But that's normal. It works. Cleanup later.

Then what?

## Production Seed

Throw it into the production seed. Push to production. Want to see? 

## Protecting Your API

By the time you read this, these endpoints might be dead. But if you're a savy reader, and the endpoint is live... then you could likely infer that other endpoints are active besides just GET `/bills`.

And you'd be right. Which is why tech blogging is so good for you. To publish something, you must protect it. For now we kill all other endpoints on our API server. Later, we put them behind admin authentication.

*pause for production update*

```
diff --git a/app/controllers/bills_controller.js b/app/controllers/bills_controller.js
index ad096cd..cadc2e1 100644
--- a/app/controllers/bills_controller.js
+++ b/app/controllers/bills_controller.js
@@ -21,7 +21,7 @@ class BillsController extends Nodal.Controller {
 
     Bill.find(this.params.route.id, (err, model) => {
 
-      this.respond(err || model);
+      //this.respond(err || model);
 
     });
 
@@ -31,7 +31,7 @@ class BillsController extends Nodal.Controller {
 
     Bill.create(this.params.body, (err, model) => {
 
-      this.respond(err || model);
+      //this.respond(err || model);
 
     });
 
@@ -41,7 +41,7 @@ class BillsController extends Nodal.Controller {
 
     Bill.update(this.params.route.id, this.params.body, (err, model) => {
 
-      this.respond(err || model);
+      //this.respond(err || model);
 
     });
 
@@ -51,7 +51,7 @@ class BillsController extends Nodal.Controller {
 
     Bill.destroy(this.params.route.id, (err, model) => {
 
-      this.respond(err || model);
+      //this.respond(err || model);
 
     });
```

We'll just comment out all these responses in our bills controller.
Push to production, reseed the db

```
git add .
git commit
git push heroku master
heroku nodal run db:compose
```

Then test:

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -H "Cache-Control: no-cache" -H "Postman-Token: d44062ba-d2ba-07eb-c10d-700e58d1d654" -d 'courseid=1&time=October 12 14:00:00 2017 PST&reserved=false&golfers=0&phone=None' "https://infinite-brushlands-18740.herokuapp.com/bills"
```

## A Poem From Postman
>Could not get any response
There was an error connecting to https://infinite-brushlands-18740.herokuapp.com/bills.
Why this might have happened:
The server couldn't send a response:
Ensure that the backend is working properly
Self-signed SSL certificates are being blocked:
Fix this by turning off 'SSL certificate verification' in Settings > General
Client certificates are required for this server:
Fix this by adding client certificates in Settings > Certificates
Request timeout:
Change request timeout in Settings > General

Music to my ears.

## It's Alive
```
quick examples
[12:03] 
https://infinite-brushlands-18740.herokuapp.com/bills?id__gt=100
[12:03] 
all bills
[12:03] 
with ID greater than 100
[12:03] 
https://infinite-brushlands-18740.herokuapp.com/bills?date=9/7/2017
[12:04] 
All bills with date 9/7/17
Eric Gray [12:04 PM] 
oh cool, that's great
Nick Sippl [12:05 PM] 
All bills where sponsors array containts
[12:05] 
https://infinite-brushlands-18740.herokuapp.com/bills?sponsors__jsoncontains=David%20G.%20Greenfield
12:05] 
David G Greenfield
```
