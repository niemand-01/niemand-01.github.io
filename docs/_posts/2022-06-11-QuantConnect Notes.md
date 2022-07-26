---
title: "Notes for QuantConnect"
excerpt_separator: "<!--more-->"
categories:
  - Trading
tags:
  - Quant Platform
  - Summary
---

Driven by job requirements, QuantConnect will be a handy platform for me to write qualitative trading algorithms. After around one week of experimenting, i would like to weigh the pros and cons that i discovered for now. Maybe i will reconsider these terms in the future.

- Pros:
  - Good structured UI for programming
  - Powerful embedded IDE with Code hinting
  - Embedded Jupyter Notebook for potential single function testing & learning
  - Comprehensive Usecase-driven Documentation covering nearly all niche areas one may encounter
  - Well incorporated Data Source from different vendors
  - Detailed tutorial videos & large community
  - Csharp as base language ensures efficiency & quick response.

- Cons:
  - No API Documentation, which makes the learning curve mostly depend on trying out different scenarios
  - Csharp originated framework creates difference in function description & parametizaion
  - Single-class based framework is very cumbersome from my point of view compared to previous framework i knew that was "function-based" [Juejin Quant API](https://www.myquant.cn/gm2/docs/api/python/)
  - Bad Experience happened with weird Self.X parameter or Self.Y function which are not fully documented, functionality overlap with other variants and behavior behind it & return values remain complete unknown, unless we dig out in source code ourself.

- Useful sites
  - [Community-provided Csharp API](https://lean-api-docs.netlify.app/classQuantConnect_1_1Algorithm_1_1QCAlgorithm.html#a9dd24a53da63ccd8fdcbe6b5d9ea8716)
  - [Documentation v1](https://www.quantconnect.com/docs/algorithm-reference/universes#Universes-Configuring-Universe-Securities)
  - [Backtesting Engine Lean Github Repo](https://github.com/QuantConnect/Lean)
  - [All in one Class QCAlgorithm](https://www.quantconnect.com/docs/algorithm-reference/overview#Overview-Introduction)
  - [Demos for Alphamodel](https://github.com/QuantConnect/Lean/tree/master/Algorithm.Python/Alphas)


Nonetheless, techincal difficulty can be solved with help of community I suppose and the more we learn, the more we can achieve better alpha. In following paragraphes, i would like to take down the key notes i summarized during last week.


## Most used architecture
Although quantconnect encourages us to use a framework like this:
```
class VerticalTachyonRegulators(QCAlgorithm):

    def Initialize(self):
        self.SetStartDate(2020, 1, 1)
        self.SetEndDate(2021, 1, 1)
        self.SetCash(100000)

        # Universe selection
        self.month = 0
        self.num_coarse = 500

        self.UniverseSettings.Resolution = Resolution.Daily
        self.AddUniverse(self.CoarseSelectionFunction, self.FineSelectionFunction)
        
        # Alpha Model
        self.AddAlpha(FundamentalFactorAlphaModel())

        # Portfolio construction model
        self.SetPortfolioConstruction(EqualWeightingPortfolioConstructionModel(self.IsRebalanceDue))
        
        # Risk model
        self.SetRiskManagement(NullRiskManagementModel())

        # Execution model
        self.SetExecution(ImmediateExecutionModel())
```
That is indeed fine-grained version of the one i show below, at least from first glance, which include the universe selection, alpha model, portfolio model (how to use our cash), risk model(stop win & loss or unexpected crash), execution model(handle troubles like if orders are not filled).

I was trying to consolidate functions sequential structured & tidy when i wrote my own backtesting framework. The hardcore is however that many parameters/decision makings are tangled with each other (e.x. i want to exit the market if i achieved 5% winrate on certain equity), i dont know how QuantConnect solved this problem, but i believe that is at least part of the reason why community rarely use the framework above, but use the straightforward one below.

Update 26.07.2022:
- HOWTO construct Alphamodel:
  - [AlphaModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/key-concepts?ref=v1#01-Introduction)
  - [Predefined Model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/alpha/supported-models)
- HOWTO construct Portfolio model:
  - [PortfolioModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/key-concepts)
  - [Predefined Portfolio Model](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/portfolio-construction/supported-models)
- HOWTO construct Risk Model:
  - [RiskManagementModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/key-concepts)
  - [PredefinedRiskManagementModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/risk-management/supported-models)

- HOWTO construct Execution Model:
  - [ExecutionModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/key-concepts)
  - [PredefinedExecutionModel](https://www.quantconnect.com/docs/v2/writing-algorithms/algorithm-framework/execution/supported-models)


More popular class looks like this 
```
class MyStrategy(QCAlgorithm):

  def Initialize(self):

    # define start date
    self.SetStartDate(2015, 1, 1)
        
    # define end date
    self.SetEndDate(2021, 1, 1)
        
    # define initial cash
    self.SetCash(100000)

    # define some target symbols

      # either define a single equity + data handling node (optional)
      self.QQQ = self.addEquity("QQQ",Resolution.Daily)
      self.QQQ.SetDataNormalizationMode(DataNormalizationMode.Raw)

      # or define a single forex + data handling node (optional)
      self.pair = self.AddForex("EURUSD", Resolution.Daily, Market.Oanda).Symbol
      self.pair.SetDataNormalizationMode(DataNormalizationMode.Raw)

      # or define a Universe Selection Node to screen equities --- this has a variant of SetUniverseSelection from this tutorial (https://www.quantconnect.com/learning/task/196/Selecting-Universe-By-Sector)
      not elegant at all...
      # Filter function to be implemented
      self.AddUniverse(self.CoarseFilter, self.FineFilter)
      self.UniverseSettings.Resolution = Resolution.Hour
      # usually OnSecuritiesChanged() event handler needs to be implemented afterwards

      # or define a Option Chain derived from an equity + strikes & expiry date limitation
      option = self.AddOption("MSFT", Resolution.Minute)
      # filter option to be +- 3strikes from base stock price, expiry date 20-40 days from now
      option.SetFilter(-3, 3, timedelta(20), timedelta(40))

    # define some indicators

      # either use a predefined one like, for other indicators check docs of QCAlgorithm
      self.sma = self.SMA(self.spy, length, Resolution.Daily)

      # or we can implement one ourself + register it to data source
      self.sma = CustomSimpleMovingAverage("CustomSMA", 30)
      self.RegisterIndicator(self.spy, self.sma, Resolution.Daily)

      # or no indicator class is needed if we dont use OnData function to get 
      # data. Instead this post (https://www.quantconnect.com/forum/discussion/
      # 9597/the-in-amp-out-strategy-continued-from-quantopian/p9) shows that you # can just write a schedule with a handler function. Inside this handler, 
      # we inquiry the self.History which returns a pandas dataframe, then we can # do our familiar pandas tricks & build an indicator column.

      # we may want to warm up for indicators using historical data
      self.SetWarmUp(timedelta(7)) # Warm up 7 days of data.

    # define some schedulers

      # scheduler for liquidating everything around EOB daily
      # self.ExitPositions to be implemented
      self.Schedule.On(self.DateRules.EveryDay(self.symbol),
                       self.TimeRules.BeforeMarketClose(self.symbol, 15),      
                       self.ExitPositions)

      # Registers a handler on the consolidated/aggregated data stream *Daily QQQ*
      self.Consolidate(self.QQQ, Resolution.Daily, self.QQQHandler)
        

    # set Benchmark to compare
    self.SetBenchmark("SPY") 

    # set Brokerage model (.Margin to allow leverage)
    self.SetBrokerageModel(BrokerageName.InteractiveBrokersBrokerage, AccountType.Margin) 

    # some useful tricks for user defined variables

      # if we want to keep a track on our portfolio
      self.portfolioTargets = []

      # if we want to have unique selected equities
      self.activeStocks = set()

      # if we want to track entry price
      self.entryPrice = 0 

      # if we want to pause the trading for a regular period
      self.period = timedelta(30)

      # if we want to control entry Time
      self.EntryTime = self.Time
```

Afterwards, we have our core function from tutorial : OnData, which contains our main trading ideas & orders. OnData is triggered according to our proposed data feed resolution. Nevertheless, this function is rarely used by community, which instead they use Scheduler/Consolidator to have custom function scheduled.

```
def OnData(self, data):

  # wait for things prepared

    # wait for indicators warmed up
    if not self.sma.IsReady:
      return

    # wait for entry time
    # purpose: custom waiting time beyond provided resolution. For example i want this algo run every 10 days
    # then i set Resolution to be daily and self.waitPeriod = delta(10)
    # similarly, i can let job run every 2 hours. Otherwise, just use normal resolution. 
    if self.Time < self.EntryTime:
      return
    else:
      self.EntryTime += self.waitPeriod

    # We can run at a specific timestamp ---> alternatively self.Schedule.On this time
    if not (self.Time.hour == 9 and self.Time.minute == 31):
      return

    # wait for equities selected
    if self.portfolioTargets == []:
      return

    # wait for equities in data ---> how can equity not in data???
    if self.QQQ not in data:
      return

  # our brilliant strategies

    # ...
    # we get our decision to buy or sell

      # Place Order 
      # long = positive quantity, short = negative quantity

      # Variation 1
      self.Buy(self.call.Symbol, quantity)

      # Variation 2
      self.MarketOrder(self.spy, int(self.Portfolio.Cash / price) )

      # Variation 3 
      self.SetHoldings(self.spy, 1)

      # Variation 4, Limit Buy Order
      self.LimitOrder(self.qqq, quantity, price, "Entry Order")

      # Variation 5
      self.portfolioTargets = [PortfolioTarget(symbol, 1/len(self.activeStocks)) 
                            for symbol in self.activeStocks]
      self.SetHoldings(self.portfolioTargets)


      # Update Limit Order
      self.entryTicket = self.LimitOrder(self.qqq, quantity, price, "Entry Order")
      updateFields = UpdateOrderFields()

      # set current price as new price
      updateFields.LimitPrice = price

      # use update() to update limit order
      self.entryTicket.Update(updateFields)

      # Sell Order
      # Variation 1
      self.Sell(self.call.Symbol, quantity)

      # Variation 2
      self.Liquidate(self.symbol)

      # Variation 3: Limit Sell Order
      self.entryTicket = self.LimitOrder(self.qqq, quantity, price, "Entry Order")
      self.stopMarketTicket = self.StopMarketOrder(self.qqq, -self.entryTicket.Quantity, 0.95 * self.entryTicket.AverageFillPrice)




```
