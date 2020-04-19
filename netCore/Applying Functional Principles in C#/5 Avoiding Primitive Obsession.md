# 5 Avoiding Primitive Obsession

## What Is Primitive Obsession?

Use primitive types for domain modelling

String => Email

*   Dishonest signature
*   Code Duplication/DRY (Validation should be in single place)

## How to get Rid of Primitive Obsession

=> Value Object

## Primitive Obsession and Defensive Programming

Checks encapsulated into the value object

## Primitive Obsession: Limitations

Don't apply for all domain concepts

## Where to Convert Primitive Types into Value Objects?

Validation before the domain boundary

## Refactoring Away from Primitive Obsession

```C#
public class Customer
{
    public CustomerName Name { get; private set; }
    public Email Email { get; private set; }

    public Customer(CustomerName name, Email email)
    {
        if (name == null)
            throw new ArgumentNullException(nameof(name));
        if (email == null)
            throw new ArgumentNullException(nameof(email));

        Name = name;
        Email = email;
    }

    public void ChangeName(CustomerName name)
    {
        if (name == null)
            throw new ArgumentNullException(nameof(name));

        Name = name;
    }

    public void ChangeEmail(Email email)
    {
        if (email == null)
            throw new ArgumentNullException(nameof(email));

        Email = email;
    }
}
```
 

