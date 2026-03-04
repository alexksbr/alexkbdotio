+++
title = "Build Adaptable and Resilient Software Systems by Unlocking Component Autonomy"
date = 2025-02-25T06:00:57+00:00
draft = false
summary = "How component autonomy across functional, data, runtime, activation, scaling, deployment, and solution dimensions improves long-term evolvability and reliability."
tags = ["software-architecture", "autonomy", "resilience", "evolvability"]
categories = ["Software Architecture"]
+++

I regularly perform architecture reviews across different companies. Most stakeholders want their systems to be evolvable, adaptable, and reliable over the long term. One particular thing I often observe hindering this goal is a lack of autonomy. Let me explain.

When a system's components become too interdependent, even a small change can have wide-ranging effects. For example, if a modification in one component triggers changes in several others, it signals architectural entanglement. This issue is especially problematic when different teams are responsible for different components, because any change requires extensive communication and coordination across teams, disrupting priorities and backlogs. Such complexity not only increases technical overhead but also adds organizational and communication burdens, ultimately resulting in longer lead times and a higher likelihood of errors.

To mitigate these challenges, it is essential to design systems where each component is as autonomous as possible. This lays the groundwork for long-term adaptability, evolvability, and reliability.

## What is Autonomy?

Autonomy comes from the Greek words *autos* (self) and *nomos* (law). It refers to an entity's ability to make its own decisions and govern its actions independently. In other words, when an entity is autonomous, the decisions of others do not directly affect its behavior. For example, if one entity acts and another is *forced* to respond *immediately*, then at least one of them is not fully autonomous. (Strong emphasis on the words "forced" and "immediately". If another entity has chosen to react and can do so eventually and on their own terms, it is still autonomous.)

In software architecture, autonomy means that a system's component can operate and evolve independently, reducing the need for frequent (inter-team) coordination.

Autonomy is not binary; it exists on a spectrum from total dependency to complete independence. Instead of viewing autonomy as something you either have or do not have, individual entities should always strive to become more autonomous. Therefore, if your team is working on a component that ranks low on the scale, you should seek ways to increase its autonomy rather than accepting its dependency.

Let's consider the following example: Component A is called by an orchestrator. Whenever the orchestrator deems it necessary to call Component A, it will decide to do so (1). Once invoked, Component A contacts Component C because it needs C to update its internal state (2). Component A can only report back after confirming that Component C's task is complete. In addition, Component A reads data from a data schema that it shares with Component B (3).

![Component autonomy baseline](/img/essays/build-adaptable-and-resilient-software/0ccdec7e-1960-4873-9ff4-b23852155da8_1357x513.png)

How autonomous is Component A? Not very much. Consider these issues:

- Component A does not decide when to be invoked; the orchestrator makes that decision.
- Component A depends on Component C being available to respond to the orchestrator.
- Component A can only scale to the degree that Component C can.
- If Component B changes the shared data schema, Component A's code must be adapted and redeployed.

## Aspects of Autonomy

It is not only the case that autonomy is not binary, it is also multi-dimensional. Rather than being defined by a single attribute, autonomy encompasses multiple aspects that capture its various facets.

### Functional Autonomy

Components must have clear responsibilities and well-defined boundaries that separate them from one another. When these boundaries are fuzzy or unclear, components end up sharing responsibilities, which means that changes in one component often force changes in others.

![Functional autonomy examples](/img/essays/build-adaptable-and-resilient-software/d1b728be-8068-4c7f-bab8-ea4fe961ebd7_1402x958.png)

The example on the left shows low functional autonomy. Component B is shared across contexts, meaning that both rely on the same domain model contained within the component. In domain-driven design, this is called a shared kernel. Although shared kernels can be advantageous in some cases, they inherently hinder functional autonomy. This is especially problematic when two different teams are responsible for the two bounded contexts. In such situations, implementing shared kernels requires high communication bandwidth between teams, a significant drawback if the teams are separated by a large organizational distance (I have written about this in [Connecting Modules Across Teams: Selecting Integration Mechanisms Based on Team Distance](https://www.shapingshifts.com/p/connecting-teams-and-modules-selecting)).

The example on the right has clear boundaries of responsibility. In this scenario, the shared Component B is replaced by an explicit integration point between the two bounded contexts, resulting in clear boundaries of responsibility.

Here are some strategies to increase functional autonomy:

- Components should expose only what is necessary through explicit interfaces while keeping internal details hidden. Interaction between components should occur solely through these interfaces, with no side channels. This interface might take the form of an API or use an intermediary such as a message queue. If the integration crosses deployment boundaries, I recommend using asynchronous and indirect communication, as I argued in [Network Communication and Leaky Abstractions](https://www.shapingshifts.com/p/network-communication-and-leaky-abstractions).
- A component should be as small as necessary to fulfill its purpose. This idea aligns with the Unix philosophy of "do one thing and do it well" and is reflected in the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).

### Data Autonomy

Shared data often undermine a component's autonomy, especially when components rely on a common, mutable data schema. When multiple components depend on a shared schema, any changes to that schema force every dependent component to adapt. To enhance data autonomy, ensure that each component manages its internal state independently, without relying on shared data.

One important caveat is that shared data does not harm autonomy if it is **immutable**. For example, if several components read from an append-only storage, where existing data cannot be altered and new data can only be appended, then data autonomy is maintained. In summary, avoid using shared mutable data or schemas to ensure each component can operate independently.

### Runtime Autonomy

Consider two scenarios involving separately deployed services that communicate over a network: In the example on the left, the order service makes a synchronous call to the inventory service before approving an order. This means the order is approved only after the inventory has reserved the products. In this case, the order service needs the inventory service to be available during its operation, resulting in lower runtime autonomy.

In the example on the right, the order service informs the inventory service about an incoming order asynchronously, using an event, for example. Once the event is committed to the middleware, the order service continues processing without blocking and waiting for a response. If inventory is insufficient, the inventory service can asynchronously notify the order service, which then decides how to proceed. Here, the order service does not depend on the inventory service's availability for the transaction, which significantly increases its runtime autonomy.

![Runtime autonomy examples](/img/essays/build-adaptable-and-resilient-software/a8d0eee9-34db-47d4-95f4-0e45aff84f02_1307x585.png)

You can increase runtime autonomy by applying a few strategies:

- **Deploy Components Independently:** Ensure components run as separate processes. If components share the same process, they inherently lack runtime autonomy.
- Decouple the runtime behavior of components by employing **asynchronous communication** such as notifications, state transfers, or event sourcing. I have detailed these approaches in [Event-Driven Communication and Its Effects on Team Independence](https://www.shapingshifts.com/p/event-driven-communication-and-its).
- **Handle Responses Optimistically:** Avoid waiting for immediate answers from dependent services. Instead, resolve issues optimistically. For example, if the inventory is insufficient, the system might automatically trigger a restock or, alternatively, the order service could notify the user of a potential delay.

### Activation Autonomy

High activation autonomy means that a component should determine on its own when to act rather than relying on other components to decide when it is invoked. In other words, **only** the component itself should know when specific actions need to be performed.

This approach contrasts with orchestration patterns, where an orchestrator is responsible for triggering actions in other components. In systems designed with high activation autonomy, each component reacts to environmental cues and decides independently whether to take action.

Consider an example: suppose you need to build a workflow where bookings over $1,000 must be reported. One component processes the bookings, and a separate component handles reporting.

![Activation autonomy examples](/img/essays/build-adaptable-and-resilient-software/cc5491fe-dee5-45f0-bd4d-4ec57ac9d144_1386x551.png)

**Low Activation Autonomy (Left Example):**

Here, the reporting component depends on both its internal logic and on an external orchestrator to determine when it should run. As a result, any change in the activation logic requires modifying both the reporting component and the orchestrator.

**High Activation Autonomy (Right Example):**

In the right example, all the knowledge about reporting sits within the reporting component. There is no need for coordination and one singular point of change. The difference is that communication over self-contained facts that are modeled after the domain (events) gives the receiving component the power to decide whether they need to take action. The activation logic is isolated within the component instead of distributed over multiple components, giving it activation autonomy.

### Scaling Autonomy

In [Beyond Linear Scaling: How Software Systems Behave Under Load](https://www.shapingshifts.com/p/beyond-linear-scaling-how-software), I discussed factors that limit the scalability of components. Two factors are particularly relevant in this context: The first are bottlenecks or resources shared with other components, which can inhibit each component's throughput. Another is the additional work that a system needs to perform to maintain consistency and coherency. As I wrote in the article:

> In certain systems, adding more load can actually decrease throughput. Let's examine the following system:

![Scaling autonomy example](/img/essays/build-adaptable-and-resilient-software/dcbd79a4-600c-4f72-b7b9-bad03a26d685_830x452.png)

> The three subtasks each have their own data storage, either persistently through separate data stores or ephemerally via caches. Let's assume the system must maintain consistency to respond correctly. Imagine one subtask updating data in its own storage; the system must then propagate these changes to other storages using a transaction.
>
> This has significant consequences. Not only do the subtasks need to stop responding until a consistent state is restored, but the process of re-establishing this state requires additional work before the system can resume service. This overhead negatively affects throughput: as more load is added, the throughput actually decreases, a phenomenon often observed in real-world throughput metrics.

If a component depends on other components to scale up its throughput, whether through resource contention, bottlenecks, or extra coherency work, it exhibits low scaling autonomy.

Here are strategies to increase scaling autonomy:

- Lower resource contention by providing sufficient resources and enabling components to acquire the resources they need through self-service interfaces (for example, using cloud infrastructure).
- Minimize coherency overhead by avoiding strict consistency across components when possible, as doing so introduces extra work and tightly couples components' scaling behaviors.

### Deployment Autonomy

High deployment autonomy is evident when teams can deploy components independently without impacting other deployments. For example, if deploying Component A requires a team to stop, restart, or redeploy Component B, the system exhibits low deployment autonomy. This low autonomy forces teams into frequent, error-prone coordination and hinders their ability to deploy updates on their own schedules.

### Solution Autonomy

One factor often overlooked when focusing solely on technical details is the autonomy of the team responsible for a component. In [Designing Effective Cross-Team Dynamics For Platform Teams](https://www.shapingshifts.com/p/designing-effective-cross-team-dynamics), I argued that team autonomy is one of the three pillars for building successful and motivated teams:

> Adopting a team-first perspective is essential for resolving these challenges. We need to focus on designing valuable and productive interactions between platform and application teams. A useful framework for this is outlined in Steven Pinker's book *Drive: The Surprising Truth About What Motivates Us*. Pinker identifies three key factors that motivate individuals and teams:
>
> Autonomy: The ability to make decisions, reflect on them, and adapt as needed.
>
> Mastery: The opportunity to perform work and receive feedback to improve skills.
>
> Purpose: The sense of working towards something achievable and worthwhile.
>
> [...]
>
> Autonomy
>
> Just as I prefer choosing my own route to a destination, autonomous teams need control over their solution space and the authority to make decisions within it. They must also be able to gather and incorporate both external and internal feedback, as well as prioritize their work within their areas of responsibility.

Solution autonomy describes the socio-technical dynamic in which teams can decide how to build, test, and deploy the components they are responsible for. This means fully empowering development teams to build each component using appropriate patterns, technologies, tools, and programming languages, rather than being constrained by a pre-defined set of rules imposed by external architecture or governance teams.

## Importance of Aspects

Not all aspects of autonomy carry the same weight; development teams must determine which are more or less important in their context. For example, runtime, scaling, and deployment autonomy are essential when prioritizing reliability, while functional, data, and solution autonomy are key to ensuring long-term evolvability.

It is important to consider all autonomy aspects because autonomous components can make your system more dynamic in the future. Make clear trade-off decisions only if certain aspects play a minor role in your context (which is rare). For instance, you might decide, "We trade off invocation autonomy for the significant benefit of monitoring orchestrated workflows." Keep in mind that such trade-offs can incur substantial costs in your system's evolvability and long-term adaptability across many dimensions. I have written about some of those dimensions in [Reactive Systems: Designing Systems for Multi-Dimensional Change](https://www.shapingshifts.com/p/reactive-systems-designing-systems).

## Improved Example

In the example at the beginning of the article, I outlined reasons for low autonomy. Having introduced some vocabulary, we can now map those issues to autonomy aspects:

- Component A does not decide when to be invoked; the orchestrator makes that decision. (**low activation autonomy**)
- Component A depends on Component C being available to respond to the orchestrator. (**low runtime autonomy**)
- Component A can only scale to the degree that Component C can. (**low scaling autonomy**)
- If Component B changes the shared data schema, Component A's code must be adapted and redeployed. (**low data and deployment autonomy**)

To improve the autonomy of both Component A and Component C, I made the following design adjustments:

![Improved architecture](/img/essays/build-adaptable-and-resilient-software/55b1cd90-4e1f-4436-bc4c-651ed78083d4_1339x549.png)

1. **Increase Activation Autonomy:** Component A now independently decides which system events warrant a reaction.
2. **Increase Runtime and Activation Autonomy:** Instead of waiting synchronously for a response from Component C, Component A now notifies C about the necessary changes and then handles responses asynchronously. This adjustment boosts Component A's runtime autonomy and also enhances Component C's activation autonomy.
3. **Increase Data and Deployment Autonomy:** Components A and B now maintain redundant data storage and separate data schemas. They communicate necessary state transitions through a message queue, which increases both components' data and deployment autonomy.

## Conclusion

Autonomy is not guaranteed. Autonomy is something you need to *buy*. And I mean that in the literal sense, because there is a cost. For example, achieving data autonomy might require additional storage for redundant data, and increasing activation autonomy could reduce the convenience of explicit, sequential monitoring. Every design decision involves trade-offs, and your task is to balance these within your context.

My advice is simple: increase the autonomy of each component as much as possible, and only compromise when absolutely necessary. I have seen too many systems become brittle over time because developers and architects failed to consider autonomy. If you aim for long-term adaptability, evolvability, and reliability, ensure you consider all autonomy aspects.

Original publication: [Shaping Shifts](https://www.shapingshifts.com/p/build-adaptable-and-resilient-software)
