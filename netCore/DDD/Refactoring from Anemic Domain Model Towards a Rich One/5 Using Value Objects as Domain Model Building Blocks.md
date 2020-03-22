# Using Value Objects as Domain Model Building Blocks

## IMportance of USing Value Objects

*   No Identity
*   Immutable
*   Interchangeable
*   Bound lifetime

i.e. money : Amount + Currency

Single Values makes sometimes also sense.
*   Fail Fast
*   More expressive domain Model
*   Avoid domain logic duplication

## Identifying  Value Objects

Problem: Violation of invariants, i.e. negative numbers for positive only
Invalid Email ... !

i.e. Name, Email, Money Amount 

Value Object base class
Equals, == overwrites

static factory methods for Creation and validation

```C#
 public class Email : ValueObject<Email>
    {
        public string Value { get; }

        private Email(string value)
        {
            Value = value;
        }

        public static Result<Email> Create(string email)
        {
            email = (email ?? string.Empty).Trim();

            if (email.Length == 0)
                return Result.Fail<Email>("Email should not be empty");

            if (!Regex.IsMatch(email, @"^(.+)@(.+)$"))
                return Result.Fail<Email>("Email is invalid");

            return Result.Ok(new Email(email));
        }

        protected override bool EqualsCore(Email other)
        {
            return Value.Equals(other.Value, StringComparison.InvariantCultureIgnoreCase);
        }

        protected override int GetHashCodeCore()
        {
            return Value.GetHashCode();
        }

        public static explicit operator Email(string email)
        {
            return Create(email).Value;
        }

        public static implicit operator string(Email email)
        {
            return email.Value;
        }
    }
```

## Gathering Validation Rules

Private Constructor + Static Factory with Result Class (See above)!

CSharpFunctionalExtensions

## Introduce Money Value Object

```C#
public class Dollars : ValueObject<Dollars>
    {
        private const decimal MaxDollarAmount = 1_000_000;

        public decimal Value { get; }

        public bool IsZero => Value == 0;

        private Dollars(decimal value)
        {
            Value = value;
        }

        public static Result<Dollars> Create(decimal dollarAmount)
        {
            if (dollarAmount < 0)
                return Result.Fail<Dollars>("Dollar amount cannot be negative");

            if (dollarAmount > MaxDollarAmount)
                return Result.Fail<Dollars>("Dollar amount cannot be greater than " + MaxDollarAmount);

            if (dollarAmount % 0.01m > 0)
                return Result.Fail<Dollars>("Dollar amount cannot contain part of a penny");

            return Result.Ok(new Dollars(dollarAmount));
        }

        public static Dollars Of(decimal dollarAmount)
        {
            return Create(dollarAmount).Value;
        }

        public static Dollars operator *(Dollars dollars, decimal multiplier)
        {
            return new Dollars(dollars.Value * multiplier);
        }

        public static Dollars operator +(Dollars dollars1, Dollars dollars2)
        {
            return new Dollars(dollars1.Value + dollars2.Value);
        }

        protected override bool EqualsCore(Dollars other)
        {
            return Value == other.Value;
        }

        protected override int GetHashCodeCore()
        {
            return Value.GetHashCode();
        }

        public static implicit operator decimal(Dollars dollars)
        {
            return dollars.Value;
        }
    }
```

### Introduce Expiration Date

```C#
    public class ExpirationDate : ValueObject<ExpirationDate>
    {
        public static readonly ExpirationDate Infinite = new ExpirationDate(null);

        public DateTime? Date { get; }

        public bool IsExpired => this != Infinite && Date < DateTime.UtcNow;

        private ExpirationDate(DateTime? date)
        {
            Date = date;
        }

        public static Result<ExpirationDate> Create(DateTime date)
        {
            return Result.Ok(new ExpirationDate(date));
        }

        protected override bool EqualsCore(ExpirationDate other)
        {
            return Date == other.Date;
        }

        protected override int GetHashCodeCore()
        {
            return Date.GetHashCode();
        }

        public static explicit operator ExpirationDate(DateTime? date)
        {
            if (date.HasValue)
                return Create(date.Value).Value;

            return Infinite;
        }

        public static implicit operator DateTime? (ExpirationDate date)
        {
            return date.Date;
        }
    }
```
