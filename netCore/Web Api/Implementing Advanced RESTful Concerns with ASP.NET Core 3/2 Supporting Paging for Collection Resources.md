# 2 Supporting Paging for Collection Resources

## Paging through Collection Resources

```C#
public class AuthorsResourceParameters
    {
        const int maxPageSize = 20;
        public string MainCategory { get; set; }
        public string SearchQuery { get; set; }
        public int PageNumber { get; set; } = 1;

        private int _pageSize = 10;
        public int PageSize
        {
            get => _pageSize;
            set => _pageSize = (value > maxPageSize) ? maxPageSize : value;
        }
    }

[HttpGet(Name = "GetAuthors")]
[HttpHead]
public ActionResult<IEnumerable<AuthorDto>> GetAuthors(
    [FromQuery] AuthorsResourceParameters authorsResourceParameters)
{
    var authorsFromRepo = _courseLibraryRepository.GetAuthors(authorsResourceParameters);

    var previousPageLink = authorsFromRepo.HasPrevious ? 
        CreateAuthorsResourceUri(authorsResourceParameters,
        ResourceUriType.PreviousPage) : null;

    var nextPageLink = authorsFromRepo.HasNext ? 
        CreateAuthorsResourceUri(authorsResourceParameters,
        ResourceUriType.NextPage) : null;

    var paginationMetadata = new
    {
        totalCount = authorsFromRepo.TotalCount,
        pageSize = authorsFromRepo.PageSize,
        currentPage = authorsFromRepo.CurrentPage,
        totalPages = authorsFromRepo.TotalPages,
        previousPageLink,
        nextPageLink
    };

    Response.Headers.Add("X-Pagination",
        JsonSerializer.Serialize(paginationMetadata));

    return Ok(_mapper.Map<IEnumerable<AuthorDto>>(authorsFromRepo));
}

private string CreateAuthorsResourceUri(
           AuthorsResourceParameters authorsResourceParameters,
           ResourceUriType type)
        {
            switch (type)
            {
                case ResourceUriType.PreviousPage:
                    return Url.Link("GetAuthors",
                      new
                      {
                          pageNumber = authorsResourceParameters.PageNumber - 1,
                          pageSize = authorsResourceParameters.PageSize,
                          mainCategory = authorsResourceParameters.MainCategory,
                          searchQuery = authorsResourceParameters.SearchQuery
                      });
                case ResourceUriType.NextPage:
                    return Url.Link("GetAuthors",
                      new
                      {
                          pageNumber = authorsResourceParameters.PageNumber + 1,
                          pageSize = authorsResourceParameters.PageSize,
                          mainCategory = authorsResourceParameters.MainCategory,
                          searchQuery = authorsResourceParameters.SearchQuery
                      });

                default:
                    return Url.Link("GetAuthors",
                    new
                    {
                        pageNumber = authorsResourceParameters.PageNumber,
                        pageSize = authorsResourceParameters.PageSize,
                        mainCategory = authorsResourceParameters.MainCategory,
                        searchQuery = authorsResourceParameters.SearchQuery
                    });
            }

        }

public PagedList<Author> GetAuthors(AuthorsResourceParameters authorsResourceParameters)
{
    if (authorsResourceParameters == null)
    {
        throw new ArgumentNullException(nameof(authorsResourceParameters));
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

    return PagedList<Author>.Create(collection,
        authorsResourceParameters.PageNumber,
        authorsResourceParameters.PageSize); 
}

public class PagedList<T>: List<T>
{
    public int CurrentPage { get; private set; }
    public int TotalPages { get; private set; }
    public int PageSize { get; private set; }
    public int TotalCount { get; private set; }
    public bool HasPrevious => (CurrentPage > 1);
    public bool HasNext => (CurrentPage < TotalPages);

    public PagedList(List<T> items, int count, int pageNumber, int pageSize)
    {
        TotalCount = count;
        PageSize = pageSize;
        CurrentPage = pageNumber;
        TotalPages = (int)Math.Ceiling(count / (double)pageSize);
        AddRange(items);
    }

    public static PagedList<T> Create(IQueryable<T> source, int pageNumber, int pageSize)
    {
        var count = source.Count();
        var items = source.Skip((pageNumber - 1) * pageSize).Take(pageSize).ToList();
        return new PagedList<T>(items, count, pageNumber, pageSize);
    }
}
```

## Returning Pagination Metadata

Possibilities:
*  In response body => not application/json
*  Header => better

X-Pagination Header
