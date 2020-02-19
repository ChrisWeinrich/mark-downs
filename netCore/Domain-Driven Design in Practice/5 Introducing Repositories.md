# 5.Introducing Repositories

## Repositories 

Repositories for Aggregates Only btw. One Repository per aggregate
 
## Repository Base Class 

```csharp
    public abstract class Repository<T>
        where T : AggregateRoot
    {
        public T GetById(long id)
        {
            using (ISession session = SessionFactory.OpenSession())
            {
                return session.Get<T>(id);
            }
        }

        public void Save(T aggregateRoot)
        {
            using (ISession session = SessionFactory.OpenSession())
            using (ITransaction transaction = session.BeginTransaction())
            {
                session.SaveOrUpdate(aggregateRoot);
                transaction.Commit();
            }
        }
    }
```

Work with IReadOnlyList !  
Naming should be independend from implementation  

## Setting up Mappings for the Aggregates

**Think about attached and non attached**

## Snack Entitiy

**Reference Data ! (Base Data)**

private factories

*none uses the Null Object Pattern*

```csharp

    public class Snack : AggregateRoot
    {
        public static readonly Snack None = new Snack(0, "None");
        public static readonly Snack Chocolate = new Snack(1, "Chocolate");
        public static readonly Snack Soda = new Snack(2, "Soda");
        public static readonly Snack Gum = new Snack(3, "Gum");

        public virtual string Name { get; }

        protected Snack()
        {
        }

        private Snack(long id, string name)
            : this()
        {
            Id = id;
            Name = name;
        }
    }
```

## Adjusting User Interface









```csharp


```


