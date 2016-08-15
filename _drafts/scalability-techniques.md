---
layout: post
title: Notes on scalability techniques
comments: true
---

Scalability is one probably one of the hardest challenges when you're building a distributed system. It also the _desired_ one: if you need to scale your app, it means you have many happy users! Recently I found a great resource which tries to answer the question of how to design a scalable system. Funny enough: it is a website which contains materials and tips preparing for software engineering interviews ([hired in tech](www.hiredintech.com)). Below are my notes from reading through those materials.

The design process can be summarized as a list of steps one has to follow. It is also called as the _strategy for approaching system design questions at tech interviews_ :)

1. **Define use-cases of the system.** This means specifying exactly what the system should and maybe more importantly what it _shouldn't_ do. The objective is to have a _clear_ understanding of the scope of the requirements (e.g. does it have to gather specific extra statistics, do we plan to provide a UI). Setting some priorities is also important: it will allow to make better decisions about trade-offs later on.
2. **Identifying constraints.** In this step we want to identify the **sources of CPU cost** and **volume of processed/stored data**. For the first part we want to typically estimate: 
  * How many users are we going to have? Time-frame (per second/per day/per week etc) is important: thanks to that you can estimate roughly how many machines are needed.
  * What operations will be typically executed?
  * How many requests of a kind can be expected based on the number of users?
  * Is there a type of request which dominates everything else? 
  * Does it require using a special system?
  * What are the limits within which we want to operate (e.g. is there a max cap on concurrent users)?

  To estimate the volume of data we need to think about those topics:
  * What information is stored? 
  * What is the size of each of the _atomic_ pieces of information?
  * What is the size of _atomic_ pieces of information _x_ the number of requests?
  * What do you need to do with that data? Only store it for further analysis? Or read it/update constantly?
  * Can you purge some of it after some time?

  At this stage it is also ok to make some assumptions: remember to clearly document/specify what assumptions are being made.

3. **Create the abstract design.** The goal here is to create the diagram of components which will be present in the system, specify how each of them works and how they work (communicate) together. This can be organized as _layers_ and _services_. The specification of each component should include the crucial details (e.g. if this service computes an integral - what kind of algorithm it will include): this allows to further refine the data load and computational costs. Try to analyze how the system works in an end-to-end case (you're trying to determine whether the use-cases from 1. are fulfilled). 

4. **Understand the bottlenecks.** First: identify where the bottlenecks are (e.g. how many requests per second are there - can a single machine cope with that? how much data is going to be read in each request? if there is a db: can it be served from a single machine? can you distribute it?). Second: understand how they impact the initial design: maybe if there would be more fine-grained components it would be easier to solve problems with the bottlenecks?

5. **Scaling.** Here we address the problems found in point 4. You should first try to use some well known techniques to avoid reinventing the wheel. List of techniques + their trade-offs is below.

The important side notes are: 
* The scope is really important. It is necessary to understand the priorities to make good trade-offs.
* The _abstract design_ is not finite - you can tweak it :)
* **ALL** estimations on this level are just ballpark figures: we want to know whether the system requires 1000s of servers or just several to handle the projected load. To know better: just get the current numbers and run them against your projections. Based on experience: this will feel good/bad. At the end you can always just experiment with letting only a small percentage of users in and checking if the projections still hold. If not - get back to design because you just identified new bottlenecks.

After writing this down I realized that this look like a check-list for any new system design process :) So, here goes the list of techniques which can be used to solve scalability/availability problems:

1. **Load-balancing.** This can be done on different levels (e.g. DNS round-robin over a pool of IPs, dedicated entry point to the stack - the load-balancer). The trade-off to make is introducing a single point of failure (the load balancer itself). To solve this: buy another load-balancer and set it to replicate the first one's work in active-active model (each of them handles a portion of traffic; if one goes down the other one takes all the load). There is also a problem with session data: user can potentially hit a different server each time. This can be solved by delegating storage to a centralized _external_ data store available to all servers (RDBMS, external persistent cache - e.g. Redis). The application code should be therefore _stateless_ what means it doesn't store any extra data on side (just contains business logic). This way it is very easy to scale the pool of application servers by simply adding more units. _External_ means that the db doesn't reside on the same machines as application code (if app server crashes - db is left intact).
2. **Caching.** Replace costly queries (typically db queries) by storing the result of query in memory. The next time a service tries to run the same query, result is almost instantaneous. The caveats are: 1) size of cache is limited to e.g. size of RAM, 2) when data in db is updated you need to update the cached value (and possibly evict all results of queries which depend on that value), 3) cached queries have to be frequent enough to see a boost (if you issue the query only once in a while, the cached result might be evicted before the query hits again). Examples of in-memory caches (other types are usually not performant enough): [memcached](https://memcached.org/) and [redis](http://redis.io/). [Le Cloud blog](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache) suggests that to avoid some of the problems (e.g. no 2 from presented list) it is better to cache _objects_ as in understanding of your programming language. The pattern is as follows: when the first request comes, the code 'assembles' the object, what may include multiple db queries. Next you serialize the object and send it to cache. When the next request come in you can just deserialize the necessary objects from cache. The other upside is that it is possible to apply the idea of _asynchronous_ processing (instead of filling the cache with the objects which are required by the current request, you can fill the cache with pre-made objects).
3. **DB optimization** . Basically the more queries you throw at the RDBMS the slower it gets (well after reaching a specific threshold set up by the machine capabilities and how well is the db tuned). There are many things actually which you can do: 
    1) _replicate the db_ in a master-slave scheme (master handles writes, slaves handle reads), 
    2) _sharding_ - table is split across multiple nodes (each node has a distinct set of rows; there are multiple problems: synchronization across nodes, network connections, handling failures, transactions spanning across several nodes etc), 
    3) _partitioning_ - dividing data logically into independent parts which can be processed on different machines/nodes (the problem arises when you need to use data from multiple partitions in the same time). Turns out that usually instead of going this route you can usually just switch to NoSQL type database, loosen the transactional constraints and experience better performance, 
    4) _denormalizing_ - usually the most costly thing are the JOINs. Restructuring data in such a way that you don't need to do them will result in a speedup. The other dimension is just making the costly JOINs in the application layer (it is easier to scale application servers than db!).
4. **Asynchronous processing.** As mentioned in _caching_ you can change the way you look at processing data: instead of waiting for requests you can prepare some data structures/objects upfront and just use them when needed. This means that even if they take a considerable amount of time to process, it is hidden from the user. _Asynchronous_ might also mean that you give feedback to the user more often: instead of just freezing the browser until the request is handled, you can show a progress bar, or just show a notification that a request was added to processing queue and you will notify user about the progress/finalization. 

It is very important to understand the difference between _horizontal_ and _vertical_ scaling. _Vertical_ means to increase the capacity of the system by using more powerful hardware (e.g. by using better CPUs, more/faster RAM, bigger hard drives). _Horizontal_ means to add more units of hardware of similar _capacity_, so instead of 2 servers with 2GHz CPUs and 4GB RAM, you're using 10 or 20. Horizontal scaling is widely considered as a better way of solving capacity issues: it is way cheaper to add more commodity hardware and you can do it by using existing technologies. _Vertical_ scaling is limited with the capacity of currently available top-shelf hardware.

In each of the sentences from the previous paragraph you can replace the _hardware_ with _virtual machines_ coming from cloud providers.

Things to consider when thinking about scalability:
* **failures** - each part of the system may fail and you have to remember to think about those scenarios
* **read-heavy or write-heavy** - depending on to which category the system falls into, the design will be completely different (there are also some templates how to deal with specific problems in each case)
* **security** - how the system design affects the security, where is the weakest link of the whole chain

Sources of information:
* [Le Cloud scalability series](http://www.lecloud.net/tagged/scalability)
* [CS75 - Scalability](https://youtu.be/-W9F__D3oY4)
* [MySQL sharding](http://highscalability.com/blog/2009/8/6/an-unorthodox-approach-to-database-design-the-coming-of-the.html)
* [Hired In Tech - system design](http://www.hiredintech.com/system-design/)
* [Networking @ Scale](https://code.facebook.com/posts/1036362693099725/networking-scale-may-2016-recap/)
* [High Scalability blog](http://highscalability.com/)
