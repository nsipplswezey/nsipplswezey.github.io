---
layout: post
title: United-Vote-Legacy-Code-and-TwoHardThings
---
> There are only two hard things in Computer Science: cache invalidation and naming things.
>-- Phil Karlton
## Goal?

Improve [http://demo.united.vote/](http://demo.united.vote/).

What's [http://demo.united.vote/](http://demo.united.vote/)?

It's participatory democracy with delegation. If legislation affects you, you can vote on it. If you trust someone elses vote, you can delegate your vote to them. When they vote, you vote along with them.

## Scope: What needs to be done?

There exists a legacy code base react client for rendering legislation and letting users vote on it. I work with engineers in Berkeley, Brooklyn and Oakland. And I have friends and family in Madison, Boulder, Santa Cruz and Sebastopol. And I live in San Francisco. I want to have a live United.Vote for friends and family to test in all of these places.

The specc: 
- a user should be able to see items on upcoming city council agendas
- a user should be able to see past city council agendas
- a user should be notified when a new city council agenda has been set
- a user should be notified of relevant city council agenda topics on new agendas
- a user should be able to regsiter a vote on a city council agenda topic

We've got past agendas worked out. Now we're working on future agendas. And we've run into a classic software engineering problem. The particular screens we're working on to render the next-agenda to the user, depends on the word `date`. And the word `date` in this context is worth a technical blog post.

## All These Dates
