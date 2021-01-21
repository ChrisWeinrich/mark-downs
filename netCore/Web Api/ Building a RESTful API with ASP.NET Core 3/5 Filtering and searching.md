# 5 Filtering and searching

## Passing Data to the API

[FromRoute]   
[FromQuery]  
[FromBody]   
[FromForm]    

**[ApiController]** Better Behavior for APIs

## Filter and searching
filtering is precise  
search => textSearch

## Demo - filtering and searching Resource Collection

```C#

public class AuthorsResourceParameters
    {
        public string MainCategory { get; set; }
        public string SearchQuery { get; set; }
    }

[HttpGet()]
[HttpHead]
        public ActionResult<IEnumerable<AuthorDto>> GetAuthors(
            [FromQuery] AuthorsResourceParameters authorsResourceParameters)
        {
            var authorsFromRepo = _courseLibraryRepository.GetAuthors(authorsResourceParameters);
            return Ok(_mapper.Map<IEnumerable<AuthorDto>>(authorsFromRepo));
        }

public IEnumerable<Author> GetAuthors(AuthorsResourceParameters authorsResourceParameters)
        {
            if (authorsResourceParameters == null)
            {
                throw new ArgumentNullException(nameof(authorsResourceParameters));
            }

            if (string.IsNullOrWhiteSpace(authorsResourceParameters.MainCategory)
                 && string.IsNullOrWhiteSpace(authorsResourceParameters.SearchQuery))
            {
                return GetAuthors();
            }

            var collection = _context.Authors as IQueryable<Author>;

            if (!string.IsNullOrWhiteSpace(authorsResourceParameters.MainCategory))
            {
                var mainCategory = authorsResourceParameters.MainCategory.Trim();
                collection = collection.Where(a => a.MainCategory == mainCategory);
            }

            if (!string.IsNullOrWhiteSpace(authorsResourceParameters.SearchQuery))
            {

                var searchQuery = authorsResourceParameters.SearchQuery.Trim();
                collection = collection.Where(a => a.MainCategory.Contains(searchQuery)
                    || a.FirstName.Contains(searchQuery)
                    || a.LastName.Contains(searchQuery));
            }

            // Deferred Execution

            return collection.ToList();
        }

``



 

