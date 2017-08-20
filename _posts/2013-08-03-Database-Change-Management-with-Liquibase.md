---
layout: post
title: Database Change Management with Liquibase
tags: [database, liquibase]
---

I still remember those good old days when the database configuration used to begin in the project with three holy scripts create.sql, data.sql and drop.sql. Every time you make a change to object model (which you do during early phase of projects), the data model needs to be adapted. As soon as the project grows, the mess begins. Additional update scripts are added for every release and someone needs to maintain those changes also in the three holy scripts. This is quite a cumbersome and error prone job. On top of that if you need ability to rollback a release, then complexity of the scripts only increases.

One of the golden mantra of Agile methodology is [orange]*Do just enough upfront design*. If this applies to code then why not apply it to database which is also a type of source which you eventually deliver to customer. With these thoughts Liquibase was born to take away the pain from developer’s life and bridge the gap between Dev and DBA. Liquibase is a database change management tool. Following blog shares some do’s and don’ts to keep life easy while using Liquibase.

## Liquibase Concepts

![](/img/Liquibase.jpg)

## Do’s and Don’ts

* Avoid multiple changes per changeset to avoid failed autocommit statements that can leave the database in an unexpected state.
* Partition the changelog. Instead of creating one huge changelog per application; create smaller sub-changelogs and use the include script statement to link these into the Master changelog.
* Maintain separate changelog for Stored Procedures and use runOnChange=”true”. This flag forces LiquiBase to check if the changeset was modified. If so, liquibase executes the change again.
* Try to write changesets in a way that they can be rolled back. e.g. use relevant change clause instead of using custom <sql> tag .
* Include a <rollback> clause whenever a change doesn’t support out of box rollback. (e.g. <sql>, <insert>, etc)
* Always include a <preconditions> clause in critical changes (e.g. before dropping a table it could be wise to check if the table has any data.)
* Use comments in the change sets. They say *A stitch in time saves nine!*
* Do not ever edit a changeset (exceptions: script, error handling, <sql> tags with runOnChange=”true”)
* Leverage Liquibase to manage your Reference Data. Environment separation (DEV, QA, PROD) can be achieved using “context”.


In general, Liquibase documentation is quite extensive. But I thought it could be handy to get a one pager summary of features, philosophy and such details about Liquibase. I’ve created a reference card using Asciidoc on GitHub. The refcard has three sections Overview, Configuration and Commands. The Overview sections summarizes Liquibase philosophy and features. Configuration section explains how to use maven plugin for different operations. And the last Commands section provides information on all the change elements (e.g. insert, createTable, etc) supported by Liquibase and their rollback capability.

https://github.com/aparnachaudhary/liquibase-refcard. Following image gives a sneak peak into how the refcard looks like. The refcard is available in HTML5 format and you can also print a 2 column layout. I’m not very happy with the current print layout. But I’ve limitations when it comes to CSS and JS. Please feel free to fork the repository and make it better.

Refcard: http://aparnachaudhary.me/liquibase-refcard/


![](/img/Liquibase-Overview.png)


*Happy Development!*

