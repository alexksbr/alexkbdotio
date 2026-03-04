+++
title = "Network Communication And Leaky Abstractions"
date = 2025-02-18T06:01:23+00:00
draft = false
summary = "Why synchronous network calls are leaky abstractions and how asynchronous, indirect communication improves resilience."
tags = ["distributed-systems", "communication", "resilience", "architecture"]
categories = ["Software Architecture"]
+++

Method and function calls are simple, right? You call a function by its name and pass in parameters, then the function returns - everything happens in a clearly defined order. However, things get more complicated in distributed systems. Imagine Service A calling Service B *synchronously* and *directly* over a network:

![Synchronous direct call](/img/essays/network-communication-and-leaky-abstractions/synchronous-direct-call.png)

Service A sends a request (1) and then waits for Service B's response. Service B processes the request (2) and sends back a response (3). Then Service A receives the response and resumes (4). In a local call, the CPU jumps directly to the function's memory address. Over a network, though, messages travel over unreliable links. They might be delayed, dropped, or even lost if Service B is down.

Synchronous network calls mask network issues by mimicking local procedure calls. This is a leaky abstraction: the network's unreliability eventually shows up in your code. If Service B is unavailable or slow, Service A must handle retries, timeouts, or implement circuit breakers. This adds complexity and creates tight runtime coupling between services.

Instead, *asynchronous* and *indirect* communication makes network properties explicit. With asynchronous communication, Service A sends a request and continues processing other tasks, handling the response later through a callback. Indirect communication means that neither the caller nor the receiver is directly connected; they interact through an intermediary, such as a message queue or append-only storage.

To understand why network behavior becomes more explicit in this communication style, look at the assumptions Service A must make.

For *synchronous* and *direct* communication:

- "I know that Service B exists"
- "I know where Service B is located so I can directly contact it"
- "I will get a response from Service B in time"

If any of these assumptions fail, Service A must deal with retries, timeouts, and fallback behavior. Network failures leak into Service A's implementation, increasing its complexity and coupling to Service B.

Now compare this with *asynchronous* and *indirect* communication, specifically request/reply messaging:

![Asynchronous indirect request/reply](/img/essays/network-communication-and-leaky-abstractions/async-indirect-request-reply.png)

Assumptions become simpler:

- "I send a message to Message Queue 1 and someone will handle it."
- "Eventually, I will receive a response on Message Queue 2 which my callback will handle."

There are fewer assumptions that leak network failure modes into Service A. Since Service A already handles responses *eventually*, network unreliability is built into the communication design.

Another important benefit is reduced temporal coupling.

## (De-)coupling in Time

Think about cooking soup. Some tasks can happen simultaneously, but other steps must happen in sequence. Software systems are similar. Some parts must run before others, which creates *temporal coupling*.

Synchronous communication over networks always introduces temporal coupling: when Service A calls Service B, Service A depends on Service B's timing and availability.

Using asynchronous communication helps services operate independently in time. Consider an order service that checks inventory before finalizing an order.

Synchronous style:

```javascript
function processOrder(order) {
  // Synchronous call: wait for the response
  const inventoryStatus = inventoryService.checkAvailability(order.items);

  if (inventoryStatus.available) {
    console.log('Inventory available, processing order.');
    // Continue processing the order
  } else {
    console.log('Item out-of-stock, notifying customer.');
    // Handle out-of-stock scenario
  }
}
```

Asynchronous style with request and response queues:

![Order and inventory with queues](/img/essays/network-communication-and-leaky-abstractions/order-inventory-queues.png)

```javascript
function processOrder(order) {
  // Asynchronous call: continue processing without waiting
  inventoryService.checkAvailabilityAsync(order.items, (inventoryStatus) => {
    if (inventoryStatus.available) {
      console.log('Inventory available, processing order.');
      // Continue processing the order
    } else {
      console.log('Item out-of-stock, notifying customer.');
      // Handle out-of-stock scenario
    }
  });

  // Other independent tasks can be performed here without waiting
  console.log('Order received; processing will continue once inventory is confirmed.');
}
```

The second version has lower temporal coupling: the order service is not blocked on inventory availability and timing.

## Drawbacks of Asynchronous Communication

Asynchronous communication has trade-offs too, including message ordering complexity, harder debugging, and eventual consistency concerns.

This is not a claim that every interaction should be asynchronous. A practical rule of thumb is to use asynchronous, indirect communication across bounded contexts and be more open to synchronous, direct communication within a bounded context.

By designing with asynchronous, indirect communication where it fits, systems become more robust, scalable, and adaptable to network unreliability.

Original publication: [Shaping Shifts](https://www.shapingshifts.com/p/network-communication-and-leaky-abstractions)
