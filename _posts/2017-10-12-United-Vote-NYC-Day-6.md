---
layout: post
title: United Vote NYC Day 6
---

## Goal?

Build [united.vote/nyc](https://united.vote/nyc).

What's [united.vote](https://united.vote/)?

See [united.vote/sf](https://united.vote/sf).

Oh [united.vote/sf](https://united.vote/sf) is broken? Not anymore! 

![under construction](https://user-images.githubusercontent.com/7946707/31561828-fdef532c-b00d-11e7-8ff2-67b47eba5e4f.gif)

Construction is over! For now.

## Day 6

Before:
![one bill](https://user-images.githubusercontent.com/7946707/31525385-d24a51d4-af74-11e7-9222-a080cf69b1d3.png)

After:
![all bills](https://user-images.githubusercontent.com/7946707/31525473-62828dde-af75-11e7-81ac-ff4d6d51cc4d.png)

The change?

```
     const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})
  
      action.bills.forEach((bill) => {
 -      bills[bill.bill_uid] = bill
 +      // US legislature uid property name
 +      if (bill.bill_uid) {
 +        bills[bill.bill_uid] = bill
 +      }
 +      // SF legislature uid property name
 +      if (bill.uid) {
 +        bills[bill.uid] = bill
 +      }
      })
  
      const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
```

Turns out the US legislature API gives each bill a `bill_uid` property, and `/sf` bill a `uid`. It got a bit lost in the context of the whole reducer.

```
  case 'SYNC_BILLS': // eslint-disable-line no-case-declarations
    const oldBills = (action.replace ? [] : state.bills[action.legislature || action.date] || [])
    const bills = oldBills.reduce((obj, bill) => Object.assign(obj, { [bill.bill_uid]: bill }), {})

    action.bills.forEach((bill) => {
      // US legislature uid property name
      if (bill.bill_uid) {
        bills[bill.bill_uid] = bill
      }
      // SF legislature uid property name
      if (bill.uid) {
        bills[bill.uid] = bill
      }
    })

    const newBills = _.orderBy(Object.values(bills), ['last_action_date', 'bill_uid'], ['desc', 'desc'])
    newBills.synced = action.synced

    return { ...state,
      bills: { ...state.bills,
        [action.legislature || action.date]: newBills,
      },
    }
```

Which contains all sorts of quirks: a check for an action flag `action.replace`, a use of `action.date` as the flag to distinguish between US national legislature and SF city council bills, a parseable but not entirely clear one-liner where the context of `||` `[]` can get lost on either side of a `:`, a variable named `bills` which in the `/sf` case never actually contains any bills, and then a reference to `action.bills` which actually contains bills, but contains a bad property reference for the response data of the API. Minor schenanigans.

It's a nice small PR, and fixes what's broken. 

And with that, I believe [united.vote](https://united.vote) and [united.vote/sf](https://united.vote/sf) should now be working as intended. In addition, Eric's PR, once merged, should have [united.vote/nyc](https://united.vote/nyc) up and running as well. Not bad. What next?

