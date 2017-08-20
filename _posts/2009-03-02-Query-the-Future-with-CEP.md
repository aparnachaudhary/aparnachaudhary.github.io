---
layout: post
title: Query the Future with CEP
tags: [esper, cep]
---

Complex Event Processing (CEP) is a buzzword that’s been running around the industry for last couple of years. The concept was introduced by David Luckham of Stanford University who has done over a decade of research in this field. Let’s try to understand some terms that are used frequently in this arena.

An Event is a piece of data that represents that something happened in the real world, or in software system. Events often observe a change in state. e.g. a stock tick or a password change. A linearly ordered sequence of events forms Event Stream. While a partially ordered set of events form Event Cloud. So an event stream could be a cloud but the reverse need not be true.

e.g. Set of all stock trades for GOOG within a 5 minute time window is an Event Stream. While all Stocks sold in a business day is an Event Cloud. And above event stream could be a part of this event cloud.

## Why Event Processing

Static data processing models like database doesn’t perform real time processing. It also adds extra cost of storing data. So there is a need of real time processing models. CEP engines behaves exactly opposite to traditional database approach. They say turn the database upside down and that’s your CEP engine. Instead of querying or polling data to run business logic on it; freeze the business logic, let the data flow through it.

## Real World Applications of Event Processing

-   Market Data
-   Financial Institutions – Credit Risk Department – Pre-Trade
-   Algorithmic trading
-   Pattern recognition
-   BAM – Business Activity Monitoring
-   Supply Chain Management
-   Flight Operations Monitoring
-   Gaming Industry

There are four main technical elements associated with Event Processing:

-   CEP: CEP provides the basic language and execution engine for processing events.
-   BAM: Provides a graphical user interface for CEP
-   DSM: DSM helps applications capture event state, perform root cause analysis on event data, and simulate anticipated automated conditions.
-   Event Middleware: Pub/Sub of events between CEP engine and JMS, Database triggers, ESB

In my next posts I would cover more details on CEP Engine, Event Driven Architectures, Event Processing Language and Esper – the open source event stream processing engine.
