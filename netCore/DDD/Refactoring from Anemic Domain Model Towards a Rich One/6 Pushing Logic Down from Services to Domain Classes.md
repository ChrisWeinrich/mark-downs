# Pushing Logic Down from Services to Domain Classes

## Refactoring Customer: Constructor

```C#
public Customer(CustomerName name, Email email) : this()
{
    _name = name ?? throw new ArgumentNullException(nameof(name));
    _email = email ?? throw new ArgumentNullException(nameof(email));

    MoneySpent = Dollars.Of(0);
    Status = CustomerStatus.Regular;
}

```
## Refactoring Customer: Collection
Protect/Hide Collections !!!

```C#
private readonly IList<PurchasedMovie> _purchasedMovies;
// To List() makes a copy of it and so it is immutable
public virtual IReadOnlyList<PurchasedMovie> PurchasedMovies => _purchasedMovies.ToList();

```

prefer IEnumerable when accepting a collection; prefer IReadOnlyList when returning one.

## Refactoring Customer: Status


```C#
public static readonly CustomerStatus Regular = new CustomerStatus(CustomerStatusType.Regular, ExpirationDate.Infinite);
```

## Pushing Down Logic from Services to Entities

Use Cases for domains Services:
*   Referring to external resources
*   Logic doesn't naturally fit any entity or value object

Services should stay thin



   
