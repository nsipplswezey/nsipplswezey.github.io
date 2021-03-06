---
layout: post
title: United Vote NYC Day 5
---

## Goal?

Build [united.vote/nyc](https://united.vote/nyc).

What's [united.vote](https://united.vote/)?

See [united.vote/sf](https://united.vote/sf).

Oh [united.vote/sf](https://united.vote/sf) is broken? They're rebuilding the new version of liquid.vote? 

And it's ![under construction](https://motherboard-images.vice.com/content-images/contentimage/26327/1444070256569233.gif)

## Day 5

Liquid Vote is now United Vote.

Everything always takes a little longer than you'd expect. Starting is easy, making progress is fun, finishing is hard.

In the spirit of finishing, here's a pearl of wisdom from [@ergray](https://ericxgray.wordpress.com/)'s cleanup of his PR:
> The goal of cleaning up a pull request is for the pull request to clearly solve one issue, while causing as few new other issues as possible. In essence, change the existing codebase as little as possible.

> Your PR should be experienced as something like "in as few lines, of clearly readable and understandable code, that doesn't affect any other functionality of the app, I offer a solution to problem X. What do you think?"

And [Eric's PR](https://github.com/unitedvote/web/pull/57) is [just about there](https://travis-ci.org/unitedvote/web/builds/287179156?utm_source=github_status&utm_medium=notification).

## Changes
A lot of good things
- it's `https://united.vote` now
- purescript API has been updated, and responds with more bills per request
- schema for the API backing `united.vote/nyc` has been updated with `uid`
- `SYNC_BILLS` reducer has been fixed correctly update the `/sf` bills state
- pathing between `/sf` and `/nyc` has been narrowed down to `NextAgendaScreen`

## Next Steps
Let's make ourselves some more problems. How do we gracefully handle more local legislature APIs? What if we add `/oak` or `/santa-cruz` or `/seattle` or `/la`

I suppose the first step towards that end is to find faster ways to build and maintain them. Then address them on the client side.

Which means refactoring my scraper. Which I'm really excited about.

What's the current state, and what are the refactor goals? There's a lot to abstract.

Current state:
- single variable hard coded urls for each agenda
- seperate runs per agenda additional
- scraper doesn't check additional bills against past bills; it just adds them
- no easy way to clean wipe the `seed.json` file if needed
- deployment takes 4-5 commands
- no automation of new agendas as they are released
- query selectors have to be changed manually
- target seed has to be changed manually

Refactor goals:
- allow scraper to take an object containing urls, query selectors, target seed.json
- check additional bills against past bills before adding them
- create a wipe script to reset the production `seed.json`
- automate deploy process if possible
- test refactor on oakland city council agenda

Alright sounds good. Let's start by refactoring our NYC to accomodate these changes. Here's our first shot at an options hash for our current scraper.

```
const nycOptions = {
  agendaUrls : [
    "http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=565756&GUID=2DE3E475-8E5B-4B27-9A8C-58C830156A1B",
    "http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=563540&GUID=26A3BD82-86F3-47FE-A13E-AAC05219B54E" ],
  agendaDataSelectors : {
    date : "#ctl00_ContentPlaceHolder1_lblDate"
  }
  billDataSelectors : {
    bill_id: "#ctl00_ContentPlaceHolder1_lblFile2",
    item_number:"",
	title:"#ctl00_ContentPlaceHolder1_lblName2",
	text: "#ctl00_ContentPlaceHolder1_lblTitle2",
	sponsors:"#ctl00_ContentPlaceHolder1_lblSponsors2",
	fiscal_impact: "",
	status_log: "",
	question: "",
    date:  "#ctl00_ContentPlaceHolder1_lblDate",
	source_doc: "",
	uid: "", 
  },
   billDataPlaceholders : {
    bill_id: "",
    item_number:"id",
	title:"",
	text: "",
	sponsors:"",
	fiscal_impact: "None",
	status_log: [{}],
	question: "A motion was made that this Introduction be Approved by Council approved by Roll Call",
    date:  "",
	source_doc: null,
	uid: "billId", 
  } outputPaths : {
    seed : "../nyc-api/config/seed.json",
    local :  "./agenda.json"
  }		
}
```

Now I want to test this, but to do that requires either the wipe script, or checking bills from the current scrape, against past bills. Definitely the better option is to check the current scrape against already existing seed bills. So let's test that out a little bit. I want to take the current seed array, and compare every element in that array, to every element in the possible scraped data array.

The current seed is about 200 elements, and the new scrape might be 100 elements. So we're talking 200x100, or 20,000 operations here. Each agenda will probably remain around 100 elements. The total body of agendas might grow to say 1,000 - 10,000 elements(10-100 agendas). So at the high end, every scrape might require a check of 10,000 x 100 elements, or 1,000,000 checks. What's that? Quadratic time complexity. Works for now, but not forever. But with some help, dedupping isn't actually that tricky; we can use `_.uniqBy` which sorts the array of scraped bill objects by some specific property like `uid`, and then deduplicates, presumably with linnear time complexity.

Turns out there are a bunch of duplicates, because there are multiple actions on a single bill, in a single agenda. And there are actions on a prior bill, on consecutive agendas. This is all fine for now, because what we want in our bill data is bills, not actions. The action history can go into the `status_log` json field. We can do that later.

What else? Created a little wipescript. Real handy for when I had to update the production seed data. Done.

I took a first shot at essentially specifying the option hash for the scraper. It currently takes the following shape.

```
const nycOptions = {
  agendaUrls : [
    "http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=565756&GUID=2DE3E475-8E5B-4B27-9A8C-58C830156A1B",
    "http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=563540&GUID=26A3BD82-86F3-47FE-A13E-AAC05219B54E" ],
  agendaDataSelectors : {
    date : "#ctl00_ContentPlaceHolder1_lblDate"
  },
  billDataSelectors : {
    bill_id: "#ctl00_ContentPlaceHolder1_lblFile2",
    item_number:"",
    title:"#ctl00_ContentPlaceHolder1_lblName2",
    text: "#ctl00_ContentPlaceHolder1_lblTitle2",
    sponsors:"#ctl00_ContentPlaceHolder1_lblSponsors2",
    fiscal_impact: "",
    status_log: "",
    question: "",
    date:  "#ctl00_ContentPlaceHolder1_lblDate",
    source_doc: "",
    uid: "", 
  },
   billDataPlaceholders : {
    bill_id: "",
    item_number:"id",
    title:"",
    text: "",
    sponsors:"",
    fiscal_impact: "None",
    status_log: [{}],
    question: "A motion was made that this Introduction be Approved by Council approved by Roll Call",
    date:  "",
    source_doc: null,
    uid: "billId", 
  },
  outputPaths : {
    seed : "../nyc-api/config/seed.json",
    local :  "./agenda.json"
  }     
}

```

And deployment automation amounts to:

```
git add .
git commit
git push heroku master
heroku run nodal db:compose
```

Followed by visual inspection. 

So that's just:

```
  "scripts": {
    "test": "mocha ./test/runner.js",
    "start": "node cluster.js",
    "worker": "node worker.js",                                                     
    "deploy": "git add .; git commit; git push heroku master; heroku run nodal db:compose"
  },
```

Works like a charm.

And last but not least... 
> test refactor on oakland city council agenda

What stands out to me from looking at the oakland city council agendas
```
https://oakland.legistar.com/MeetingDetail.aspx?ID=562400&GUID=DF762A8E-4D55-4E79-814D-131985D28C12&Options=info&Search=
https://oakland.legistar.com/MeetingDetail.aspx?ID=568477&GUID=28B72F66-522E-4C39-B005-599E0D41DDB3&Options=info&Search=
https://oakland.legistar.com/MeetingDetail.aspx?ID=564804&GUID=5B718CAF-E7C8-4B65-AAEB-D11850B8EBB9&Options=info&Search=
```

Is that they take a slightly different form from NYC and SF. So while the options hash might be consistent, I imagine there's going to be some fine tuning of the actual scraping. I'll give that a shot tomorrow. I think this is what steady good work looks like.










