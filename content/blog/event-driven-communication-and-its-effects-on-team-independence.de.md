+++
title = "Event-Driven Communication and Its Effects on Team Independence"
date = 2024-09-30T00:00:00+02:00
draft = false
summary = "How event-driven communication reduces coupling and shifts responsibility to the right team boundaries."
tags = ["event-driven", "architecture", "team-topologies", "integration"]
categories = ["Software Architecture"]
+++

I am an avid reader of The New Yorker magazine. For several years, I read the magazine every week and bought it whenever I passed a bookstore at a train station or airport. Sometimes, I was so eager for great journalism that I traveled 30 minutes to my local bookstore to get the latest issue.

When The New Yorker expanded its subscription offering to Europe, I immediately subscribed and paid the full year's price. I felt comfortable knowing the latest issue would be delivered to my doorstep. I would no longer have to rummage in airport bookstores or travel an hour back and forth to get the newest magazine. I delegated all this work to The New Yorker's subscription office.

This is what the publish-subscribe pattern does. Consumers should not have to remember to buy the latest magazine every Monday. Having to organize myself to get the latest issue every week resembled a form of direct communication. With request-reply or command patterns, modules call other modules and ask for data or issue commands they want them to carry out. Imagine an order service receiving a customer order and then calling a shipping service to trigger the shipment of the order.

Why does the order service have to know about shipment at all?

Using the publish-subscribe pattern, the order service would publish order information, and the shipping service would subscribe to it. Now, the order service does not need to know anything about the shipping service, and vice versa. The shipping service just needs to know that completed orders are relevant to its business domain and that it has to act upon them.

This pattern changes the behavior of the system's modules. But before we explore this further, let us look into what information modules exchange.

## Module Interaction

Every time a module interacts with another, it sends a message, such as a method call, an HTTP request, or a request sent to a message queue.

![Module interaction overview](/img/essays/event-driven-communication/module-interaction.png)

The content of this message can be as different as the behavior it triggers in the recipients. There are three types of messages: commands, queries, and events.

Commands trigger the recipient to modify state or execute behavior. Examples of commands are "create customer" or "delete order". Most of the time, the recipient returns a reply. This can be a synchronous reply or a separate response in the case of asynchronous communication.

Queries always induce a reply. A query retrieves data from the receiving module, like "get all new customers over the last month".

Events are statements of facts that occurred. The emitting module releases an event, letting others know something happened, like "new order placed" or "order canceled". They never induce a direct reply because replying to statements of facts would not make much sense. Events have an effect that commands or queries do not. The producer has no knowledge of the receiving modules, their behavior, or what they do with the data.

It releases the event and considers its job done, reducing the coupling between modules. Events are often used in combination with the publish-subscribe pattern. Usually, any module interested in a specific type of event can subscribe to it.

![Commands, queries, and events](/img/essays/event-driven-communication/message-types.png)

## Using Events

Events themselves can lead to different outcomes in receiving modules.

The first is a notification, an event that triggers behavior in downstream modules. Imagine a sports game module notifying receiving modules that a particular game has ended. The event may have no significant data. It may only contain information about an API where receiving modules can query the data they need.

Be careful because you can easily fall into the trap of misusing event notifications as disguised commands. I have seen events such as "order creation triggered", which are essentially "create order" commands. Events should not introduce assumptions about receiving modules within the producing module. A module emitting an event should not know that it will trigger functionality in a receiving module.

The second form is using events as a form of state transfer. Here, each module has its own local representation of the state it needs. Events then carry information about state changes, and modules can project this onto their own data representation. An "order placed" event could carry all needed order information. It would let a shipping module change its internal state and trigger a delivery.

![Event usage patterns](/img/essays/event-driven-communication/event-usage-patterns.png)

Finally, you can use events by treating internal state changes as events and writing them to an event store. We call this event sourcing. The event store is an append-only log. You can then build the current state by aggregating events in the store. Event-sourced modules often use a database to store this aggregated state, which avoids querying the entire event sequence for every request.

![Event-driven state transfer example](/img/essays/event-driven-communication/state-transfer-example.png)

## Events and Maintainability

How you use events to convey information has an essential effect on your system's maintainability because of the decoupling you can achieve. Events move data from one module to another. Now, the receiving module performs operations on that data instead of the producing module. A request-response mechanism turns this relationship completely upside down. It means the producer has to trigger something at the consumer over an interface.

So, the producer has to have knowledge about the consumer and the behaviors it provides. Therefore, the domain knowledge within both services is intertwined. One has to know what the other modules want and provide a public interface, and the other has to know when it should call that interface.

Event mechanisms, particularly event-driven state transfer, effectively shift the responsibility downstream to the shipping service. Now, the shipping service is tasked with determining when to trigger shipments and under what conditions to do so. This reallocation of responsibility aligns with the shipping service's domain, making it a more logical choice for this task.

That's why event mechanisms are often used for cross-team communication across value streams:

> Event-driven communication works incredibly well between value streams. It is a means to decouple services as it reduces request-response and point-to-point integration. However, it is hard to implement across organizations and often is overkill for communication within team responsibilities. It works well when communication across value streams is implemented by using asynchronous communication like events.

Teams within a value stream may use events but can also use direct point-to-point communication or other integration patterns.

## Drawbacks

Everything is a trade-off. Event-driven communication often lacks a central mechanism for coordinating workflows, making systems more difficult to track and observe. It's hard to reason about an event-driven system without seeing the flow of events and control.

Another important point is eventual consistency. Emitted events do not immediately change state everywhere within a single transaction. They also do not send a reply to the producer indicating that the event was processed. Therefore, you have to design your system with eventual consistency in mind.

## Conclusion

Event-driven communication does not mean you need to implement every communication channel between each service with event-driven state transfer. Consider places where events fit well, often between teams and across independent value streams. Many organizations are happy with event-driven communication across teams or bounded contexts and more direct communication within bounded contexts.

## Further Reading

- [What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html) by Martin Fowler
- [Designing Data-Intensive Applications](https://www.goodreads.com/book/show/23463279-designing-data-intensive-applications) by Martin Kleppmann
- [Learning Domain-Driven Design: Aligning Software Architecture and Business Strategy](https://www.goodreads.com/book/show/57573212-learning-domain-driven-design) by Vlad Khononov

Original publication: [Shaping Shifts](https://www.shapingshifts.com/p/event-driven-communication-and-its)
