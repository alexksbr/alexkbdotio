+++
title = "A Case Against 'One Model to Rule Them All'"
date = 2024-10-07T05:01:21+00:00
draft = false
summary = "Why a single canonical model creates coupling, bottlenecks, and ambiguity across teams."
tags = ["domain-driven-design", "bounded-contexts", "architecture", "modelling"]
categories = ["Software Architecture"]
+++

Vienna has a population of around 2 million people, including me. A [2019 report](https://www.wienerlinien.at/media/files/2020/wl_betriebsangaben_2019_deutsch_358274.pdf) by "Wiener Linien," Vienna's main transit provider, says its subway moves nearly 1.3 million people daily. Below, you see a satellite picture of Vienna and a depiction of its subway line network on the right.

![Figure](/img/essays/a-case-against-one-model-to-rule-them-all/vienna-viewpoints.png)

What's interesting is that both portray a specific viewpoint of Vienna. The subway network leaves out many details about the city, like which buildings there are or how the street network looks. Meanwhile, it magnifies other aspects, particularly the subway stations and their connections. It fulfills a specific purpose: showing passengers how to get from Station A to Station B.

This is the primary purpose of a *model*: to omit certain details while emphasizing others. What to leave out and what to include depends on the purpose of the discussion.

## Models in Software

Software engineering is a discipline that relies significantly on models, encompassing model-driven practices in embedded systems and data modeling along with UI mocking in information systems. For the latter, a popular and widely used modeling technique is *Domain-Driven Design*. With Domain-Driven Design, you use the business domain model as the software's primary internal model. Using the subway network model as the software's business domain would result in entities like station, rail, or passenger, which are used by domain experts and within the software's code. Domain-Driven Design calls this a *ubiquitous language*.

## One Model to Rule Them All?

The subway network plan from above is not the only one used by passengers and employees of Wiener Linien. Here are two more:

![Figure](/img/essays/a-case-against-one-model-to-rule-them-all/subway-network-detail.png)

![Figure](/img/essays/a-case-against-one-model-to-rule-them-all/station-plan.png)

The plan f adds information about the stations' geographical locality and how the rails are positioned between them. The right plan zooms in on a specific station. It still has information about crossing subway lines but adds context about the station, such as stories and walkways between subway lines.

What's interesting is that all these pictures represent *models* of the same domain: the subway network of Vienna. No model is more useful than the others. The usefulness of a particular model depends on who uses it and how. The model in Figure 1 helps those who design subway timetables. The above plan in Figure 2 might suit city planners better. Employees who design subway stations likely use the model of a subway station from Figure 3, e.g., to optimize walkways between subway lines.

Now, we run into a problem with the ubiquitous language. Imagine the concept of a *rail*. It exists in all models above but carries a different meaning that varies slightly between them. This makes using a ubiquitous language across all models very challenging. The model from Figure 1 might use the length of the rail between stations and the maximum allowed speed to calculate the trip duration between two stations. The geographically more accurate model from Figure 2 might show the exact degree of bending at each point in the network. It could also include the cost per unit, especially if city planners use it. The station model from Figure 3 also depicts different aspects of the rail, like its width or how they are equipped with power supply mechanisms for the trains.

In software systems, we often create a common (canonic) data model. It includes a universal entity of rail with all the above attributes. You may end up with a *rail module* or* rail service*, which all other modules (like a city planning module) poll for their specific data. I think this is an anti-pattern. The situation is not only unfavorable due to the modules needing to poll all the data and filter out their relevant information; it also results in two significant problems.

First, it decreases the long-term evolvability of the involved modules by introducing increased coordination effort, especially if different teams maintain the involved modules. Imagine a team in charge of a *city planning module* implementing a new feature requiring additional fields in the *rail entity*. They now have to go to the team responsible for the *rail module* and request to introduce those fields. After adding the new fields, the *rail module* team must return to the *city planning module* team to announce that they made the required changes. Ultimately, the *rail module* team must deploy their changes before the *city planning* team can deploy theirs.

Further, many teams develop features that use the concept of rails. So, the team that maintains the *rail module* will receive many change requests. This will create a bottleneck and reduce all the teams' ability to act independently. Also, imagine the ripple effects going through the entire system when the *rail module* changes its API.

The second problem is that each module uses the concept of rails slightly differently, making different assumptions and invariants about it. Then, they write data back to the *rail module*, which is now augmented with those assumptions and invariants. This introduces bugs that are very hard to find and require a huge coordination effort to fix.

## Module Boundaries

Domain-Driven Design approaches this in a different way. It breaks up the ubiquitous language into *bounded contexts*. Each bounded context contains its **own** ubiquitous language and stores its **own** data. In our example, all bounded contexts contain the entity *rail* with its context-specific meaning and stores its context-specific representation in its data store.

![Figure](/img/essays/a-case-against-one-model-to-rule-them-all/bounded-contexts.png)

Now, within the bounded context of *city planning*, *rail* has a specific meaning: a piece of rail track with exact degrees of bending and associated costs. Meanwhile, the other bounded contexts have their own definitions of *rail *they use internally.

This reduces the misuse of overloaded domain terms. It also enforces a more model-specific language within a bounded context, reducing ambiguity.

## Methods To Design Bounded Contexts

Coming up with Bounded Contexts is not straightforward. It is an iterative design activity that probably starts out messy. This is entirely normal. You will not find perfect bounded contexts on the first go. It is an iterative process, requiring lots of different domain knowledge (and thus many different domain experts). That's why collaborative modeling tools are a great fit.

One of these tools is [Event Storming](https://leanpub.com/introducing_eventstorming). It is best performed on-site and requires several meters of wall space to hang different-colored post-its. Several domain experts brainstorm things like domain events, commands, and actors, bringing them into a coherent time-bound flow from left to right. Participants then use this mishmash of post-its to draw bounded contexts.

Another one is [Domain Storytelling](https://www.goodreads.com/en/book/show/58385794-domain-storytelling), in which domain experts model activities within the domain as *domain stories*, which are mapped on a visual map to derive bounded context.

The [Domain-driven Design Starter Modeling Process](https://github.com/ddd-crew/ddd-starter-modelling-process?tab=readme-ov-file#discover) lists further techniques if you are interested.

One very important thing is that a bounded context must be small enough to be clearly assignable to a development team. Don't spread the responsibility of bounded contexts over multiple teams; this only leads to unclear responsibility and makes it harder to establish a ubiquitous language within the bounded context because more people are involved. If you have a bounded context too large for a single team, consider splitting it.

## Conclusion

Understanding Bounded Contexts requires intuition of the concept. Once you grasp it, it is a powerful tool to help you design evolvable software. However, maintaining different representations of data distributed over multiple bounded contexts comes with challenges, e.g., data consistency and designing an eventually consistent system. Keep these considerations in mind and find the sweet spot in the resulting trade-offs when architecting such a system.

Original publication: [Shaping Shifts](https://www.shapingshifts.com/p/a-case-against-one-model-to-rule)
