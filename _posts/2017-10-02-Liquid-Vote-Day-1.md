---
layout: post
title: Liquid Vote Day 1
---

## Goal?

Build [liquid.vote/berkeley]("https://liquid.vote/berkeley").

What's [liquid.vote]("https://liquid.vote/")?

See [liquid.vote/sf]("https://liquid.vote/sf").

Oh [liquid.vote/sf]("https://liquid.vote/sf") is broken? They're rebuilding the new version of liquid.vote? 

And it's ![under construction](https://motherboard-images.vice.com/content-images/contentimage/26327/1444070256569233.gif)

```
[3:00] 
yup... but it's messy
[3:00] 
they're between versions it looks like
[3:00] 
2 different APIs
[3:00] 
two different front ends
[3:00] 
and we all know that the solution to 2 different APIs
[3:00] 
is to make a 3rd API

Lawrence Stiers [3:00 PM] 
hahahaha

Nick Sippl [3:00 PM] 
duh.
[3:01] 
it looks like the backend code is still private
[3:01] 
and David is in India
[3:01] 
to give a talk
[3:01] 
and his CTO partner... I think....
[3:01] 
lives up the street from me
[3:01] 
but maybe he's in india too

Lawrence Stiers [3:02 PM] 
at least they're not far
[3:02] 
(pause for canned laughter)
```
 
## By Time You Read This Blog Post

liquid.vote might be rebranded `united.vote`.
And `united.vote/sf` might again be alive.

But not today, on Day 1 of `liquid.vote/berkeley` 

## Today We Do git Forensics

```
Lawrence Stiers [3:22 PM] 
gh-pages is like 100 commits ahead of that 0.1.0 branch
[3:22] 
but that branch was updated more recently
[3:22] 
hard to tell what's going on

Nick Sippl [3:22 PM] 
Yeah.

Lawrence Stiers [3:22 PM] 
not ahead of that branch in particualr
[3:22] 
but relative to master

Nick Sippl [3:22 PM] 
So this should be a blog post
[3:22] 
technical blog post
[3:22] 
Day 1
[3:22] 
because this is real software engineering
[3:23] 
"hey here's a mess. figure it out."
[3:23] 
this is what it takes to write production code
[3:23] 
it's not about cleanliness and order
[3:24] 
it's about "wtf really? ok. let's see what we can do."

Lawrence Stiers [3:25 PM] 
yeah

[3:25] 
woof

[3:25] 
so ostensibly.. a whole lot more was done between commits in the 0.1.0 branch

[3:26] 
or one of the commits pulled in gh-pages' changes, and then started making changes on top of that?

Nick Sippl [3:26 PM] 
yeah

[3:26] 
sounds about right

Lawrence Stiers [3:26 PM] 
just trying to wrap my head around it

[3:27] 
can't use # of commits as a measuring stick between two branches, even if there's a large disparity

[3:27] 
i feel like a forensic accountant sussing the two branches out haha
```

Indeed. This is what Day 1 on any legacy project feels like.

"Who did this? And why did they do this?"

I suppose we use that `v0.1.0` branch, even though `master` and `gh-pages` seem somewhat complete. We go towards the future.

## Now Where To Start?

Starting somewhere is easy. Starting in the right place is hard.

Make a GET to {% raw %} `https://pure-api.liquid.vote/v2/legislation/?json={%22legislature%22:%22us%22}` {% endraw %}

And it responds with `{"message":"You found a bug! Let us know at bugs@liquid.vote and we'll buy you coffee.","status":500}`

Make a GET to `https://api.liquid.vote/`

And it responds with `Hello world`

Hello indeed.

## Dev and Prod

I have git access to some version of the source for this `api.liquid.vote` server. It's backed by RethinkDB, and integrated with New Relic, deployed to heroku. I'd have to ask David for the keys. Or I'd have to generate my own. Redploy it.

I also have access to the backing legislative data for San Francisco. So I know the schema of the input data. David's backend handles the other things like users, votes and delegation. Mostly I care about handling the legislative data. That's the right place to start.

My personal preference in databases is Postgres anyway. So if I can figure out what the front-end is asking for, maybe I can sling my own API to serve up the necessary data. That's actually not a bad refactor, in that it splits the backend into two smaller services; one for handling legislation data... and the other for handling delegation and voting. It allows for some specialization as well. Clearly structured data like legislation should be in a structured database. And user graphs, should be handled by something that's good for handling graphs, which I assume might have been the initial motivation for rethink.

So let's get a handle on endpoints. What does the client request? What fails? What does it intend to render?












