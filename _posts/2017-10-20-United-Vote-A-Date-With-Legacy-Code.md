---
layout: post
title: United-Vote-A-Date-With-Legacy-Code
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

At first glance this was confusing.

Here's one instance of `date` in the render method of a component. Here date is a property of `match.params` which means that something in the path of this route matches to date.

``` js
render() {
    const { bills, homescreen, history, location, match, votes } = this.props
    const { date } = match.params
    const key = date || 'us'
    let agenda = bills[key]

    if (!agenda) {
      return <ActivityIndicator />
    }

    if (date) {
      agenda = _.sortBy(agenda, 'itemNumber', 'number')
    }
```

And here's another three: `${API}/bills?date__gt=2017-10-19` and then `billData[0].date` and then `nextAgenda = { ... date:dateData}` in a fetch. In this case, `date` is both a query string argument, and then it's also a property on the response data. And then it becomes a property of the `action` object that's passed to the `SYNC_NEXT_AGENDA` reducer.

``` js
function getNextAgenda() {

      fetch(`${API}/bills?date__gt=2017-10-19`)
        .then(response => response.json())
        .then((nextAgenda) => {

          let billData = nextAgenda.data
          let dateData = billData[0].date
          nextAgenda = {
            bills: billData,
            date: dateData
          }
          
          props.dispatch({ nextAgenda, type: 'SYNC_NEXT_AGENDA' })
```

And here's another instance of date, in the reducer that handles the fetched data. Except the presence of `action.date` here is used as a property name of `state.bills`... and it turns out that `action.replace` is undefined anyway.

``` js
  case 'SYNC_BILLS': // eslint-disable-line no-case-declarations

    //Collecting existing old bills, and deduplicate them(US legislature API has dupes)
    const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
    const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})

    action.bills.forEach((bill) => {
      //US legislature uid property name
      if(bill["bill_uid"]){
        bills[bill.bill_uid] = bill
      }
      //SF legislature uid property name
      if(bill["uid"]){
        bills[bill.uid] = bill
      }
    })
```

And then another, in the `render` method of another component. And here, `date` is used to create a billURL –for the client router.

```
render() {
    const { bill, history, agendaVotes } = this.props

    let currentMunicipality = history.location.pathname.split("/")[1]

    //NYC uses uid
    let billUrl = `/${currentMunicipality}/${bill.date}/${bill.uid}`
    if (bill.introduced) {
      billUrl = `/legislation/${bill.bill_uid}`
    }
```

Anyway. This was a wortwhile blog post to highlight how naming things is hard. United Vote uses `date` in at least 5 different contexts, and subtly they all mean something different. Initially in writing out this post, I thought I would go through and rename these variables, to provide more clarity. But honestly, I can't, off the top of my head think of a good way to add additional names in a way that adds clarity, largely because even though it was confusing to understand the first time, "once I understood then it made sense" -a halmark of a hard problem. I ended up just adding a few comments.

It turns out this multi-context use of date required a funny fix, because at different times in the flow of the application, we either use the `match.params.date` or the `bill.date` which is messy; but hey that's legacy code.

So we ended up with a couple ternary checks:

```
  componentDidMount() {
    const { dispatch, isVerified, match, sessionId, nextAgenda } = this.props
    //const { date } = match.params
    const date = nextAgenda ? nextAgenda.date : match.params.date
```

```
  render() {
    const { bills, homescreen, history, location, match, votes, nextAgenda } = this.props
    //const { date } = match.params
    //const { date } = nextAgenda
    const date = nextAgenda ? nextAgenda.date : match.params.date
```

## Up And Running

The specc: 
- √ a user should be able to see items on upcoming city council agendas √
- √ a user should be able to see past city council agendas √
- a user should be notified when a new city council agenda has been set
- a user should be notified of relevant city council agenda topics on new agendas
- a user should be able to regsiter a vote on a city council agenda topic

Anyway. We're up and running with new city council agendas. And past city council agendas. You should be able to see a working staging server for future NYC bills at until Halloween. https://still-beach-17383.herokuapp.com/nyc

Next up it's automation and notification time, so that new agendas get scraped and pushed automatically when they're released.





