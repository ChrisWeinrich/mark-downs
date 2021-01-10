# 6 Creating Resources

## Method Safety and Method Idempotency

 Safe => doesn't change the source representation  
 idempotent => same result for multiple calls

## Creating a Resource

```C#

public class AuthorForCreationDto
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTimeOffset DateOfBirth { get; set; }
    public string MainCategory { get; set; }
    public ICollection<CourseForCreationDto> Courses { get; set; }
        = new List<CourseForCreationDto>();
}

[HttpPost]
public ActionResult<AuthorDto> CreateAuthor(AuthorForCreationDto author)
{
    var authorEntity = _mapper.Map<Entities.Author>(author);
    _courseLibraryRepository.AddAuthor(authorEntity);
    _courseLibraryRepository.Save();

    var authorToReturn = _mapper.Map<AuthorDto>(authorEntity);
    return CreatedAtRoute("GetAuthor",
        new { authorId = authorToReturn.Id },
        authorToReturn);
}
```

## Creating child Resources

```C#

public class CourseForCreationDto
{
    public string Title { get; set; }
    public string Description { get; set; }
}

[HttpGet("{courseId}", Name = "GetCourseForAuthor")]
public ActionResult<CourseDto> GetCourseForAuthor(Guid authorId, Guid courseId)
{
...
}

[HttpPost]
public ActionResult<CourseDto> CreateCourseForAuthor(
    Guid authorId, CourseForCreationDto course)
{
    if (!_courseLibraryRepository.AuthorExists(authorId))
    {
        return NotFound();
    }

    var courseEntity = _mapper.Map<Entities.Course>(course);
    _courseLibraryRepository.AddCourse(authorId, courseEntity);
    _courseLibraryRepository.Save();

    var courseToReturn = _mapper.Map<CourseDto>(courseEntity);
    return CreatedAtRoute("GetCourseForAuthor",
        new { authorId = authorId, courseId = courseToReturn.Id }, 
        courseToReturn);
}
```

## Creating Child resource together with parent

```C#
public class AuthorForCreationDto
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTimeOffset DateOfBirth { get; set; }
    public string MainCategory { get; set; }
    public ICollection<CourseForCreationDto> Courses { get; set; }
        = new List<CourseForCreationDto>();
}
```

## Creating a Collection of Authors / Working with Array Keys

```C#
   [ApiController]
    [Route("api/authorcollections")]
    public class AuthorCollectionsController : ControllerBase
    {
        private readonly ICourseLibraryRepository _courseLibraryRepository;
        private readonly IMapper _mapper;

        public AuthorCollectionsController(ICourseLibraryRepository courseLibraryRepository,
            IMapper mapper)
        {
            _courseLibraryRepository = courseLibraryRepository ??
                throw new ArgumentNullException(nameof(courseLibraryRepository));
            _mapper = mapper ??
                throw new ArgumentNullException(nameof(mapper));
        }

        [HttpGet("({ids})", Name ="GetAuthorCollection")]
        public IActionResult GetAuthorCollection(
        [FromRoute]
        [ModelBinder(BinderType = typeof(ArrayModelBinder))] IEnumerable<Guid> ids)
        {
            if (ids == null)
            {
                return BadRequest();
            }

            var authorEntities = _courseLibraryRepository.GetAuthors(ids);

            if (ids.Count() != authorEntities.Count())
            {
                return NotFound();
            }

            var authorsToReturn = _mapper.Map<IEnumerable<AuthorDto>>(authorEntities);

            return Ok(authorsToReturn);
        }


        [HttpPost]
        public ActionResult<IEnumerable<AuthorDto>> CreateAuthorCollection(
            IEnumerable<AuthorForCreationDto> authorCollection)
        {
            var authorEntities = _mapper.Map<IEnumerable<Entities.Author>>(authorCollection);
            foreach (var author in authorEntities)
            {
                _courseLibraryRepository.AddAuthor(author);
            }

            _courseLibraryRepository.Save();

            var authorCollectionToReturn = _mapper.Map<IEnumerable<AuthorDto>>(authorEntities);
            var idsAsString = string.Join(",", authorCollectionToReturn.Select(a => a.Id));
            return CreatedAtRoute("GetAuthorCollection",
             new { ids = idsAsString },
             authorCollectionToReturn);
        }
    }

    public class ArrayModelBinder : IModelBinder
    {
        public Task BindModelAsync(ModelBindingContext bindingContext)
        {
            // Our binder works only on enumerable types
            if (!bindingContext.ModelMetadata.IsEnumerableType)
            {
                bindingContext.Result = ModelBindingResult.Failed();
                return Task.CompletedTask;
            }

            // Get the inputted value through the value provider
            var value = bindingContext.ValueProvider
                .GetValue(bindingContext.ModelName).ToString();

            // If that value is null or whitespace, we return null
            if (string.IsNullOrWhiteSpace(value))
            {
                bindingContext.Result = ModelBindingResult.Success(null);
                return Task.CompletedTask;
            }

            // The value isn't null or whitespace, 
            // and the type of the model is enumerable. 
            // Get the enumerable's type, and a converter 
            var elementType = bindingContext.ModelType.GetTypeInfo().GenericTypeArguments[0];
            var converter = TypeDescriptor.GetConverter(elementType);

            // Convert each item in the value list to the enumerable type
            var values = value.Split(new[] { "," }, StringSplitOptions.RemoveEmptyEntries)
                .Select(x => converter.ConvertFromString(x.Trim()))
                .ToArray();

            // Create an array of that type, and set it as the Model value 
            var typedValues = Array.CreateInstance(elementType, values.Length);
            values.CopyTo(typedValues, 0);
            bindingContext.Model = typedValues;

            // return a successful result, passing in the Model 
            bindingContext.Result = ModelBindingResult.Success(bindingContext.Model);
            return Task.CompletedTask;
        }
    }
```

## Handling Post to a single resource

=> 405

## Supporting Options

```C#
[HttpOptions]
public IActionResult GetAuthorsOptions()
{
    Response.Headers.Add("Allow", "GET,OPTIONS,POST");
    return Ok();
}
```

##  Add AdditionalContent Type and Input formater

``` C#
services.AddControllers(setupAction =>
            {
                setupAction.ReturnHttpNotAcceptable = true;

            }).AddXmlDataContractSerializerFormatters();
```

