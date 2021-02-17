# 6 Improving Reliability with Advanced Content Negotiation

## Revisiting the Contract between Server and Client

- URI
- HTTP method
- payload

application/json

## HATEOAS and Content Negotation

problem links are not part of standard application/json

Accept: application/vnd.marvin.hateoas+json

- Vendor prefix: vnd
- Vendor identifier: marvin
- Meta type name: hateoas
- Suffix: json

## Demo: HATEOAS and Content Negotation

```C#

        [HttpGet("{authorId}", Name ="GetAuthor")]
        public IActionResult GetAuthor(Guid authorId, string fields,
              [FromHeader(Name = "Accept")] string mediaType)
        {
            if (!MediaTypeHeaderValue.TryParse(mediaType,
                out MediaTypeHeaderValue parsedMediaType))
            {
                return BadRequest();
            }

// Global Configuration of Media Types

    		services.Configure<MvcOptions>(config =>
            {
                var newtonsoftJsonOutputFormatter = config.OutputFormatters
                      .OfType<NewtonsoftJsonOutputFormatter>()?.FirstOrDefault();

                if (newtonsoftJsonOutputFormatter != null)
                {
                    newtonsoftJsonOutputFormatter.SupportedMediaTypes.Add("application/vnd.marvin.hateoas+json");
                }
            }); 
```

## Tightening the Contract Between Client and Server with Vendor-specific Media Types

Semantic Media Types
- "application/json" (Default)
- "application/vnd.marvin.author.full+json", 
- "application/vnd.marvin.author.full.hateoas+json",
- "application/vnd.marvin.author.friendly+json", 
- "application/vnd.marvin.author.friendly.hateoas+json"

## Demo: Tightening the Contract Between Client and Server with Vendor-specific Media Types

```C#
    // full author
    if (primaryMediaType == "vnd.marvin.author.full")
    {
        var fullResourceToReturn = _mapper.Map<AuthorFullDto>(authorFromRepo)
            .ShapeData(fields) as IDictionary<string, object>;

        if (includeLinks)
        {
            fullResourceToReturn.Add("links", links);
        }

        return Ok(fullResourceToReturn);
    }


         [Produces("application/json", 
            "application/vnd.marvin.hateoas+json",
            "application/vnd.marvin.author.full+json", 
            "application/vnd.marvin.author.full.hateoas+json",
            "application/vnd.marvin.author.friendly+json", 
            "application/vnd.marvin.author.friendly.hateoas+json")]
        [HttpGet("{authorId}", Name ="GetAuthor")]
        public IActionResult GetAuthor(Guid authorId, string fields,
              [FromHeader(Name = "Accept")] string mediaType)

```

## Demo - Working with Vendor-specific Media Types on Input

=> Action Constraint

```C#
    [AttributeUsage(AttributeTargets.All, Inherited = true, AllowMultiple = true)]
    public class RequestHeaderMatchesMediaTypeAttribute : Attribute, IActionConstraint
    {
        private readonly MediaTypeCollection _mediaTypes = new MediaTypeCollection();
        private readonly string _requestHeaderToMatch;

        public RequestHeaderMatchesMediaTypeAttribute(string requestHeaderToMatch,
            string mediaType, params string[] otherMediaTypes)
        {
            _requestHeaderToMatch = requestHeaderToMatch
               ?? throw new ArgumentNullException(nameof(requestHeaderToMatch));

            // check if the inputted media types are valid media types
            // and add them to the _mediaTypes collection                     

            if (MediaTypeHeaderValue.TryParse(mediaType,
                out MediaTypeHeaderValue parsedMediaType))
            {
                _mediaTypes.Add(parsedMediaType);
            }
            else
            {
                throw new ArgumentException(nameof(mediaType));
            }

            foreach (var otherMediaType in otherMediaTypes)
            {
                if (MediaTypeHeaderValue.TryParse(otherMediaType,
                   out MediaTypeHeaderValue parsedOtherMediaType))
                {
                    _mediaTypes.Add(parsedOtherMediaType);
                }
                else
                {
                    throw new ArgumentException(nameof(otherMediaTypes));
                }
            }

        }

        public int Order => 0;

        public bool Accept(ActionConstraintContext context)
        {
            var requestHeaders = context.RouteContext.HttpContext.Request.Headers;
            if (!requestHeaders.ContainsKey(_requestHeaderToMatch))
            {
                return false;
            }

            var parsedRequestMediaType = new MediaType(requestHeaders[_requestHeaderToMatch]);

            // if one of the media types matches, return true
            foreach (var mediaType in _mediaTypes)
            {
                var parsedMediaType = new MediaType(mediaType);
                if (parsedRequestMediaType.Equals(parsedMediaType))
                {
                    return true;
                }
            }
            return false;
        }
    }

        [HttpPost(Name = "CreateAuthor")]
        [RequestHeaderMatchesMediaType("Content-Type",
            "application/json",
            "application/vnd.marvin.authorforcreation+json")]
        [Consumes("application/json",
            "application/vnd.marvin.authorforcreation+json")]

```

## Versioning in a RESTful World
- URI
- Query strong
- Custom Header

=> Version Media Types 

-application/vnd.marvin.author.friendly.v1+json

                    