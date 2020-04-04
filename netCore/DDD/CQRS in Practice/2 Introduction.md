# 2 Introduction

## CQRS and Its Origins

Command and Query Responsibility Segregation

ReadModel  
WriteModel

CQS 
*   Command: Side Effects => void
*   Queries: No Side effects => non void

Not every time possible i.e. stack.Pop()
Problems with atomic operations

## Why CQRS

*   Scalability
*   Performance (ORM for write, pure sql for read)
*   Simplicity

CQRS is SRP on architectural level

## CQRS in the Real World

ORM for Writing, ADO.NET for reading  
elasticsearch












