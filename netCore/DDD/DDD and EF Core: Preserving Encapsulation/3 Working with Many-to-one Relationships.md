# Working with Many-to-one Relationships
## DbContext Encapsulation

**Expose an minimum of configuration options !**  
Possible Add Scoped Method for own constructor

```C#
 public sealed class SchoolContext : DbContext
    {
        private readonly string _connectionString;
        private readonly bool _useConsoleLogger;

        public DbSet<Student> Students { get; set; }
        public DbSet<Course> Courses { get; set; }

        public SchoolContext(string connectionString, bool useConsoleLogger)
        {
            _connectionString = connectionString;
            _useConsoleLogger = useConsoleLogger;
        }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            ILoggerFactory loggerFactory = LoggerFactory.Create(builder =>
            {
                builder
                    .AddFilter((category, level) =>
                        category == DbLoggerCategory.Database.Command.Name && level == LogLevel.Information)
                    .AddConsole();
            });

            optionsBuilder
                .UseSqlServer(_connectionString);

            if (_useConsoleLogger)
            {
                optionsBuilder
                    .UseLoggerFactory(loggerFactory)
                    .EnableSensitiveDataLogging();
            }
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Student>(x =>
            {
                x.ToTable("Student").HasKey(k => k.Id);
                x.Property(p => p.Id).HasColumnName("StudentID");
                x.Property(p => p.Email);
                x.Property(p => p.Name);
                x.HasOne(p => p.FavoriteCourse).WithMany();
            });
            modelBuilder.Entity<Course>(x =>
            {
                x.ToTable("Course").HasKey(k => k.Id);
                x.Property(p => p.Id).HasColumnName("CourseID");
                x.Property(p => p.Name);
            });
        }
    }
```

## Getting Rid of Public Setters

Setters should be private, even private setters are not needed for ef core!    Constructor should be contain mandatory fields, field names Should Be same than the properties

```C#
    public class Student
    {
        public long Id { get; }
        public string Name { get; }
        public string Email { get; }
        public Course FavoriteCourse { get;  }

        private Student()
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
```
## Many To One

Id or Navigation Property  
Prefer Navigation Properties !  
Persistence ignorance !

Surrogate vs Natural Ids 

Ids can Used Outside of the domain model.  
But IDs should not used inside the domain model

## Refactoring to navigation Properties

private constructor needed for Entity Framework  
