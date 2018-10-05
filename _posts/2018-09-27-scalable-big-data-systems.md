---
layout: post
title: Four principles of scalable Big Data systems
comments: true
---

Below are my notes from listening to a podcast with Ian Gorton who talked about
principles in design of systems which process massive amounts of data.

What are the key problems with the Big Data:
* Volume of the data: there is a lot of it, storing it and providing resources
  for processing is a challenge.
* Velocity of the data: how fast does it cahenge, the rate of change.
* Variety of data types which need to be analyzed.
* Veracity which is about data integrity, quality etc.

They all boil down to a single issue: very large, complex data sets are
available now and it is not possible anymore to process them with traditional
databases and data processing techniques. 

_My comment: in 2014 it was indeed true. I think that there is one caveat to
this: Moore's law. 4 years ago CPUs and memory were not as fast/not as
optimized for parallel processing as of today. Some size of data which might be
considered Big Data back then, right now could be handled by older
technologies. This boundary is only going up. This means that for some slower
organizations, actually moving to Big Data tech doesn't make any sense, as they
can solve the problem by just buying faster hardware instead of building a
Hadoop/Spark cluster._

The processing systems need to be distributed and horizontally scalable (i.e.
you can add capacity by adding more hosts of the same type instead of building 
a faster CPU).

Principles to follow building Big Data systems:
* "You can't scale your efforts and costs for building a big data system at the
  same rate that you scale the capacity of the system." - if you estimate that
  within a year your system will be 4x bigger, you can't expect to have 4x
  bigger team.
* "The more complex solution, the less it will scale" - this one is more about
  making the right decisions on choosing technologies. If you choose something
  very complex - it will be very complex to scale.
* "When you start to scale things, statefulness is a real problem because it’s 
  really hard to balance the load of the stateful objects across the available 
  servers" - it is difficult to handle failures, because loosing state and
  recreating it is hard. Using stateless objects is the way to go here.
* Failure is inevitable - redundancy and resilience to failure is key. You have
  to account for failure and  be ready for problems with many parts of the
  system.

I think one golden thought which may be easily overlooked is this:

"Hence, how do you know that your test cases and the actual code that you have 
built are going to work anymore? The answer is you do not, and you never will."

_If you are working with a Big Data system, you can never know how it will
behave in production, because recreating the **real** conditions is too costly.
This means that the only reliable and predictable way to build such a system is
to introduce a feedback loop which will tell you if you haven't broken
anything as early as possible, what boils down to: continuous, in-depth
monitoring of the infrastructure and using CI/CD in connection with techniques
like [blue/green deployment][3]._

My observations:
* It is important to do proper capacity planning and if not capacity planning,
  just estimation of data inflow in the future (e.g. year).
* Key factor which allows to introduce efficiency and cut the costs of
  operating a large system is automation (e.g. instead of manually installing
  servers you need to automate this).
* Simplicity allows for better understanding of what is happening in the system
  => this leads to better understanding bottlenecks and figuring out how to
  avoid them.

Link to the original [talk][1] and [transcript][2].

[1]: https://resources.sei.cmu.edu/library/asset-view.cfm?assetid=310358
[2]: https://resources.sei.cmu.edu/asset_files/Podcast/2014_016_100_310361.pdf
[3]: https://martinfowler.com/bliki/BlueGreenDeployment.html
