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