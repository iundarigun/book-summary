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
The cost of software is not in its initial development, but in its ongoing maintenance. We can and should design software in such a wat that it will hopefully minimize pain during maintenance. We can follow three design principle to achive this:
- Operability: Make it easy to keep the system running
- Simplicity: Make it easy for new engineers to understand the system
- Evolvability: Make it easy for engineers to make changes to the system in the future.
As always, there are no easy solutions for acheiving these goals

## Operability: Making life easy for operations
An operation team typically is responsible for:
- Monitoring the health of the system
- Tracing down the cause of problems
- Keeping software and platforms up to date
- Keeping tabs on how different systems affect each other
- Anticipating future problems
- Establishing good practices and tools for deployment
- Performing complex maintenance tasks
- Maintaining the security of the system
- Defining process that make operations predictable

Good operability means making routine tasks easy. Data systems can do various thing to make routine tasks easy.
- Providing visibility into the runtime behavior.
- Providing food support for automation and integration
- Avoiding dependency on individual machines
- Providing good documentation
- Providing good default behavior
- Self-healing where appropriate

## Simplicity: Managing complexity
Software projects often become very complex and difficult to understand (big ball of mud). There are various symptoms of the complexity: Explosion of the state space, tight coupling of modules, tangled dependencies, inconsistent naming and terminology, hacks aimed to solving performance problems, spacial-casing to work around issues elsewhere, etc.

When complexity makes maintenance hard, there is a greater risk of introducing bugs when making a change. Reducing complexity greatly improves the maintainability of software, and thus simplicity should be a key goal for the systems we build.

One of the best tools we have for removing accidental complexity is _abstraction_.  A good abstraction can also be used for a wide range of different applications. Not only is this reuse more efficient than reimplementing a similar thing multiple times, but it also leads to higher-quality software.

## Evolvability: Making change easy
It’s extremely unlikely that your system’s requirements will remain unchanged forever. The Agile community has also developed technical tools and patterns that are helpful when developing software in a frequently changing environment,  focus on a fairly small, local scale.

The ease with which you can modify a data system, and adapt it to changing requirements, is closely linked to its simplicity and its abstractions.

---

# Summary

An application has to meet various requirements in order to be useful. There are _functional requirements_ (what it should do, such as allowing data to be stored, retrieved, searched, and processed in various ways), and some _nonfunctional requirements_ (general properties like security, reliability, compliance, scalability, compatibility, and maintainability).

_Reliability_ means making systems work correctly, even when faults occur. Faults can be in hardware (typically random and uncorrelated), software (bugs are typically systematic and hard to deal with), and humans (who inevitably make mistakes from time to time). Fault-tolerance techniques can hide certain types of faults from the end user.

_Scalability_ means having strategies for keeping performance good, even when load increases. In order to discuss scalability, we first need ways of describing load and performance quantitatively.

_Maintainability_ has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system. Good abstractions can help reduce complexity and make the system easier to modify and adapt for new use cases. Good operability means having good visibility into the system’s health, and having effective ways of managing it.