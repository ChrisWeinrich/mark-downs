# 7 Using Immutable Objects When Possible

## Causing a Bug That Comes from a Mutable Object

Value Type in DDD

## Discovering the Aliasing Bug

object has more then one reference btw. is a **shared resource** => dirty read in database terms

=> don't modify shared object i.e. method arguments

## Understanding Value Objects

integers

## Implementing Reference Type as a Value Type

Use of Constructor, remove all setters !

```C#
    class MoneyAmount
    {
        public decimal Amount { get; }
        public string CurrencySymbol { get; }

        public MoneyAmount(decimal amount, string currencySymbol)
        {
            this.Amount = amount;
            this.CurrencySymbol = currencySymbol;
        }

        public MoneyAmount Scale(decimal factor) =>
            new MoneyAmount(this.Amount * factor, this.CurrencySymbol);

        public static MoneyAmount operator *(MoneyAmount amount, decimal factor) => amount.Scale(factor);

        public override string ToString() => $"{this.Amount} {this.CurrencySymbol}";
    }
```

## Turning Immutable Objects into Value Objects & Supporting Hash Tables

=> implementing Custom EQuality

**Sealed because of problems with equality and possible additional fields in derived classes**

GetHasCode is need i.e. for HashSets. Normal GetHasGod from Object takes the reference of an Object so it must overridden.

```C#
    sealed class MoneyAmount : IEquatable<MoneyAmount>
    {
        public decimal Amount { get; }
        public string CurrencySymbol { get; }

        public MoneyAmount(decimal amount, string currencySymbol)
        {
            this.Amount = amount;
            this.CurrencySymbol = currencySymbol;
        }

        public MoneyAmount Scale(decimal factor) =>
            new MoneyAmount(this.Amount * factor, this.CurrencySymbol);

        public static MoneyAmount operator *(MoneyAmount amount, decimal factor) => amount.Scale(factor);

        public override bool Equals(object obj) =>
            this.Equals(obj as MoneyAmount);

        public bool Equals(MoneyAmount other) =>
            other != null &&
            this.Amount == other.Amount &&
            this.CurrencySymbol == other.CurrencySymbol;

        public override int GetHashCode() =>
            this.Amount.GetHashCode() ^ this.CurrencySymbol.GetHashCode();

        public static bool operator ==(MoneyAmount a, MoneyAmount b) =>
            (object.ReferenceEquals(a, null) && object.ReferenceEquals(b, null)) ||
            (!object.ReferenceEquals(a, null) && a.Equals(b));

        public static bool operator !=(MoneyAmount a, MoneyAmount b) => !(a == b);

        public override string ToString() => $"{this.Amount} {this.CurrencySymbol}";
    }
```

## Mutable vs. Immutable vs. Value Object

