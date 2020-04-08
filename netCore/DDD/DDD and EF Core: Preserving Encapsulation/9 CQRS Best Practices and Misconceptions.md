# 9 CQRS Best Practices and Misconceptions

## CQRS and Event Sourcing

Event Sourcing is very complex
Event Sourcing needs CQRS

## Evolutionary Design

CQRS should bound to one BC

## Using Commands and Queries from Handlers

Normaly No, Query from Query is possible but normally not needed

Clients Should Trigger Commands !

## One-way Commands

Commands should never return,

Run the Commend Synchronously
Return a Locator(Id)

## CQRS vs. the Specification Pattern

specification => DRY
CQRS => Loose coupling (Mostly Wins)










