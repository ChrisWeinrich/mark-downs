# 5 Learning and Implementing HATEOS
## Hypermedia as the Engine of Application State

Hypermedia drives how to consume and use the API

```jSON
{ ...
    "links2": [
        {
            "HREF": "",
            "Rel":"",
            "method":"Delete"            
        }
    ]
}
```

## Demo Introduction- Supporting HATEOS

Static vs Dynamic

## Demo - Implementing HATEOAS Support for a Single Resource

Use URL Helper

Give Routes a Name to use in URL Helper

## Demo - Implementing HATEOAS Support after posting

## Demo - Implementing HATEOAS Support for a Collection Resource

## Using HATEOAS for Pagination Links

next and previous page should be in the links Attribute

## Demo - Working Towards Self-discoverability with a Root Document

```C#
    [Route("api")]
    [ApiController]
    public class RootController : ControllerBase
    {
        [HttpGet(Name = "GetRoot")]
        public IActionResult GetRoot()
        {  
            // create links for root
            var links = new List<LinkDto>();

            links.Add(
              new LinkDto(Url.Link("GetRoot", new { }),
              "self",
              "GET"));

            links.Add(
              new LinkDto(Url.Link("GetAuthors", new { }),
              "authors",
              "GET"));

            links.Add(
              new LinkDto(Url.Link("CreateAuthor", new { }),
              "create_author",
              "POST"));

            return Ok(links);

        }
    }

```

## Other Approaches and Options

- Json-LD
- Json-API
- OData