---
layout: post
title: Mate vs Cairngorm
tags: [flex, mate, maven]
---

I’ve recently been comparing frameworks for Flex event handling. We are already using Mate in one of our projects. I encountered couple of problems in the usage and thought of exploring other frameworks available on the shelf.

Here are some Pros and Cons of both the frameworks:

## Mate

### Pros

-   Tag based event driven Flex framework
-   Declarative way of Event management
-   Custom Events are inherited from default Flash event; no framework code in the application
-   Uses the event-bubbling to catch the events with the EventMap without defining a bunch of wiring code

### Cons

-   State changes are to be notified explicitly to all the associated views by making use of property injection
-   Application can fail silently when you inadvertently misspell the name of the event parameter; compiler checking on tags in EventMap is missing
-   Chances of bloated MXML file since the Event dispatch logic is composed into the MXML

## Cairngorm

### Pros

-   Singleton Model – Uses Observer pattern to refresh all associated views on state change
-   Command Pattern – Clearly defines the Unit of Work. Reduces chances of bloated MXML files
-   Introduction of Business Delegate can expedite development process in projects with different teams for Client and Server implementation. Delegate can be mocked by the client development team to return dummy data.

### Cons

-   Makes codebase bit verbose
-   Custom events are inherited from CairngormEvent which introduces tight coupling between the application and the framework code

As per me, Cairngorm is more suitable for enterprise application development with huge development teams. Use of command pattern can really expedite the development process. Mate on the other hand is more suitable for small applications. However, I would expect one improvement in Mate to add compiler checking on tags in EventMap. Sometimes it really sucks when you realise there are some typos only at the runtime when you application breaks.

## References

-   http://www.adobe.com/devnet/flex/articles/cairngorm\_pt1.html
-   http://mate.asfusion.com/
