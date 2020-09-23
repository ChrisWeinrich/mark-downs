# Domain Events

## Introducing new bounded Contexts

As little Coupling as Possible between Bounded Contexts

## Domain Events

Event + Domain significance

*   Decouple Bounded Contexts
*   Communication between BCs
*   Decouple entities inside BC

### GuidLines
*   Naming => Past Tense i.e. BalanceChangedEvent
*   Include as little Data as possible
*   Don't use Domain Classes in Events
*   Use Primitives in Events

### Physical Delivery

*   Inner-Memory Structures
*   Service Bus

### Classic Approach
*   Domain Events Class which Holds all Handlers
*   When a Event is Raised It Scans Collection of handlers and Execute correct ones

```C#
```

### Modern Approach
*   Seperate creating an Event and Dispatching Event
*   Domain Entities Creates Event
*   Move Dispatching to unit Of Work Save Changes
*   Dispatching should be done after persisted Changes ! (In this Course NHibernate is used so this has to be adjusted to EF Core)

```C#
    public abstract class AggregateRoot : Entity
    {
        private readonly List<IDomainEvent> _domainEvents = new List<IDomainEvent>();
        public virtual IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents;

        protected virtual void AddDomainEvent(IDomainEvent newEvent)
        {
            _domainEvents.Add(newEvent);
        }

        public virtual void ClearEvents()
        {
            _domainEvents.Clear();
        }
    }
    
        public static class DomainEvents
    {
        private static List<Type> _handlers;

        public static void Init()
        {
            _handlers = Assembly.GetExecutingAssembly()
                .GetTypes()
                .Where(x => x.GetInterfaces().Any(y => y.IsGenericType && y.GetGenericTypeDefinition() == typeof(IHandler<>)))
                .ToList();
        }

        public static void Dispatch(IDomainEvent domainEvent)
        {
            foreach (Type handlerType in _handlers)
            {
                bool canHandleEvent = handlerType.GetInterfaces()
                    .Any(x => x.IsGenericType
                        && x.GetGenericTypeDefinition() == typeof(IHandler<>)
                        && x.GenericTypeArguments[0] == domainEvent.GetType());

                if (canHandleEvent)
                {
                    dynamic handler = Activator.CreateInstance(handlerType);
                    handler.Handle((dynamic)domainEvent);
                }
            }
        }               
    }
    
    public interface IHandler<T>
        where T : IDomainEvent
    {
        void Handle(T domainEvent);
    }
    
    public interface IDomainEvent
    {
    }
    
    public class BalanceChangedEventHandler : IHandler<BalanceChangedEvent>
    {
        public void Handle(BalanceChangedEvent domainEvent)
        {
            var repository = new HeadOfficeRepository();
            HeadOffice headOffice = HeadOfficeInstance.Instance;
            headOffice.ChangeBalance(domainEvent.Delta);
            repository.Save(headOffice);
        }
    }
    
    public class BalanceChangedEvent : IDomainEvent
    {
        public decimal Delta { get; private set; }

        public BalanceChangedEvent(decimal delta)
        {
            Delta = delta;
        }
    }         
```

### Between Microservices
=> EsbGateway

### ORM
=> Read Database directly into DTOs !!!

