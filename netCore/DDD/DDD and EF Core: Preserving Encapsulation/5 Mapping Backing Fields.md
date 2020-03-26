# 5 Mapping Backing Fields

## Introducing On-To-many Relationships without encapsulation

**To Wide API !!!**

```C#

    public class Student : Entity
    {
        public string Name { get; }
        public string Email { get; }
        public virtual Course FavoriteCourse { get; }
        public virtual ICollection<Enrollment> Enrollments { get; set; }

        protected Student()
        {
        }

        public Student(string name, string email, Course favoriteCourse)
            : this()
        {
            Name = name;
            Email = email;
            FavoriteCourse = favoriteCourse;
        }
    }



  modelBuilder.Entity<Student>(x =>
            {
                x.ToTable("Student").HasKey(k => k.Id);
                x.Property(p => p.Id).HasColumnName("StudentID");
                x.Property(p => p.Email);
                x.Property(p => p.Name);
                x.HasOne(p => p.FavoriteCourse).WithMany();
                x.HasMany(p => p.Enrollments).WithOne(p => p.Student);
            });

```

## Hiding the Collection Behind a Backing Field

Ef Core Maps automatic to Baking Fields ! Code Example is the explicit way
don't expose internal entities as DbSets !

```C#
    public class Student : Entity
    {
        public string Name { get; }
        public string Email { get; }
        public virtual Course FavoriteCourse { get; }

        private readonly List<Enrollment> _enrollments = new List<Enrollment>();

        //Expose the most specific, consume the at least specific
        public virtual IReadOnlyList<Enrollment> Enrollments => _enrollments.ToList();

        protected Student()
        {
        }

        public Student(string name, string email, Course favoriteCourse)
            : this()
        {
            Name = name;
            Email = email;
            FavoriteCourse = favoriteCourse;
        }

        public void EnrollIn(Course course, Grade grade)
        {
            var enrollment = new Enrollment(course, this, grade);
            _enrollments.Add(enrollment);
        }
    }

           modelBuilder.Entity<Student>(x =>
            {
                x.ToTable("Student").HasKey(k => k.Id);
                x.Property(p => p.Id).HasColumnName("StudentID");
                x.Property(p => p.Email);
                x.Property(p => p.Name);
                x.HasOne(p => p.FavoriteCourse).WithMany();
                //Explicit way
                x.HasMany(p => p.Enrollments).WithOne(p => p.Student)
                    .Metadata.PrincipalToDependent.SetPropertyAccessMode(PropertyAccessMode.Field);
            });

```
## Introducing a Collection Invariant

Lazy Loading Problem, when accessing the backing Field!

Prefer find with explicit loading because of IDentity Map Pattern


```C#
public Student GetById(long studentId)
{
    Student student = _context.Students.Find(studentId);

    if (student == null)
        return null;

    _context.Entry(student).Collection(x => x.Enrollments).Load();

    return student;
}
```

## Deleting an item from the Collection


```C#
public void Disenroll(Course course)
{
    Enrollment enrollment = _enrollments.FirstOrDefault(x => x.Course == course);

    if (enrollment == null)
        return;

    _enrollments.Remove(enrollment);
}
```

## Shortcomings of Mapping to Backing Fields in EF Core

Navigation and Backing have to be the Same Type

```C#
private readonly List<Enrollment> _enrollments = new List<Enrollment>();
public virtual IReadOnlyList<Enrollment> Enrollments => _enrollments.ToList();
```
