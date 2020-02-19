# 6 Introducing Second Bounded Context

## New Task ATM Model

*   Dispense Cash
*   Charge Users Bank Card
*   Keep Track of charged money

## Bounded Context

Separation of model into smaller ones

*   Boundary for ubiquitous language
*   Span Across on all layers
*   Explicit Relationships between different bounded contexts

## Bounded Context and Sub-domains

*   Sub Domain is part of Problem Space
*   Bounded Context is part of Solution Space
*   Mostly 1on1 Relationship

## Choosing Boundaries for Bounded Contexts

*   Team "6-8 developers"
* Code size "fit your head"

## Drawing a context map

![alt text][sharedKernel]

[sharedKernel]: sharedKernel.png "Shared Kernel"

## Types of physical isolation

*   Same Assemblies and shared Database instance
*   Separate Assemblies under same Solution
*   Separate Deployments => micro services

## Communication between bounded contexts

* Without Anti Corruption Layer
    * Direct
    * Domain events
* With Anti Corruption Layer
    * Proxy
* Separate Processes => Micro services

## Code reuse

Should Be serious thinking about it ...
Avoid Domain Classes

## Implementing ATM Domain Logic



