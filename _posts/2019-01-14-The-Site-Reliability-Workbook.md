---
layout: post
title: Notes from The Site Reliability Workbook
tags: [SRE, SLI, SLO, SLA]
---

Recently I read the book “The Site Reliability Workbook”. I tried to capture the notes in the following post.
  
## SLI Menu

### Request Response
  * Availability - The proportion of valid requests served successfully.
  * Latency - The proportion of valid requests served faster than a threshold.
  > If you have a batch processing pipeline that runs daily, that pipeline probably shouldn't take more than a day to complete. 
  > One thing to be careful of here is only reporting the latency of long-running operations on their eventual success or failure. If the threshold for operation latency is 30 minutes but the latency is only reported when it fails after 2 hours, there is a 90 minute window where that operation was missing expectations but not measurably so.
  * Quality - The proportion of valid requests served without degrading quality.
  > Degrading quality means serving less relevant ads to users, reducing click-through rates
  
### Data Processing
  * Freshness - The proportion of valid data updated more recently than a threshold.
  > Parts of the system responsible for generating the serving data must also produce a generation timestamp that the serving infrastructure can check against a freshness threshold when it reads data
  * Coverage - The proportion of valid data processed successfully.
  * Correctness - The proportion of valid data producing correct output.
  * Throughput - The number of events that can be executed per unit of time
  
### Storage
  * Durability

## Generic 

* Aggregation intervals: “Averaged over 1 minute”
* Aggregation regions: “All the tasks in a cluster”
* How frequently measurements are made: “Every 10 seconds”
* Which requests are included: “HTTP GETs from black-box monitoring jobs”
* How the data is acquired: “Through our monitoring, measured at the server”
* Data-access latency: “Time to last byte”



## Measurement Strategies

* Application Level Metrics
* Server-side Logs
* Frontend Infra Metrics
* Synthetic Clients/Data
* Client-side Instrumentation

## Examples
* 99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms (measured across all the backend servers).
* 99% of Get RPC calls will complete in less than 100 ms.
* 90% of Get RPC calls will complete in less than 1 ms.
* 99% of Get RPC calls will complete in less than 10 ms.
* 99.9% of Get RPC calls will complete in less than 100 ms.
* 99% of latency clients’ Set RPC calls with payloads < 1 kB will complete in < 10 ms.


## Availability SLI
* The proportion of valid requests served successfully.
* Percentage of HTTP GET requests for /profile/{user} or /profile/{user}/avatar that have 2XX, 3XX or 4XX (excl. 429) status measured at the load balancer

## Latency SLI

#### Request-Response
* The proportion of valid requests served faster than a threshold.
* Percentage of HTTP GET requests for /profile/{user} that send their entire response within Xms measured at the load balancer

Turning this specification into an implementation requires making two
choices: which of the requests this system serves are valid for the SLI,
and what threshold marks the difference between requests that are fast
enough and those that are not?


### Batch

Latency can be equally important to track for data processing or
asynchronous work-queue tasks. If you have a batch processing pipeline
that runs daily, that pipeline probably shouldn't take more than a day to
complete. Users care more about the time it takes to complete a task
they queued than the latency of the queue acknowledgement.
One thing to be careful of here is only reporting the latency of
long-running operations on their eventual success or failure. If the
threshold for operation latency is 30 minutes but the latency is only
reported when it fails after 2 hours, there is a 90 minute window where
that operation was missing expectations but not measurably so.

> Make sure that your SLIs have an _event_, a success criterion, and specify where and how you record success or failure. Describe your specification as the proportion of events that were good. 
> Make sure that your SLO specifies both a _target_ and a _measurement window_.

> Big data systems, such as data processing pipelines, tend to care about throughput and end-to-end latency. In other words: How much data is being processed? How long does it take the data to progress from ingestion to completion? (Some pipelines may also have targets for latency on individual processing stages.)

> Pipelines are a great way of doing batch processing: And measuring how good your batch processing is going can’t be done on “Did this request work,” because provided the pipeline is running correctly all work will eventually be done, and even if it breaks we have time to fix it without user pain. So we have to think about it a different way.

## Resources and Services

* USE for Resources (CPU, MEM, DISK)
  * resource: all physical server functional components (CPUs, disks, busses, ...) [1]
  * utilization: the average time that the resource was busy servicing work [2]
  * saturation: the degree to which the resource has extra work which it can't service, often queued
  * errors: the count of error events
  * utilization: as a percent over a time interval. eg, "one disk is running at 90% utilization".
  * saturation: as a queue length. eg, "the CPUs have an average run queue length of four".
  * errors: scalar counts. eg, "this network interface has had fifty late collisions".

* RED for Services
  * (Request) Rate - the number of requests, per second, you services are serving.
  * (Request) Errors - the number of failed requests per second.
  * (Request) Duration - distributions of the amount of time each request takes.

* Software Resources
  * mutex locks: utilization may be defined as the time the lock was held; saturation by those threads queued waiting on the lock.
  * thread pools: utilization may be defined as the time threads were busy processing work; saturation by the number of requests waiting to be serviced by the thread pool.
  * process/thread capacity: the system may have a limited number of processes or threads, the current usage of which may be defined as utilization; waiting on allocation may be saturation; and errors are when the allocation failed (eg, "cannot fork").
  * file descriptor capacity: similar to the above, but for file descriptors.

## References

* https://www.usenix.org/sites/default/files/conference/protected-files/srecon18emea_slides_geisberger.pdf
* https://www.usenix.org/sites/default/files/conference/protected-files/srecon18emea_slides_fong-jones.pdf

