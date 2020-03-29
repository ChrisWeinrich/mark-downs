# 7 Mapping Value Objects

## Introducing a Value Object: Email

Value object: No Identity,Immutable
Value can be validated through Value Object

```C#
    public class Email : ValueObject
    {
        public string Value { get; }

        private Email(string value)
        {
            Value = value;
        }

        public static Result<Email> Create(string email)
        {
            if (string.IsNullOrWhiteSpace(email))
                return Result.Failure<Email>("Email should not be empty");

            email = email.Trim();

            if (email.Length > 200)
                return Result.Failure<Email>("Email is too long");

            if (!Regex.IsMatch(email, @"^(.+)@(.+)$"))
                return Result.Failure<Email>("Email is invalid");

            return Result.Success(new Email(email));
        }

        protected override IEnumerable<object> GetEqualityComponents()
        {
            yield return Value;
        }

        public static implicit operator string(Email email)
        {
            return email.Value;
        }
    }
```

Use Value Conversion

```C#
x.Property(p => p.Email)
        .HasConversion(p => p.Value, p => Email.Create(p).Value);
```

## Shortcomings of EF Core Value Conversions

Ef Core Doesn't pass nulls to static Factory Method  
Nullable DataColumn and non Nullable Type are not working !


## Introducing a Multi-property Value Object

Use Owed Types

```C#

  x.OwnsOne(p => p.Name, p =>
    {
        p.Property(pp => pp.First).HasColumnName("FirstName");
        p.Property(pp => pp.Last).HasColumnName("LastName");
    });

    public class Name : ValueObject
    {
        public string First { get; }
        public string Last { get; }

        protected Name()
        {
        }

        private Name(string first, string last)
            : this()
        {
            First = first;
            Last = last;
        }

        public static Result<Name> Create(string firstName, string lastName)
        {
            if (string.IsNullOrWhiteSpace(firstName))
                return Result.Failure<Name>("First name should not be empty");
            if (string.IsNullOrWhiteSpace(lastName))
                return Result.Failure<Name>("Last name should not be empty");

            firstName = firstName.Trim();
            lastName = lastName.Trim();

            if (firstName.Length > 200)
                return Result.Failure<Name>("First name is too long");
            if (lastName.Length > 200)
                return Result.Failure<Name>("Last name is too long");

            return Result.Success(new Name(firstName, lastName));
        }

        protected override IEnumerable<object> GetEqualityComponents()
        {
            yield return First;
            yield return Last;
        }
    }
```

## Owned Entity Types Behind the Scenes

Owned Entity => regular entities With hidden Id 

Same Table And Separate Tables are possible

Owned Entities have to be virtual

Nullable now possible in ef core 3.1 !

## Adding a Navigation Property to an Owned Entity

Shadow Property Id has to Mapped to a Column

```C#
 x.OwnsOne(p => p.Name, p =>
    {
        p.Property<long?>("NameSuffixID").HasColumnName("NameSuffixID");
        p.Property(pp => pp.First).HasColumnName("FirstName");
        p.Property(pp => pp.Last).HasColumnName("LastName");
        p.HasOne(pp => pp.Suffix).WithMany().HasForeignKey("NameSuffixID").IsRequired(false);
    });
```