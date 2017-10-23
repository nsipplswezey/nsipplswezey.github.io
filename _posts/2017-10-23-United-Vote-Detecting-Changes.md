---
layout: post
title: United-Vote-Detecting-Changes
---
## Goal?

Render city council agendas before city council votes for NYC, Oakland, Berkeley and Boulder.

This work is based off of [http://united.vote/](http://united.vote/)?

What's [http://united.vote/](http://united.vote/)?

It's participatory democracy with delegation. If legislation affects you, you can vote on it. If you trust someone elses vote, you can delegate your vote to them. When they vote, you vote along with them.

You can see a staging environment with a mock future agenda for NYC legislation here: [https://still-beach-17383.herokuapp.com/nyc](https://still-beach-17383.herokuapp.com/nyc)

## Scope: What needs to be done?

There exists a legacy code base react client for rendering legislation and letting users vote on it. I work with engineers in Berkeley, Brooklyn and Oakland. And I have friends and family in Madison, Boulder, Santa Cruz and Sebastopol. And I live in San Francisco. I want to have a live United.Vote for friends and family to test in all of these places.

The specc: 
- √ a user should be able to see items on upcoming city council agendas √
- √ a user should be able to see past city council agendas √
- a user should be notified when a new city council agenda has been set
- a user should be notified of relevant city council agenda topics on new agendas
- a user should be able to regsiter a vote on a city council agenda topic

Next up it's automation and notification time, so that new agendas get scraped and pushed automatically when they're released.

## The Simplest Thing Possible

Detecting change in a webpage, or in a collection of webpages, is not a new problem. In fact it's probably what's known as a solved problem. A solution has been around since 1999 [https://www.changedetection.com/](https://www.changedetection.com/). They'll send you an email whenever they detect change on a webpage. You get emails daily. For NYC, however, setting a change detection notification on the Oct 17 agenda didn't trigger. In fact the Oct 17 agenda dissapeared, and presumably a new agenda was released in its place at a different URL. In fact, here are the two URLs that I fed to changedetection:

- NYC 2017-10-17 [http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=559322&GUID=347EDA3E-24AA-433D-97F9-D19787B7D162&Options=info&Search=](http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=559322&GUID=347EDA3E-24AA-433D-97F9-D19787B7D162&Options=info&Search=) 
- NYC 2017-10-31 [http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=559323&GUID=7EDB29B3-7E53-4B53-BDFF-6C39E9885646](http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=559323&GUID=7EDB29B3-7E53-4B53-BDFF-6C39E9885646). 

The first one is toast, presumably it was replaced with [http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=570244&GUID=9D379D29-09C9-4AB4-822E-9D1AAB699FB6](http://legistar.council.nyc.gov/MeetingDetail.aspx?ID=570244&GUID=9D379D29-09C9-4AB4-822E-9D1AAB699FB6) which contains the meeting minutes. The second one hasn't been updated yet. But if something similar happens to it as happened to the first 2017-10-17 URL, then presumably it'll just disappear, and be replaced with a new URL.

This is an irritating problem, because the change can only be observed once or twice a month, at the time of the city council meetings. Not a very good feedback loop. So what do we do about that. Changes happen infrequently. One approach is to just throw a bunch of different pages into the change detector, and wait until halloween. That's not unreasonable, but it kinda sucks. What if we miss? We wait again?

Another approach is to see if there are archive snapshots of those pages, and see what changed in past months. The wayback machine has a couple crawls [https://web.archive.org/web/*/http://legistar.council.nyc.gov/Calendar.aspx](https://web.archive.org/web/*/http://legistar.council.nyc.gov/Calendar.aspx).

Ah wouldn't you know it! I'm not the only one who's watching the NY City Council agenda page for changes AND changedetector regularly crawls pages that are already on its list. So there's actually a history of [Address: legistar.council.nyc.gov/Calendar.aspx](Address: legistar.council.nyc.gov/Calendar.aspx)
```
Address: legistar.council.nyc.gov/Calendar.aspx

Current status: normal - monitoring 
Number of users monitoring this page: 2 
Last notified: Never 
Most recent sizable change (text, html) : 2017-10-23 13:23 
Most recent change (text, html) : 2017-10-23 13:23 
Last checked for changes: 2017-10-23 13:23 
You started monitoring: 2017-10-23 14:09 
Adult content: no 
Spam page: no 
```

Truthfully I'm not sure exactly what happened. So the first challenge is to figure out what page actually changes when a new agenda is released. And that's going to be different for every municipality.

On top of all of this, legistar claims to offer its own email alerts functionality. So there's something testable today. I'll set an alert for a meeting upcoming tomorrow, and the next day and see how well that works. And of course set alerts for the upcoming meeting on the 31st.

## Up And Running

The specc: 
- √ a user should be able to see items on upcoming city council agendas √
- √ a user should be able to see past city council agendas √
- a user should be notified when a new city council agenda has been set
- a user should be notified of relevant city council agenda topics on new agendas
- a user should be able to regsiter a vote on a city council agenda topic



