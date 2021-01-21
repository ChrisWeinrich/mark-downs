# 8 Updating Resources

## Put vs Patch

Put => Full update
Patch => Partial

## Updating a resource

```C#
[HttpPut("{courseId}")]
public IActionResult UpdateCourseForAuthor(Guid authorId, 
    Guid courseId, 
    CourseForUpdateDto course)
{
    if (!_courseLibraryRepository.AuthorExists(authorId))
    {
        return NotFound();
    }

    var courseForAuthorFromRepo = _courseLibraryRepository.GetCourse(authorId, courseId);

    if (courseForAuthorFromRepo == null)
    {
        var courseToAdd = _mapper.Map<Entities.Course>(course);
        courseToAdd.Id = courseId;

        _courseLibraryRepository.AddCourse(authorId, courseToAdd);

        _courseLibraryRepository.Save();

        var courseToReturn = _mapper.Map<CourseDto>(courseToAdd);

        return CreatedAtRoute("GetCourseForAuthor",
            new { authorId, courseId = courseToReturn.Id },
            courseToReturn);
    }

    // map the entity to a CourseForUpdateDto
    // apply the updated field values to that dto
    // map the CourseForUpdateDto back to an entity
    _mapper.Map(course, courseForAuthorFromRepo);

    _courseLibraryRepository.UpdateCourse(courseForAuthorFromRepo);

    _courseLibraryRepository.Save();
    return NoContent();
}
```

## Updating Collection Resources

PUT => .../api/authors/{authorid}/courses

overwrite whole Collection => could be very destructive !

## Upserting

Create or Update !

**only senseful when consumer creates the uri respectively the id**


## Partially Updating a Resource

```C#
[HttpPatch("{courseId}")]
public ActionResult PartiallyUpdateCourseForAuthor(Guid authorId, 
    Guid courseId,
    JsonPatchDocument<CourseForUpdateDto> patchDocument)
{
    if (!_courseLibraryRepository.AuthorExists(authorId))
    {
        return NotFound();
    }

    var courseForAuthorFromRepo = _courseLibraryRepository.GetCourse(authorId, courseId);

    if (courseForAuthorFromRepo == null)
    {
        var courseDto = new CourseForUpdateDto();
        patchDocument.ApplyTo(courseDto, ModelState);

        if (!TryValidateModel(courseDto))
        {
            return ValidationProblem(ModelState);
        }

        var courseToAdd = _mapper.Map<Entities.Course>(courseDto);
        courseToAdd.Id = courseId;

        _courseLibraryRepository.AddCourse(authorId, courseToAdd);
        _courseLibraryRepository.Save();

        var courseToReturn = _mapper.Map<CourseDto>(courseToAdd);

        return CreatedAtRoute("GetCourseForAuthor",
            new { authorId, courseId = courseToReturn.Id }, 
            courseToReturn);
    }

    var courseToPatch = _mapper.Map<CourseForUpdateDto>(courseForAuthorFromRepo);
    // add validation
    patchDocument.ApplyTo(courseToPatch, ModelState);

    if (!TryValidateModel(courseToPatch))
    {
        return ValidationProblem(ModelState);
    }

    _mapper.Map(courseToPatch, courseForAuthorFromRepo);

    _courseLibraryRepository.UpdateCourse(courseForAuthorFromRepo);

    _courseLibraryRepository.Save();

    return NoContent();
        }

```

