# Introduction

## Anemic Domain Model

Separates Data and Operations

### Reasons Against
*   Discoverability
*   Duplication
*   Lack of Encapsulation

### Encapsulation

Act of protecting the data integrity
*   Information Hiding
*   Bundling data and operations together
*   no Operation should be able to violate invariants

In OOP, an invariant is a set of assertions that must always hold true during the life of an object for the program to be valid. It should hold true from the end of the constructor to the start of the destructor whenever the object is not currently executing a method that changes its state.

### Anemic Domain Model and Encapsulation

*   no way to retain invariants  
*   no restrictions  
*   potential duplication

### Advantages of Anemic Domain Model

*   Intuitive
*   Easy to implement

### Anemic Domain Model and Functional Programming

*   Immutable data structures
*   Operations upon data kept separately

In this case it is Correct, cause you don't worry about internal state