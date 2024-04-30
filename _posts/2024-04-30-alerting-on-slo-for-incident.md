---
layout: post
title: SLO Alert rule for detect incident
subtitle: Why SREs using SLO burn rate for detect incident
tags: [sre, slo]
comments: true
mathjax: true
author: VuNguyen
---


Using SLO (Service-Level Objective) for determine issue about Reliability of system is good choice that's today system using for estimate ability "Serve enduser good enough" or "Detect issues about healthy of system" under view of Engineer. Maybe SLO not bring much value when system is operating stably or in other words "stasifying user" the way we see it from "surface". However, for evaluate more accurate something what we see requires accurate statistic and from this we can evaluate "quality" system more exactly. 
So, SREs (Site-Reliability Engineer) using SLO for determine "threshold" that they think it serve enduser "good enough"

There is no problem when system running stably like something we hope. This will be more difficult when system extend over time. At this time, complexity will be linear increase with risk that the system will meet. Identify problems before it happen help SREs have better strategy for response, so target will focus to metrics around SLO.

## Idea for determine risks using Error Budget (EB)

It makes sense that our first step would be to define warnings related to "Error Budget (EB) will likely run out in the next X days.", this more early with target that team defined. That mean, maybe using current SLI (Service-Level Indicator) for calculate Error Rate (ER) of system and ratio it with EB determine before for return result about Budget Remaining (BR).

That idea with key concept is: "If at a time, the amount of EB "burned" will be proportional with severity of problem encountered"

That mean, at a time, the amount of EB more burned <-> The issue is even more concerning

_**What does this mean ?**_

Example, we have a cycle for evaluate SLO is 28 days (time_window=28d) and SLO threshold defined is 99.9% (SLO = 0.999) requests received will be handle successful by the server.

```text
-> EB = 1 - 0.999 = 0.001
```

This is request ratio that system **Allow handle error** in 28d time window. If exceed, "mission fail" (Did not reach the goal).

_Let's say within 28 days, we have total 1 million requests received. According to target we have set, the system **will only be allowed for handle failed 1000 requests** and **999,000 requests remaining will have to be be handle successful**_

If our system running stably like something we hope, then after 28 days, 100% EB will be exhausted (corresponding with BR = 0% after 28 days). This means the same thing about **Burn rate = 1**. This case called "perfect case"

-> When outages occur with system, suppose we calculate and determine that this will exhausted 100% EB after the 14th days (1/2 time_window) -> **Burn rate = 2** (EB burn fast double -> This mean we will have problems with double risk).

At this time, we can show charts corresponding between time window and burn rate (with time_window = 30 days in bellow image)

![burn rate and days](assets/img/burnrate_and_days.png)
  
SLO target is set will be considered passed if in time window, amount EB not "exhausted" earlier. This idea base on EB amount consumed corresponding with (BR) in "perfect case" for estimate system health could be worse.

**-> The higher the burn rate at SLO time window, the faster EB is exhausted -> the greater the risk.**

From there we can be seen evaluating periodically and alert prolems related error budget consumed is necessary.

How evaluate and alert periodically is the question arises now, the following plan we can use for identify EB threshold consume of system for alert when the service has problem.

## Solution for identify burn rate

Following idea we discussed above. Let go to a specific example as follow: 

"Determine that the system is looking for a problem worth paying attention to when the amount of EB consumed in 1 hour is greater than 5%"

_Analysis:_

We have fully Error budget (EB) in 1 cycle 28 days (672 hours). Following perfect case, 100% EB will be use "all" in 672h
-> Avg in 1 hour we allow maximum use

```text
(1/672) x 100% EB = 0.1488% 
```

**So, threshold with value > 5% EB is fit for alert.**

100% EB corresponding with 672h. At this point, we need define the downtime range so that the service does not impact the previously defined SLO.

For 5% EB -> The service allowed incident occured in **33.6h (33h + 3/5h) in range 28 days**

So, in 1h, error budget amount allowed for consumed (corresponding with ER accepted in 1h) is:

```text
ER[1h] < 33.6 x (1 - SLO) = 33.6 x (EB) = 33.6 x 0.1% = 0.036 (error rate per hour)
```

This is ER threshold allowed for time window = 1h in a SLO cycle evaluate with 28 days. We can see with each time calculate ER, ER amount will be allowed < 33.6 times EB we defined

*remind: EB = (1 - SLO)_

However, there is one problem occur:

**x 33.6 times EB** in 1 hour is fit for give decision service occuring problems make consumes EB significantly compare to EB we set. But, if EB consumed < 33.6 times little a bit, there won't be any alert (for example ER = 33 times EB) athough it "burn" 100% EB in 20.3 h.

This is overcome by using an additional short time window to detect problems that occur briefly but cause significant to the EB and this require quick response. Additionally, the short window time helps ensure that the chart recovers quickly when the problem is resolved.

![rolling-window](image-2.png)

_As recommended. We could use **short time window = 1/12 long time window** with the same burn rate._

Therefore, it is recommended to add an evaluation condition for a short time window to help detect problems make burn significantly EB.

**Rule config:**
```
-> expr: 
ER[1h] < 33.6 
OR 
ER[1/12 of 1h = 5m] < 33.6
```

From there, we can see the use of **long time window** to identify the risks of EB consumption in the future to have plans or solutions to overcome when that situation continues in the coming days while **short time window** is used to detect problems that have a significant impact on the system and need to be resolved as soon as possible.

Reference:

- <https://docs.wavefront.com/query_language_windows_trends.html#moving-windows>
- <https://sre.google/workbook/alerting-on-slos/>