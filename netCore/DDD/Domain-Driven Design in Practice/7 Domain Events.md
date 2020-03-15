# Domain Events

## Introducing new bounded Contexts

As little Coupling as Possible between Bounded Contexts

## Domain Events

Event + Domain significance

*   Decouple Bounded Contexts
*   Communication between BCs
*   Decouple entities inside BC

### GuidLines
*   Naming => Past Tense i.e. BalanceChangedEvent
*   Include as little Data as possible
*   Don't use Domain Classes in Events
*   Use Primitives in Events

### Physical Delivery

*   Inner-Memory Structures
*   Service Bus

### Classic Approach
*   Domain Events Class which Holds all Handlers
*   When a Event is Raised It Scans Collection of handlers and Execute correct ones

### Modern Approach
*   Seperate creating an Event and Dispatching Event
*   Domain Entities Creates Event
*   Move Dispatching to unit Of Work Save Changes
*   Dispatching should be done after persisted Changes !

### Between Microservices
=> EsbGateway

### ORM
=> Read Database directly into DTOs !!!

