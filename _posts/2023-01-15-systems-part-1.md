---
title: Designing an Enterprise System (Part I)
author:
name: Vladimir Podolskiy
link: https://github.com/Remit
date: 2023-01-15 13:00:00 +0100
categories: [Software Design]
tags: [customer expectations, enterprise software, software, requirements analysis]
render_with_liquid: false
toc: false
comments: true
---

# Designing an Enterprise System, Part I: Customer Expectations

## How Enterprise Systems Emerge and What Contributes to This

Well-entrenched in their businesses for decades, enterprises grow complex processes that they eventually start to automate and optimize. These processes emerge in different domains, from business value production all the way to the distribution and supporting functions, like hiring.

Both automation and optimization of the business processes are reachable with means of the software. However, if automation is completely unachievable without the software, business process optimization does not necessarily entail its addition into the process. Business process optimization predominantly relies on domain experts rethinking the existing processes and adapting them to produce more revenue or to reduce costs of following them. Formalizable domain expertise is a rich soil for introducing the software into the business processes.

Regardless of the exact purpose that the software is used for, enterprises can always choose between building (and maintaining!) an in-house solution or purchasing one from a third party. The latter naturally benefits from the _resource pooling effect_. The idea behind this effect is rather straightforward: in short, third-party solution providers, such as product companies, tackle a niche need that is present across a spectrum of the use cases. For instance, there is a need to move the data from some sort of producers, like sensors, to the consumers, like analytical applications. This is quite a common need for businesses in many industries. Multiple use case owners, like businesses or business units, utilize the same third-party solution to fulfill their common need. This enables the provider of this solution to pool the incoming financial and other kinds of resources for the sake of focusing its efforts on improving the solution.

Why is resource pooling needed? When taken separately, the available resources of each use case owner, be they financial or experience, would be _insufficient_ to create the solution of the desired quality and to maintain it. The insufficiency is often due to this need being only a small piece of the complex core business puzzle. For example, car manufacturers are in the business of producing and selling cars. Although data movement is an inherent part of their factory automation processes or connected vehicles technology platform, this might be a mere tiny link of a very long and versatile value-adding chain. Thus, it might not be economically justifiable to make the company completely self-reliant in respect to this small piece. However, when pooled, financial, know-how, as well as the other resources, become sufficient to produce the solution that fulfills the need in the best possible way.

When opting for the third-party solution providers, the use case owners have a multitude of expectations. These expectations offer themselves for a neat grouping into a few categories. When discussing these categories, I’ll focus on what I believe to be the most important expectations that translate into how the solution is shaped from the technological point of view. Every expectation is formulated as a requirement to help you have an easier time finding examples and counterexamples in your experience.

## Expectations Categories

### Category I: Alignment with the Use Case

This category is the most important one, and the reason for that is straightforward. If the solution is not well-aligned with the use case, then the business of the third-party solution provider will face huge impediments to its growth due to having hard time convincing potential customers to rely on the offering. On a high level, this is where both the value proposition and the positioning of the solution play a crucial role.

**Expectation 1. Mapping from the primitives and the patterns of the use case to the primitives and the patterns of the solution should be clear.**

This expectation means that the software system should come as close as possible to using the primitives of the business process that it aims to automate or optimize.

If we consider an MQTT broker implementing a publish-subscribe messaging pattern, then we may see how well its message distribution pattern captures the data flow from the single source of truth (sensors) to various services that each give different arrangement and view on the same data. These varying views on the same data are required to fulfill different business needs relying on the same data, such as parts ordering, production optimization, cost management, etc. Intuitively, complimentary services relying on the same data (or the data produced by the same entities such as manufacturing lines) push the use case owner to think in a paradigm of _data distribution_. As an example, all the measurements from a sensor are required to dynamically adjust the production process, but just a single outlier measurement could suffice to alarm the personnel that they should leave the shop floor.

**Expectation 2. (Non-functional) characteristics of functions performed by the software should align with the expectations from the business process that the software is used in.**

Depending on the use case, the characteristics may differ or have different priorities. If the continuity of the business process depends on the software, then it has to be reliable (avoiding or minimizing the data loss) and it has to be at least as available as the other systems/components interact with it. In other use cases, the delay introduced by the software is expected to be within some bounds. For example, the cumulative time budget allocated to the website backend processing the user request, such as adding an item to the cart, can reach a hundred milliseconds. On the other hand, adaptive production lines deal with physical objects rather than with human perception, hence the expectation might be that the delay is at most a few milliseconds.

Alignment demands third-party solution providers to have really deep insights into the business cases of their customers. For the general-purpose solutions used in business processes across diverse use cases, such as ones loosely unified under an umbrella term ‘Internet of Things’, this might be extremely tricky since the expectations might be contradictory. For instance, by attempting to increase the reliability of the software system in face of failures, it might be unwillingly made slower since more time is needed to reconcile the updates performed on the data. Or, by decreasing the end-to-end latency of the software system, its memory usage may grow substantially since more results are now cached on the application level. If the expectations from the software in different use cases diverge too widely, then it might be the right time to introduce a new specialized product, should the opportunity be attractive enough.

### Category II: Frictionless operating experience

One might of course argue against this category since the value proposition of the software might outweigh its rough edges. However, when the market is competitive, providing frictionless experience becomes crucial. Multiple expectations contribute to this category, but the key idea is that anyone using the software in their business processes wants it to run smoothly. In the end, not many of us like to be reminded that we or the world around us is far from perfect.

**Expectation 3. The new software should run on the existing infrastructure and require little to no training.**

Although this might be an impediment when we are talking about tech enthusiasts, in general, people would be willing to spend as little effort as possible to integrate software into their business processes and operate it. Every reference to the manual or the support service creates friction. Every such event conveys the message that the person operating the software cannot do it on their own, which inevitably leads to negative emotions. This is, of course, the least desirable outcome for the product.

**Expectation 4. The new software should incur as little additional costs as possible.**

This expectation is shaped by the costs already being specified in the contract. However, even if the software license was already paid for, the required infrastructure, maintenance, and operating costs may fall out of the contract’s scope. To a large extent, SaaS solutions are exempt from this issue. Nevertheless, not all the use cases align well with this distribution format, so offering a SaaS-only solution might become a limiting factor rather than an enabling one, depending on the use cases in question. If the software needs to be operated by the use case owner (perhaps, due to need to stay available even if the network connection is lost), its appetite towards compute and storage resources might surface as an additional cost on top of what one already pays for the software. Design choices made in software impact the usage of resources too. In case of a SaaS solution, these costs are being paid by the solution provider, hence the incentive to reduce them is high. In contrast, the on-premise solution’s appetites are being paid for by the use case owner. In the latter case, there is a high chance of friction emerging since the feedback mechanism is poorly (if at all) defined: the impact of high resource usage is not experienced by the solution provider first-hand.

**Expectation 5. The new software should seamlessly integrate into the business process.**

It should be expected that the use cases are to some extent (or even end-to-end) automated. In this case, the software solution offered by the third party provider must align well with the existing systems and avoid creating hiccups when in operation. Technically, this boils down to clear definition of APIs and correct usage of APIs of other software in operation. Even if this sounds easy, there is an additional twist: API versioning. Deprecation of the old API calls and the introduction of the new ones could impair the added software’s capability to perform expected tasks. In addition to offering the required functionality, the added piece of software should not become a bottleneck in the business process, meaning that it should integrate with other systems not only on a functional level but also on a level of its quality characteristics, like introduced delay, resource usage, and throughput.

**Expectation 6. The new software should be future-proof (since the business conditions change).**

It is to be expected that the use case relying on the third-party software gets new requirements, such as keeping the data secure or tracing the requests through the chain of software systems. In that case, the added software needs to enable integrations for the new systems that it potentially does not even have an API for! Since there is no way to foretell all the possible integrations that the software system might need to support in the future, the best that one can hope for is either to make it easily extendable or to constantly expand its functionality to adapt to the changing requirements of the use cases. If the resources of the solution provider are scarce and the number of the use cases grows explosively, then it makes sense to introduce extensibility in the form of SDKs and pluggable modules. Should the system be applied in a new use case, the use case owner can swiftly adapt the system to their needs by creating a custom module.

### Category III: Technological transparency

This whole category is obliged for its existence to the very fear of the unknown that lives within every human being. Naturally, this fear is directed at the technologies as well. Both due to the intellectual property protection reasons and the high complexity of the modern software systems, third-party solution providers are incentivized to hide as much as possible about the inner workings of their software. In turn, this leads to the use case owners not being able to fully rely on the software. They then can opt to continue using the cumbersome non-automated way of doing things since every action is known and has been tried multiple times before. To get a foot in the door, the third-party solution provider needs to ensure that its offering is transparent to the extent that it helps the user rely on it. If the underlying technology is very complex to explain, then it is even more the case that the time and effort should be taken to provide an understandable explanation of how the software behaves and why it does what it does.

**Expectation 7. The new software should be based on proven technologies.**

If the software solution uses some technology that no other software with the same or similar functionality uses, then the use case owners might express uncertainties about the implications of using the technology that they are not accustomed to. This issue gets particularly noticeable in the absence of significant advantages over other comparable solutions. The logic here is quite simple - any unknown technology is an additional risk. If a new risk factor is introduced to the use case, then it should offer an order of magnitude improvement over the competing solutions for at least one measurement of importance.

In addition to the risk-aversion that every one of us has to some degree, the expectation for the software to be based on proven technologies has a rational explanation as well. The behavior of the so-called ‘proven’ technologies is quite well-known thanks to them having large user bases. The more users, the more use cases were thrown at the technologies, thus most of the edge cases have already been discovered and accounted for. Discovering such edge cases comes hand-in-hand with documenting them, hence ‘proven’ technologies are also proven in a way that they have a larger body of knowledge explaining peculiarities of their behaviors and configurations.

## Conclusion and Outlook

Sometimes directly, sometimes not, the outlined expectations determine the success of the third-party software solutions. When designing and implementing a solution, one needs to keep them in mind. Even though these expectations might seem to be not actionable, the ability to meet them determines the long-term success of the product. Therefore, when building a software product, one will profit by passing it through a filter of these expectations from time to time.

Although the presented list is in no way exhaustive, it is a starting point for the discussion on how the expectations translate into the technical requirements to the functionality and characteristics of how the functions are performed.

Tell me what is your take on the expectations from the third-party solutions! If you want me to dive into a related topic, feel free to reach out or leave a comment.
