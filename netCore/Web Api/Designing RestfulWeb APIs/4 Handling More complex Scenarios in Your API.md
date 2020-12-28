# 4 Handling More complex Scenarios in Your API

## Designing Associations

/api/customers/123/invoices  
/api/customers/123/payments  
/api/customers/123/shipments  

## Designing Paging

- lists should support paging
- Query strings are commonly used  
/api/site?page=1&page_size=25
- Use wrappers to imply paging
```json
{
    "totalResult":255,
    "nextPage: "...",
    "prevPage": "...",
    "results": []
}
```

## Error Handling
- Not just status codes
- Error info (Security problem)

## Designing Caching

Basic Tenet of REST APIs

Use Http for caching => 304 not modified

**Entity Tags**

412 not matched

## Functional APIs

not 100% RESTful

exception rather than the rule

## Asynchronous APIs

- SignalR
- gRPC
- Comet
- Firebase















