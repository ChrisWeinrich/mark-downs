# 8 Putting It All Together

## Refactoring Away from Exceptions

General Error Handler for 500
Caught know exceptions from 3rd Party APIs as lowest as possible

## Refactoring Away from Primitive Obsession

```C#
public class Email : ValueObject<Email>
{
    public string Value { get; }

    private Email(string value)
    {
        Value = value;
    }

    public static Result<Email> Create(Maybe<string> emailOrNothing)
    {
        return emailOrNothing.ToResult("Email should not be empty")
            .OnSuccess(email => email.Trim())
            .Ensure(email => email != string.Empty, "Email should not be empty")
            .Ensure(email => email.Length <= 256, "Email is too long")
            .Ensure(email => Regex.IsMatch(email, @"^(.+)@(.+)$"), "Email is invalid")
            .Map(email => new Email(email));
    }
    
    protected override bool EqualsCore(Email other)
    {
        return Value == other.Value;
    }

    protected override int GetHashCodeCore()
    {
        return Value.GetHashCode();
    }

    public static explicit operator Email(string email)
    {
        return Create(email).Value;
    }

    public static implicit operator string (Email email)
    {
        return email.Value;
    }
}
```

## Refactoring to More Explicit Code

=> Value Object

## Making Null Explicit

*   Nullguard
*   Use of MAybe
*   Incoming Null get converted to Maybe
*   Converted Back when they leave the boundary

```C#
    private string _secondaryEmail;
    public virtual Maybe<Email> SecondaryEmail
    {
        get { return _secondaryEmail == null ? null : (Email)_secondaryEmail; }
        protected set { _secondaryEmail = value.Unwrap(x => x.Value); }
    }
```

## Representing Reference Data as Code

```C#
    public class Industry : Entity
    {
        public static readonly Industry Cars = new Industry(1, "Cars");
        public static readonly Industry Pharmacy = new Industry(2, "Pharmacy");
        public static readonly Industry Other = new Industry(3, "Other");
        
        public virtual string Name { get; protected set; }

        protected Industry()
        {
        }

        private Industry(int id, string name)
        {
            Id = id;
            Name = name;
        }

        public static Result<Industry> Get(Maybe<string> name)
        {
            if (name.HasNoValue)
                return Result.Fail<Industry>("Industry name is not specified");

            if (name.Value == Cars.Name)
                return Result.Ok(Cars);

            if (name.Value == Pharmacy.Name)
                return Result.Ok(Pharmacy);

            if (name.Value == Other.Name)
                return Result.Ok(Other);

            return Result.Fail<Industry>("Industry name is invalid: " + name);
        }
    }
```

## Railway-oriented Programming

```C#
   public static Result<Email> Create(Maybe<string> emailOrNothing)
        {
            return emailOrNothing.ToResult("Email should not be empty")
                .OnSuccess(email => email.Trim())
                .Ensure(email => email != string.Empty, "Email should not be empty")
                .Ensure(email => email.Length <= 256, "Email is too long")
                .Ensure(email => Regex.IsMatch(email, @"^(.+)@(.+)$"), "Email is invalid")
                .Map(email => new Email(email));
        }
```