# 4 Refactoring Towards a Task-based Interface

## CRUD-based Interface

Create,Read,Update,Delete 

*   Not best way to structuring the application
*   Can have bad effects

Problems
*   Growth of complexity
*   Lack of ubiquitous language
*   Damaging of UX (Hard to build a proper mental model)

## Task-based Interface

Each windows dows one thing

## Untangling the Update Method

=> Task Centric

Update becomes many Tasks

=> Every Task gets this own Dto: Enrollment Dto, TransferDto ... 

Cyclomatic complexity corresponds to line indentation

Avoid DTO's which are full of holes !


## Task based user interface

=> Simpler Interface 


## Dealing with Create and Delete Methods

=> Dto for creation

Operation named after CRUD Terms not so good

Create => Register Student
Delete => Unregister Student


