---
title: "Automate your TradeRepublic"
excerpt_separator: "<!--more-->"
categories:
  - Trading
tags:
  - TradeRepublic
---

As i mentioned in self-intro, i was working on automating my trading account using python. The broker i use is called TradeRepublic, which is a neo-broker, a startup, a fintech with round C completed and most importantly, a broker with trading fee only 1 EUR for sell or buy.

Cut the crap, the API i used was a non-official version, possibly a former inside programmer, who codes the interface, but deleted it cuz of legality reason. The current version is a community one, check [here](https://github.com/Zarathustra2/TradeRepublicApi). I checked through all codes before last merge on january this year, no suspicious code which sent your sensitive information/password to some anonymous server. So we can use it with safety ensured, i also forked the repo and tested against my account, it worked.

The strategy i want to share today is a simple but effective one: double-line strategy, it can be categorized as a fundamental technical strategy, which we set a lower buy price(lower than the current price) and a higher sell price(higher than the current price). The example below focuses only on single security trading, but can be extended to a portfolio with least amount of effort.

## Login part
```
from trapi import *

NUMBER = "Your Phone Number with country code e.x.+49"
PIN = "Your Password"

tr = TrBlockingApi(NUMBER, PIN)
res = tr.login()
```
## Meta data 
We can set our meta data here or dynamically in the code. 
Here we acquire the current market price and set 3 prices based on that.
```
## META 
size = 1
isin = "The Isin to trade" # For example "US01609W1027" BABA_ADR
current_markt_price = tr.ticker(isin)[0]['last']['price']
stop_loss_price = current_markt_price - 0.5
entry_price = current_markt_price - 0.1
stop_profit_price = current_markt_price + 1.0
```

## Market Entry
```
# place a market buy order
# order id: arbitrary number, not ProcessId/trackingId from Login
# gfd: good for day, the limit order is valid for Day Session/Night Session
# gtd: good till day (till day end)
# gtc: good till cancelation (need to cancel)
tr.limit_order(order_id = '123',
        isin=isin,
        order_type="buy",
        size=size,
        limit=entry_price,
        expiry='gfd') 

```

## Double line 
Because TradeRepubic only supported one-side limit price setting, we need to take care of the other side using a while loop.

- That leaves us with 2 options: 
  - limit price for stop loss & while loop for stop profit (1)
  - limit price for stop profit & while loop for stop loss (2)
  - while loop for stop loss & stop profit

But keep in mind, because we are monitoring the market price on a 500ms basis, thus, the target price may be slided. Depends on purposes, we can choose among three options. 

### Option 1
```
# place a market Sell order for stopping loss
tr.limit_order(order_id = '123',
        isin=isin,
        order_type="sell",
        size=size,
        limit=stop_loss_price,
        expiry='gfd')

# while loop to monitor on the price
while (1):
  current_markt_price = tr.ticker(isin)[0]['last']['price']

  if current_markt_price <= stop_loss_price:
    print("Stop Loss......")
    break

  if current_markt_price >= stop_profit_price:
    print("Stop Profit......")

    # place a market Sell order  for stopping profit
    tr.limit_order(order_id = '123',
            isin=isin,
            order_type="sell",
            size=size,
            limit=tr.ticker(isin)[0]['last']['price'],
            expiry='gfd')

    break

  # monitor price change at 500ms level
  sleep(0.5)


```
### Option 3
```
# while loop to monitor on the price
while (1):
  current_markt_price = tr.ticker(isin)[0]['last']['price']
  if current_markt_price <= stop_loss_price:
    print("Stop Loss......")
    break

  if current_markt_price >= stop_profit_price:
    print("Stop Profit......")
    break

  # monitor price change at 500ms level
  sleep(0.5)

# place a market Sell order 
tr.limit_order(order_id = '123',
        isin=isin,
        order_type="sell",
        size=size,
        limit=tr.ticker(isin)[0]['last']['price'],
        expiry='gfd')
```