---
layout: post
title: Scaling a service - examples
comments: true
---

Notes about scalability techniques from list mentioned [here](http://www.hiredintech.com/system-design/sample-architectures/).

* [Instagram](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances)
  * Load balancing: DNS round-robin, then Amazon Elastic Load Balancer with 3 nginx instances; to limit the load SSL is terminated at ELB level.
  * Application servers: stateless application code, can scale horizontally; the workload is rather CPU-bound than memory-bound, so they use High-CPU Extra Large machines
  * Data storage: metadata, users, tags etc live in PostgreSQL (sharded across many different instances: [description](http://instagram-engineering.tumblr.com/post/10853187575/sharding-ids-at-instagram)). All Postges instances run in master-replica setup with Streaming Replication. Photos are stored in S3. Amazon Cloudfront is used as the CDN.

List of ideas/projects to remember:

* Redis - persistent key/value store; can be used as a cache (instead of memcached) 


