---
layout: post
title: Independent Developer Workstation
tags: [productivity, agile]
---

In one of my projects I did some work on process improvements. The main target was to set up independent developer workstation. We stubbed out all the interfaces. Earlier we tried to share a development database in a group of 4-5 developers. But believe me, I’ve seen developers throwing coffee cups when someone touches their data. Imagine you spending couple of precious hours out of your day on analysing the data and then getting it in right state to run your test cases and the moment before you hit “Run”, someone drops in and mess up all your data. How insane!!

Well the answer is simple, give me my own database and save yourself from the typical excuses. I evaluated couple of open source databases like MySQL, Postgres, EnterpriseDB and finally settled on OracleXE. Well we use Oracle in production and hence its easy to setup development environment with OracleXE with almost no extra tweaks in the scripts. I used DBUnit for testing database modules. It works quite well, but I think my strategy to setup test data was not optimum. It takes bit longer to run the tests now. I used Dumbster as a fake SMTP server and XStream for creating enriched objects for my tests.

Also one useful tool I would like to share here is DBMonster. It really helped me for generating mass random test data. Now I don’t have to request the DBA’ to give me test data. Well its not that simple also, as you need to understand the states of the dataset to create the seed xml files. But I think its still better than having human dependencies.

Happy Development :-) !!
