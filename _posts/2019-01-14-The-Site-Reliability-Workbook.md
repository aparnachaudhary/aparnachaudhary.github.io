---
layout: post
title: Notes from The Site Reliability Workbook
tags: [SRE, SLI, SLO, SLA]
---

Recently I read the book “The Site Reliability Workbook”. I tried to capture the notes in the following post.
  
## SLI Menu

An SLI is a service level indicator—a carefully defined quantitative measure of some aspect of the level of service that is provided.

### Request Response

#### Availability - The proportion of valid requests served successfully.
  
  > The availability of a system serving interactive requests from users is a critical reliability measure. If your system is not responding to requests successfully, it's safe to assume it is not meeting your users' expectations of its reliability.
  
  > Turning this specification into an implementation requires making two choices: which of the requests this system serves are valid for the SLI, and what makes a response successful?

  > Percentage of HTTP GET requests for /profile/{user} or /profile/{user}/avatar that have 2XX, 3XX or 4XX (excl. 429) status measured at the load balancer

#### Latency - The proportion of valid requests served faster than a threshold.
  
  > The latency of a system serving interactive requests from users is an important reliability measure. A system is not perceived as "interactive" by its users if their requests are not responded to in a timely fashion.
  
  > Turning this specification into an implementation requires making two choices: which of the requests this system serves are valid for the SLI, and what threshold marks the difference between requests that are fast enough and those that are not?

  > Latency can be equally important to track for data processing or asynchronous work-queue tasks
  
  > If you have a batch processing pipeline that runs daily, that pipeline probably shouldn't take more than a day to complete. Users care more about the time it takes to complete a task they queued than the latency of the queue acknowledgement.
  
  > One thing to be careful of here is only reporting the latency of long-running operations on their eventual success or failure. If the threshold for operation latency is 30 minutes but the latency is only reported when it fails after 2 hours, there is a 90 minute window where that operation was missing expectations but not measurably so.
  
  > Big data systems, such as data processing pipelines, tend to care about throughput and end-to-end latency. In other words: How much data is being processed? How long does it take the data to progress from ingestion to completion? (Some pipelines may also have targets for latency on individual processing stages.)

  > Pipelines are a great way of doing batch processing: And measuring how good your batch processing is going can’t be done on “Did this request work,” because provided the pipeline is running correctly all work will eventually be done, and even if it breaks we have time to fix it without user pain. So we have to think about it a different way.

  > Percentage of HTTP GET requests for /profile/{user} that send their entire response within Xms measured at the load balancer

#### Quality - The proportion of valid requests served without degrading quality.

  > Degrading quality means serving less relevant ads to users, reducing click-through rates
  
  > Turning this specification into an implementation requires making two choices: which of the requests this system serves are valid for the SLI, and how to determine whether the response was served with degraded quality.

### Data Processing

#### Freshness - The proportion of valid data updated more recently than a threshold.

  > Parts of the system responsible for generating the serving data must also produce a generation timestamp that the serving infrastructure can check against a freshness threshold when it reads data

  > Turning this specification into an implementation requires making two choices: which of the data this system processes are valid for the SLI, and the threshold after which generated data should be considered stale.

#### Coverage - The proportion of valid data processed successfully.

  > A coverage SLI functions similarly to an availability SLI when processing data in a system. When users have expectations that data will be processed and the outputs made available to them, you should consider using a coverage SLI.

  > Turning this specification into an implementation requires making two choices: which of the data this system processes are valid for the SLI, and how to determine whether the processing of a particular piece of data was successful.

#### Correctness - The proportion of valid data producing correct output.

  > In some cases it can be important to measure not just that a processing system processes all the data it should have, but that it produces the correct outputs while doing so.

  > Turning this specification into an implementation requires making two choices: which of the data this system processes are valid for the SLI, and how to determine the correctness of output records.
  
  > A common strategy is to have "golden" input data that produces known-good outputs when processed. If this input data is sufficiently representative of real user data, and is designed to exercise most of the processing system's code paths, then this can be sufficient to estimate overall correctness.

#### Throughput - The number of events that can be executed per unit of time
  
  

### Storage

#### Durability


## SLI Measurement

### Generic Assumptions about SLI/SLO measurements

* Aggregation intervals: “Averaged over 1 minute”
* Aggregation regions: “All the tasks in a cluster”
* How frequently measurements are made: “Every 10 seconds”
* Which requests are included: “HTTP GETs from black-box monitoring jobs”
* How the data is acquired: “Through our monitoring, measured at the server”
* Data-access latency: “Time to last byte”

### SLI Measurement Strategies

* Application Level Metrics
* Server-side Logs
* Frontend Infra Metrics
* Synthetic Clients/Data
* Client-side Instrumentation

## Why SLOs are important?

An SLO is a service level objective: a target value or range of values for a service level that is measured by an SLI. A natural structure for SLOs is thus SLI ≤ target, or lower bound ≤ SLI ≤ upper bound.

> The product perspective: 
If reliability is a feature, when do you prioritise it versus other features?

> The development perspective:
How do you balance the risk to reliability from changing a system with the requirement to build new, cool features for that system?

> The operations perspective:
What is the right level of reliability for the system you support?



## SLO Examples

* 99% (averaged over 1 minute) of Get RPC calls will complete in less than 100 ms (measured across all the backend servers).
* 99% of Get RPC calls will complete in less than 100 ms.
* 90% of Get RPC calls will complete in less than 1 ms.
* 99% of Get RPC calls will complete in less than 10 ms.
* 99.9% of Get RPC calls will complete in less than 100 ms.
* 99% of latency clients’ Set RPC calls with payloads < 1 kB will complete in < 10 ms.

## SLA

SLAs are service level agreements: an explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs they contain. The consequences are most easily recognized when they are financial (a rebate or a penalty) but they can take other forms.

Whether or not a particular service has an SLA, it’s valuable to define SLIs and SLOs and use them to manage the service.

## Developing SLOs and SLIs

For each critical user journey, stack-ranked by business impact
1. Choose an SLI specification from the menu
2. Refine the specification into a detailed SLI implementation
3. Walk through the user journey and look for coverage gaps
4. Set SLOs based on past performance or business needs

> Make sure that your SLIs have an _event_, a success criterion, and specify where and how you record success or failure. Describe your specification as the proportion of events that were good. 
> Make sure that your SLO specifies both a _target_ and a _measurement window_.

### SLIs + SLOs: A Simple Recipe

1. Identify system boundaries 
2. Define capabilities exposed by each system
3. Plain-English definition of “available” for each capability
4. Define corresponding technical SLIs
5. Start measuring to get a baseline
6. Define SLO targets (per SLI or per capability)
7. Iterate and tune

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

* https://www.usenix.org/sites/default/files/conference/protected-files/srecon18emea_slides_fong-jones.pdf
* https://www.oreilly.com/library/view/the-site-reliability/9781492029496/app01.html
* https://www.usenix.org/sites/default/files/conference/protected-files/srecon18emea_slides_geisberger.pdf
* https://www.usenix.org/sites/default/files/conference/protected-files/srecon18americas_slides_flaming.pdf
* https://landing.google.com/sre/sre-book/chapters/service-level-objectives/

