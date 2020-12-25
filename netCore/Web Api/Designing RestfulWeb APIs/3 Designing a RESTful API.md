# 3 Designing a RESTful API

## Designing for REST

*   Can't fix an API after publishing
*   Too easy to add ad-hoc endpoints
*   Helps understand the requirements
*   Well designed API can mature

URIs are just path to resources  
Query Strings for non-data elements

Nouns are good, verbs are bad

/customers  
/products

...

### Resources
=> Entities ?

### Identifiers in URIs
/sites/1  
/sites/stone-henge  
...

### Query string

non-resource properties e.g. sorting,formating

## Designing Verbs

### Get
### Post
### Put
### Patch
### Delete

## Idempotency

Operation result in same sie effect   
Idempotent => Get, Put, Patch, Delete
Not Idempotent => Post

## Designing Result

- Shouldn't expose server details
- camelCase
- ar least be consistent

### Collections

- resultCount
- nextPageUrl 
- ...
- HypedMedia Route 


## Formatting Result

- Accept Header
- Content Type
- Query String for format is AntiPattern !

## Hypermedia

- Allows results to be self-describing
- Allows programmatic navigation
- Adds complexity !

``` json
...,
"_links : {
    "self": "/api/asdf/1",
    "region": ...,
    "...":"...,
    ...,
    }
```


















