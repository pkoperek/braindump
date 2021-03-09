---
layout: post
title: Amazon Dynamo paper - a review
comments: true
---

## Key concepts/principles

* Dynamo is a key-value store
* Objects are identified by primary key
* No operations span over multiple data items
* Objects are supposed to be relatively small (< 1MB)
* Failures are expected
* From the four ACID (Atomicity, Consistency, Isolation, Durability) properties
  of a typical database, Dynamo sacrifices _Consistency_ for high availability.
* Efficiency is measured at 99,9th percentile
* Security (authentication and authorization) was removed from the system.
  Dynamo (at least in the paper) is used only for internal services which are
  considered non-hostile.

## Features of Dynamo DB architecture

>> Dynamo uses a synthesis of well known techniques to achieve
>> scalability and availability: Data is partitioned and replicated
>> using consistent hashing [10], and consistency is facilitated by
>> object versioning [12]. The consistency among replicas during
>> updates is maintained by a quorum-like technique and a
>> decentralized replica synchronization protocol. Dynamo employs 
>> a gossip based distributed failure detection and membership
>> protocol. Dynamo is a completely decentralized system with
>> minimal need for manual administration. Storage nodes can be
>> added and removed from Dynamo without requiring any manual
>> partitioning or redistribution. 

## Quotes

* For example, customers should be able to view and add items to their shopping 
  cart even if disks are failing, network routes are flapping, or data centers 
  are being destroyed by tornados. 
*  

## Side observations

* Amazon uses SLA contracts internally (services are expected to behave exactly
  as specified).

[Link to the paper
