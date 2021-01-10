# 3 Structuring and Implementing the Outer Facing Contract

## Structuring Our Outer Facing Contract

* Resource Identifier
* Http Method
* Payload

### Resource Naming Guidelines

*   Things,not actions
*   Choosing Nouns
*   Pluralize
*   Consistent
*   Hierarchy: api/author/{id}/courses
*   Filters, sorting orders aren't resources
*   Be pragmatic with RPC style calls

## Demo - Structuring Our Outer Facing Contract

```C#
[ApiController]
    [Route("api/authors")]
    public class AuthorsController : ControllerBase
    {
        private readonly ICourseLibraryRepository _courseLibraryRepository;

        public AuthorsController(ICourseLibraryRepository courseLibraryRepository)
        {
            _courseLibraryRepository = courseLibraryRepository ??
                throw new ArgumentNullException(nameof(courseLibraryRepository));
        }

        [HttpGet()]
        public IActionResult GetAuthors()
        {
            var authorsFromRepo = _courseLibraryRepository.GetAuthors();
            return Ok(authorsFromRepo);
        }

        [HttpGet("{authorId}")]
        public IActionResult GetAuthor(Guid authorId)
        {
            var authorFromRepo = _courseLibraryRepository.GetAuthor(authorId);

            if (authorFromRepo == null)
            {
                return NotFound();
            }
             
            return Ok(authorFromRepo);
        }
    }
```

## Working with Endpoint Routing

app.UseRouting();
app.UseEndpoints();

## The Importance of Status Codes

*   200 Success
    *   200 OK
    *   201 Created
    *   204 No Content
*   400 Client Mistakes
    *   400 Bad Request
    *   401 Unauthorized
    *   403 Forbidden
    *   404 Not Found
    *   405 Method not allowed
    *   406 Not acceptable
    *   409 Conflict
    *   415 Unsupported Media Type
    *   422 Unprocessable entity
*    500 Server Mistakes
    * 500 Internal server error

## Error vs Faults

Errors => 400 
Faults => 500

## Enhancing Responses with Problem Details

https://tools.ietf.org/html/rfc7807

## Content Negotiation

Best Representation for a given response

### Output Formater

Media Type via Accept Header

- application/json
- application/xml
- ...

Always include accept header !

### Input Formatter

Deals with input

Media Type via content-type header

```C#
services.AddControllers(setupAction =>
{
    setupAction.ReturnHttpNotAcceptable = true;
    
}).AddXmlDataContractSerializerFormatters();
```















