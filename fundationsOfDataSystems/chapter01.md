# 1. Reliable, Scalable, and Maintainable Applications
Many applications today are data-intensive, as opposed to compute-intensive. The bigger problems are usually the amount of data, the complexity of data, and the speed at which it is changing.

A data-intensive application is typically built from standard building blocks. Examples:
- Store data (Databases)
- Rememver the result of an expensive operation (Caches)
- Allow users to search data (Search indexes)
- Send message to another process (Stream processing)
- Periodically crunch a large amount of accumulated data (Bach processing)

When building an application, we still need to figure out which tools and which approaches are the most appropriate for the task at hand. And it can be hard to combine tools.

Many new tools for data storage and processing have emerged in recent years. They are optimized for a variety of different use cases, and they no longer neatly fit into traditional categories, for example, Redis is datastore that also used as message Queue, or Kafka, that it have database-lie durability guarantees.

Three concerns that are important in most software systems:
- **Reliability**: The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error)
- **Scalability**: As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth
- **Maintainability**: Over time, many different people will work on the system, and they should all be able to work on it productively.

## 1.1 Reliability
Expectations about it:
- The application performs the function that the user expected.
- It can tolerate the user making mistakes or using the software in unexpected ways.
- Its performance is good enough for the required use case
- The system prevents any unauthorized access

“continuing to work correctly, even when things go wrong.”: Systems that anticipate faults and can cope with them are called fault-tolerant or resilient. But doing a system tolerant of every possible kind of fault, is not feasible in reality. Only make sense to talk about tolerating _certain types_ of faults.

_Advice_: Fault is not the same as a failure, that is when the system as a whole stops providing the required service. It is usually best to design fault-tolerance mechanisms that prevent faults from causing failures. Keep in mind that we prefer tolerating faults over preventing faults, but there are cases where prevention is better than cure. This is the case with security matters

### Some type of faults
- **Hardware faults**: Our first response is usually to add redundancy to the individual hardware components in order to reduce the failure rate of the system. When one component dies, the redundant component can take its place while the broken component is replaced. Redundancy of hardware components was sufficient for most applications. 

    However, more applications have begun using larger numbers of machines. In some cloud platforms is fairly common for virtual machine instances to become unavailable without warning. Platforms are designed to prioritize flexibility and elasticityi over single-machine reliability.
    Systems that can tolerate the loss of entire machines, by using software fault-tolerance techniques 

- **Software errors**: A systematic error within the system. Such faults are harder to anticipate, and because they are correlated across nodes, they tend to cause many more system failures than uncorrelated hardware faults. The bugs that cause these kinds of software faults often lie dormant for a long time until they are triggered by an unusual set of circumstances.

    There is no quick solution. Lots of small things can help: carefully thinking about assumptions and interactions in the system; thorough testing; process isolation; allowing processes to crash and restart; measuring, monitoring, and analyzing system behavior in production.

- **Human Errors**: Humans design and build software systems. Operators who keep the systems running are also humans. But humans are known to be unreliable. How do we make our systems reliable?
    - Design systems in a way that minimizes opportunities for error.
    - Provide fully featured non-production _sandbox_
    - Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests Especially valuable for covering corner cases that rarely arise in normal operation.
    - Allow quick and easy recovery from human errors, roll out new code gradually and provide tools to recompute data
    - Set up detailed and clear monitoring, such as performance metrics and error rates.  When a problem occurs, metrics can be invaluable in diagnosing the issue.

Reliability is not just for nuclear power stations and air traffic control software. Bugs in business applications can have huge costs in terms of lost revenue, legal risks, and damage to reputation.

There are situations in which we may choose to sacrifice reliability in order to reduce development cost, but we should be very conscious of when we are cutting corners.

# 1.2 Scalability

When a system is working reliably today, that doesn’t mean it will necessarily work reliably in the future. The common reason for degradation is _increased load_.

Scalability is the term we use to describe a system’s ability to cope with increased load. But it is meaningless to say “X is scalable” or “Y doesn’t scale.”, is more about how can cope when the system load grows.

## Describing Load
Load can be described with a few numbers, load parameters. The best choice of parameters depends on the architecture of your system: it may be requests per second, the ratio of reads to writes, the number of simultaneously active users, the hit rate on a cache, etc

## Describing Performance
What happens when the load increases? Keeping the same resources, how is the performance of your system affected? How much you need increase resources to keep the same performance unchanged?

- **Latency and response time**: Latency and response time are often used synonymously, but they are not the same. The response time is what the client sees. Latency is the duration that a request is waiting to be handled. We need to think of response time not as a single number, but as a distribution of values that you can measure.

    Even in a scenario where all requests should take the same time, you get variation: random additional latency could be introduced. It's common to see the _average_ response time as a metric, but it is no a very good metric, because  it doesn’t tell you how many users actually experienced that delay.

    It is better to use _percentiles_. High percentiles of response times, also known as tail latencies, are important because they directly affect users’ experience of the service. But reducing response times at very high percentiles is difficult because they are easily affected by random events outside of your control, and the benefits are diminishing.

    Queueing delays often account for a large part of the response time at high percentiles. It only takes a small number of slow requests to hold up the processing of subsequent requests, effect known as _head-of-line blocking_. So it is important to messore response time on the side client, not on only on the application.

## Approaches for coping with Load
An architecture that is appropriate for one level of load is unlikely to cope with 10 times that load. 

People often talk of a dichotomy between _scaling up (vertical scaling)_ and _scaling out (horizontal scaling)_.  A system that can run on a single machine is often simpler, but high-end machines can become very expensive, so very intensive workloads often can't avoid scaling out.

Some systems are elastic, meaning that they can automatically add computing resources when they detect a load increase, whereas other systems are scaled manually.

While distributing stateless services across multiple machines is fairly straightforward, stateful data systems from a single node to a distributed setup can introduce a lot of additional complexity. For example, until recently the common choice was to keep your database on a single node (scale up). As the tools and abstractions for distributed systems get better, this common wisdom may change.

_advice_: The architecture of systems that operate at large scale is usually highly specific to the application.

# 1.3 Maintainability
