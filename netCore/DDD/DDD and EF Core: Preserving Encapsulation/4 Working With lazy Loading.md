#  Working With lazy Loading

## Eager Loading

**Anti Pattern: Partially Load Entities**

Load All Entities Relationships

## Lazy Loading

*   More Performance in some scenarios
*   Simplicity
*   N+1 problems

Best Lazy Loading for Write and Eager Loading for Reads

Write => EF with Lazy Loading
Read => SQL With Dapper

Don't use ILazyLoader !

## Identity Map PAttern

First EF looks for Entity in Cache, only if not existing it makes a DB Call  
**Always prefer DbContext.Find() over Single() or First()!**

* Performance
* Referential Equality

## Identity Map Referential Equality

If Find is used twice, the returned objects are referential equal

## Encapsulation Equality Comparer

*   Per Id
*   Referential


## Introducing Base Entity Class

```C#
    public abstract class Entity
    {
        public long Id { get; }

        protected Entity()
        {
        }

        protected Entity(long id)
            : this()
        {
            Id = id;
        }

        public override bool Equals(object obj)
        {
            if (!(obj is Entity other))
                return false;

            if (ReferenceEquals(this, other))
                return true;

            if (GetRealType() != other.GetRealType())
                return false;

            if (Id == 0 || other.Id == 0)
                return false;

            return Id == other.Id;
        }

        public static bool operator ==(Entity a, Entity b)
        {
            if (a is null && b is null)
                return true;

            if (a is null || b is null)
                return false;

            return a.Equals(b);
        }

        public static bool operator !=(Entity a, Entity b)
        {
            return !(a == b);
        }

        public override int GetHashCode()
        {
            return (GetRealType().ToString() + Id).GetHashCode();
        }


        // Violates theoretically the SoC
        private Type GetRealType()
        {
            Type type = GetType();

            if (type.ToString().Contains("Castle.Proxies."))
                return type.BaseType;

            return type;
        }
    }
}
```



