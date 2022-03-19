---
title: "Tradetest Update 19.03.2022"
excerpt_separator: "<!--more-->"
categories:
  - Tradetest
tags:
  - Update
---

- New features:
  - user can define global variable in init function
    - e.x. we want to define a variable called "check": `context.check = "me"`
- Bugs:
  - fixed order volume didnt use session based config dict
  - fixed the user holdings not updated if user didnt trade on that day
  - fixed minor viz trouble with json prase

- TODO:
  - smaple trading algo: Nasdaqs Dog