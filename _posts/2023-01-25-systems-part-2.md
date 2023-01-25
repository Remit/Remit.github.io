---
title: Designing an Enterprise System (Part II)
author:
name: Vladimir Podolskiy
link: https://github.com/Remit
date: 2023-01-25 12:00:00 +0100
categories: [Software Design]
tags: [enterprise software, software, software architecture, modularization, self-adaptability, configurability]
render_with_liquid: false
toc: false
comments: true
---

# Designing an Enterprise System, Part II: Aligning with the Use Case via System Design

## Balancing between Specialization and Generalization

In the second part of the blog post on designing an enterprise system, I will focus on the high-level technical aspects that the software should embody to align with its use cases. This blogpost does not aim to provide an exhaustive list of things you should do to align your software with the use case expectations. Instead, the goal here is to highlight the general connection between the expectations of the use cases and how they can be utilized to shape the software system. You can read more about these expectations in the [previous part](https://remit.github.io/posts/systems-part-1/).

The previous blog post depicted the alignment of the software with the use case as the most important category of expectations to fulfill. The daunting issue of fulfilling these kinds of expectations is to design software that both aligns with every use case and remains generic enough to be useful beyond one specific application.

If the software aligns with a very narrow set of requirements of a particular use case at the expense of other use cases, then we enter the territory of _custom software development_. This is a kind of service that IT consultancy companies commonly offer to their customers. Custom software is required when the use case in question is _unique_. For instance, the use cases in the public administration domain fall well into this category since they need to align with how a _specific society_ is structured in terms of institutions, laws, economic relations, and even customs.

On the other end, we have some very generic software systems that have a very uniform value proposition across a very wide spectrum of uses. Operating systems are one of the prime examples. In a nutshell, an operating system provides the same basic functionality of hardware resource management to all of its users. It does not attempt to differentiate between its users. Sure, a user can customize their operating system of choice in some way, but the out-of-the-box value proposition is exactly the same for every user.

Between the custom software and general-purpose systems, there is a whole range of software systems of various degrees of specialization. So, what are the ways to achieve better alignment with the use cases and at the same time to cover a lot of ground?

## Alignment Practices

### Functional Modularization to Cover the Diverse Use Cases

_Functional modularization_ is one of the most common ways to generalize the software functionality in order to expand the portfolio of the supported use cases and at the same time to keep its value differentiated for every single one of them. Unlike what the name of this approach suggests, the first step is actually to figure out the common denominator, that is the _functionality shared across the use cases_. For instance, the data movement service and adherence to some standard protocol for seamless integration are the shared requirements for the use cases in the Internet of Things domain. Once we zoom in, we discover that the specific requirements of every use case are quite unique in comparison to the use cases in other industries. That’s where functional modularization comes to rescue.

Once the common functionality has been distilled from the multitude of the use cases and embodied in the solution, it is the time to discover the finer groupings of the use cases. The use cases should be grouped based on similarity of their requirements to the functionality. When grouping is done properly, the functionality required by a group of use cases will likely not intersect with the functionality required by another group of the use cases. This is what modularization helps us with. The discovered niche functionality is wrapped into a software package (module) that can be plugged into the core software system. Sometimes these packages are called extensions, since they extend the core offering of the system in certain ways.

To expand on the data movement example, one can imagine a use case in industrial automation that demands swift decisions to adapt the flow of parts on the manufacturing line if the parts start to accumulate somewhere along the line. Acceptable latency of at most several milliseconds and constrained capabilities of the shop floor hardware may render the deployment of a full-blown analytics system unfeasible. However, if the data movement platform supports extensions, then fulfilling the strict latency and resource utilization requirements can be achieved by plugging a custom analytics module directly into the data movement software running on the edge. Such an extension will not offer the rich capabilities of a dedicated analytics platform deployed on a rack full of servers, but will do its job just as well. By adding an extension to the data movement service, one gets the alignment with the strict requirements of the use case. This is achieved by extension staying close to where the data is and utilizing the existing edge infrastructure.

### Designing for Nonfunctional Requirements

If the gaps between the needed functionality across niches are easy to address with help of modularization, the requirements _to the characteristics of how these functions are performed_ (e.g. how fast, how reliably, etc) are not that easy to compartmentalize. It turns out that such requirements often touch the very core of the enterprise software and are quite frequently at odds with each other. For instance, if the use case requires the data to be stored extremely reliably and consistently, then the network partition may propel the system to pause the queries to ensure that its state converges after the partition is healed. On the other end, another use case may care less about whether the retrieved data is fully consistent at any given point in time, but requires the operations to be performed at the very instant when they land on the software. By increasing the ability of the software to meet the requirements to data reliability and convergence for one niche, the overall availability of the software may fail to meet the expectations of other use cases. Such tradeoffs are typical in engineering of complex enterprise systems, especially once they become distributed. There are multiple high-level approaches that can be taken to address these tensions.

#### Compromise Solution:

First of all, one can think of a specific implementation as being one of the possibilities in a multidimensional design space. The axes in this space represent the characteristics of the system that are relevant to the use cases, e.g. latency, resource usage. One studies this multidimensional space to find a point that meets the expectations of how the functions of the software are performed for _the majority of the use cases_. The advantage of finding such a _compromise solution_ is that the implementation does not get a lot of moving parts and does not become unnecessarily sophisticated to maintain. The downside of hitting a compromise is that the use cases with very strict requirements on the functionality delivered by the software, like sub-millisecond message passing latency, may find it impossible to rely on such a compromise solution. In addition to that, finding the right balance of characteristics that the software exhibits requires substantial engineering efforts and may likely result in the development slowdown. Largely, the quality of the compromise solution depends on _diversity of the use cases_ available to the solution creators.

#### Configurability:

Another option is to make software configurable to align better with the specific demands of the use case. By setting some flags and changing the numerical options, the use case owner may adapt the software to use less memory in exchange for certain functions being performed slower. Such configurable tradeoffs are frequently a good alternative to the rigid design resulting from a compromise since the use cases are likely to rely only on a subset of characteristics of how the functions are performed by the software. This path is often taken by the providers of the so-called _platform solutions_ that should be adapted to the use cases running on top of them.

Kubernetes is one of such examples. Kubernetes offers a lot of configurations to adapt itself to the requirements of an incredibly diverse set of applications. Naturally, its rich configurability led to a complexity explosion. As is the case with any solution going down this path, Kubernetes is frequently criticized with an abundance of moving parts and configurations that might be quite complicated to set up for the use case if the use case owner lacks deep understanding of the platform.

The workaround for the configuration complexity explosion is usually in providing the consultation services either by establishing a dedicated team within the company or by referring for help to the third-party consultancies. Although this is a widely-adopted practice, still, this leads to frustration in using the solution as well as to monetary and time overhead. Last but not least, this ‘hotfix’ makes use case owners even less self-reliant.

#### Self-adaptability:

To address the complexity explosion resulting from parameterization of the system configuration, engineers sometimes introduce _self-adaptive components_ to the system. The main purpose of making the system or the parts thereof self-adaptive is to avoid manual tuning of software to every specific use case. Every self-adaptive component manages the software behavior in some way. Very commonly, self-adaptive components in the modern systems are introduced to handle the resource management aspect. Overload protection and autoscaling mechanisms are the most prominent representatives of this self-adaptation subdomain.

_Overload protection_ manages the incoming load. If the current or anticipated shortage of system resources does not (will not) allow it to process the incoming load, then the incoming load might get throttled or redirected to some other deployment or a running instance. Thus, overload protection deals with sizing the incoming load to adjust it to the available resources. Its counterpart in cloud environments is _autoscaling_.

The purpose of autoscaling is to dynamically adapt the resources to the incoming load. The decision to add or remove the instances of software (horizontal scaling) or to allocate more or less resources to the running instance(s) of the software (vertical scaling) is taken automatically, based on the available indicators, e.g. current or forecasted resource utilization. Autoscaling can (and should!) be combined with the overload protection. If the former can handle most of the changes in workload, the unanticipated explosive traffic growth can be handled by overload protection until the autoscaling mechanism catches up. This kind of traffic is known as _flash crowd_.

Although I refer to such mechanisms as autoscaling and overload protection as ‘self-adaptive’, self-adaptation is frequently a subject to some form of tuning, so it is not entirely ‘_self_-adaptive’. The tuning opportunities usually include selection of the managed parameters, such as instance count or the amount of resources allocated to the running Kubernetes pods, and of observed indicators used to decide if an adaptation action should be performed. Regardless of the specific tuning, the adaptation mechanism itself can rarely be changed.

Despite the idea of self-adaptive components being very attractive in a sense of _removing the workload-dependent assumptions from the software design process_, the obstacles to introducing more of such components into the enterprise software are profound. First of all, it is not always clear what impact would the interplay between self-adaptation of multiple system characteristics yield for the system as a whole. It may even happen that poorly-tuned self-adaptation mechanisms will behave worse than the predefined configurations, e.g. too aggressive overload protection may render the system unavailable despite having sufficient resources. Secondly, the design of self-adaptation mechanisms often demands very deep insights into the system behavior and the workloads. Unfortunately, such insights are not usually available to the designers of the software. Specifically, workloads and the infrastructure that the software runs on may differ a lot depending on the specific use case and the budgets available for the use case owner. Yet another reason is rooted in the limited transparency of the self-adaptation mechanisms. One can easily identify the root cause of decisions taken by such mechanisms if they are based on formulas combining well-understood parameters such as memory usage or the latency of requests in the system. However, the use of more abstract primitives like credits in the overload protection lends itself to misinterpretation.

The listed challenges in no way mean that it is futile to introduce the self-adaptive mechanisms into the systems. What I intend to convey is that such mechanisms demand careful design and evaluation of their possible interactions when added to the software.

## Key Takeaways

In this post, I’ve tried to summarize the high-level software design strategies that help to align it with multiple use cases but at the same time not to dive into the custom software development.

So, here they are…

To align on the functional axis:
- **functional modularization** - core software product that meets the shared requirements of the use cases whose functionality can be expanded with modules.

To align on how the functions are performed (non-functional requirements):
- **discover a compromise solution** - software should be adapted to meet the demands of the majority of the use cases, the extremes can be ignored;
- **configurability** - how the software performs its functions can be configured;
- **self-adaptability** - parts of (or the whole) software product adapts itself to the requirements of a specific use case dynamically.

The above strategies can also be mixed to get the best out of all the available options.

## Share your experience!

Which of the above strategies did you find successful and which - not? What other software design strategies do you follow? Feel free to share your insights in the comments or in the [private messages](https://www.linkedin.com/in/vladimirpodolskiy/).

