---
title: "Tradetest API Beta Verison"
excerpt_separator: "<!--more-->"
categories:
  - Tradetest
tags:
  - API
---

Welcome to the beta version of tradetest.

**FYI:** The beta version of tradetest API can be partially supported, if you find malfunction of some functionality, please leave us a message.
{: .notice}

**  **
{: .notice--primary}
- Description:

- Params:

- Return:

** order_volume(symbol=None,volume=100,order_side=OrderSide_Buy,position_effect=PositionEffect_Open,event=None) **
{: .notice--primary}

- Description: Provided interface to broker or a simulated broker. It can be called multiple times inside algo function, whilst each call results in an order to be placed in system.

- Params:
  - symbol: the symbol of your trading target, such as AAPL,MSFT,BABA
  - volume: the count of security you want to trade, overplaced order will be alerted.
  - order_side: OrderSide_Buy or OrderSide_Sell
  - position_effect: PositionEffect_Open or PositionEffect_Close
  - event: pass the event from parameter of superior function

- Return:
  - None

** get_history(symbol=None,start=None,end=None,prepost=True,interval='1d',period='1mo') **
{: .notice--primary}
- Description:
  - Function wrapper for yf.download. Get historical OHLCV data for target security.

- Params:
  - find more on [yfinance Doc](https://github.com/ranaroussi/yfinance)

- Return:
  - pd.Dataframe





** get_fundamentals(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance, get fundamental brief of a symbol.

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_financials(symbol=None,quarterly=True) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance, get financial report of a symbol.

- Params:
  - symbol
  - quarterly: if get quarterly aggregated financial reports?

- Return:
  - pd.Dataframe

** get_balance_sheet(symbol=None,quarterly=True) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance, get balance sheet of a symbol.

- Params:
  - symbol
  - quarterly: if get quarterly aggregated financial reports?

- Return:
  - pd.Dataframe

** get_cash_flow(symbol=None,quarterly=True) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance, get cash flow of a symbol.

- Params:
  - symbol
  - quarterly: if get quarterly aggregated financial reports?

- Return:
  - pd.Dataframe


** get_earnings(symbol=None,quarterly=True) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance, get earning report of a symbol.

- Params:
  - symbol
  - quarterly: if get quarterly aggregated financial reports?

- Return:
  - pd.Dataframe


** get_institutional_holders(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance 

- Params:
  - symbol

- Return:
  - pd.Dataframe

** get_major_holders(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance 

- Params:
  - symbol

- Return:
  - pd.Dataframe

** get_analysts_recommendations(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance 

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_next_release(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Next major press event date.

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_options_max(symbol=None,type="call") **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Get all symbol related option chains regardless of due date.

- Params:
  - symbol
  - type: "call" or "put"

- Return:
  - pd.Dataframe


** get_options(symbol=None,type="call",date=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Get all symbol related option chains given proposed due date.

- Params:
  - symbol
  - type: "call" or "put"
  - date: estimated due date, will return all options with closure date before that.

- Return:
  - pd.Dataframe

** get_news(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Get news availble on yahoo for a symbol.

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_dividends(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Get dividends share for a symbol if there are any.

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_splits(symbol=None) **
{: .notice--primary}
- Description:
  - Function wrapper for yfinance. Get splits share for a symbol if there are any.

- Params:
  - symbol

- Return:
  - pd.Dataframe


** get_constituents(index=None) **
{: .notice--primary}
- Description:
  - Get composites of an index(used ETF in background).

- Params:
  - index: 'DOW JONES','S&P 500','NASDAQ','OTHERS','CHINA','EUROPE'

- Return:
  - pd.Dataframe

** get_all_sectors() **
{: .notice--primary}
- Description:
  - Get all sectors' names in NASDAQ

- Params:

- Return:
  - pd.Dataframe


** get_all_industries() **
{: .notice--primary}
- Description:
  - Get all industries' names in NASDAQ

- Params:

- Return:
  - pd.Dataframe


** get_composites(sector=None,industry=None) **
{: .notice--primary}
- Description:
  - Get components of a specific sector, or of an industry or both. If both params are given, first serach sector then industry.

- Params:
  - sector: u can find all names from `get_all_sectors`
  - industires: u can find all names from `get_all_industries`

- Return:
  - pd.Dataframe

**Currently Not supported function:** get_current_ticker,get_history_local,get_allsymbols_local,get_indicator_local
{: .notice--warning}
