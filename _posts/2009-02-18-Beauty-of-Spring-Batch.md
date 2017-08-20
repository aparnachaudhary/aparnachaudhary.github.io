---
layout: post
title: Beauty of Spring Batch
tags: [spring-batch, enterprise-integration]
---

Spring source community is coming up with spring batch 2.0 in Q2-2009. Spring batch is the first java based framework for batch processing. I think the decades of experience of Accenture in enterprise batch processing really helped for defining the use cases.

Most of the batch applications need to process high volume business critical transactional data. While doing so some set of non functional requirements (NFR) are sort of mandatory in such applications. These NFRs include performance, scalability, restartability, repeatability. I worked with couple of investment banks and my experience says that such batch applications are developed based on either Messaging model or Multi-threading model. Lot of efforts and time is spent by architects, developers and testers in building this robust infrastructure for batch processing. Also we cannot overlook the cost involved. Whenever you move across projects, you end up creating your own batch processing framework. Sigh!!

Some nice features that are introduced in Spring Batch 2.0 are conditional step execution, finer metadata access control and chunk based processing. To perform chunk based processing, we need to configure the commit-interval in a step. The transaction is committed after number of items specified in commit-interval are processed.

Given the features provided and use cases handled by Spring batch, it can prove to be the de-facto framework for enterprise batch applications.
