# Decoupling the Domain Model from Data Contracts

## Domain Model and Data Contracts

Shape (Data Contract) and Url cannot easily changed !

Not Couple Domain Models to Data Contract => Use Dto's !

## Extracting Output Data Contracts

Protect Domain Model !

Translation between Data Contract and Domain Model

Output: DTO
Input: Model or Request or DTO

Non Aggregate Roots as DTO's should not have Id, because they are bound to an Aggregate !

## Identifying a Security Issue

**Only include Properties which are needed for manipulation/Creation etc.**

## Extract Input Data Contract

