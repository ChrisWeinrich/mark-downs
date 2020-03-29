# 8 Implementing a Domain Event Dispatcher

## Domain Events

=> Domain Meaning, i.e. Student Email Changed Event

*   Rasing an Event: Part of the domain Logic
*   Dispatching an event: Converting the event into a side effect

## Implementing Domain Events

```C#
    public interface IDomainEvent
    {
    }

    public sealed class StudentEmailChangedEvent : IDomainEvent
    {
        public long StudentId { get; }
        public Email NewEmail { get; }

        public StudentEmailChangedEvent(long studentId, Email newEmail)
        {
            StudentId = studentId;
            NewEmail = newEmail;
        }
    }
```
Events should only include primitives or value objects.

```C#
    public abstract class Entity
    {
        private readonly List<IDomainEvent> _domainEvents = new List<IDomainEvent>();
        public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents;

        public long Id { get; }

        protected Entity()
        {
        }

        protected Entity(long id)
            : this()
        {
            Id = id;
        }

        protected void RaiseDomainEvent(IDomainEvent domainEvent)
        {
            _domainEvents.Add(domainEvent);
        }

        public void ClearDomainEvents()
        {
            _domainEvents.Clear();
        }
    }
```

Dispatching should be done in EF Core SaveChanges

```C#
        public override int SaveChanges()
        {
            IEnumerable<EntityEntry> enumerationEntries = ChangeTracker.Entries()
                .Where(x => EnumerationTypes.Contains(x.Entity.GetType()));

            foreach (EntityEntry enumerationEntry in enumerationEntries)
            {
                enumerationEntry.State = EntityState.Unchanged;
            }

            List<Entity> entities = ChangeTracker
                .Entries()
                .Where(x => x.Entity is Entity)
                .Select(x => (Entity)x.Entity)
                .ToList();

            int result = base.SaveChanges();

            foreach (Entity entity in entities)
            {
                _eventDispatcher.Dispatch(entity.DomainEvents);
                entity.ClearDomainEvents();
            }

            return result;
        }
```

Message Bus

```C#
    public interface IBus
    {
        void Send(string message);
    }

    public sealed class Bus : IBus
    {
        public void Send(string message)
        {
            // Put the message on a bus instead
            Console.WriteLine($"Message sent: '{message}'");
        }
    }

    public sealed class MessageBus
    {
        private readonly IBus _bus;

        public MessageBus(IBus bus)
        {
            _bus = bus;
        }

        public void SendEmailChangedMessage(long studentId, string newEmail)
        {
            _bus.Send("Type: STUDENT_EMAIL_CHANGED; " +
                $"Id: {studentId}; " +
                $"NewEmail: {newEmail}");
        }
    }

```

Dispatcher

```C#
        private readonly MessageBus _messageBus;

        public EventDispatcher(MessageBus messageBus)
        {
            _messageBus = messageBus;
        }

        public void Dispatch(IEnumerable<IDomainEvent> events)
        {
            foreach (IDomainEvent ev in events)
            {
                Dispatch(ev);
            }
        }

        private void Dispatch(IDomainEvent ev)
        {
            switch (ev)
            {
                case StudentEmailChangedEvent emailChangedEvent:
                    _messageBus.SendEmailChangedMessage(
                        emailChangedEvent.StudentId,
                        emailChangedEvent.NewEmail);
                    break;

                // new domain events go here

                default:
                    throw new Exception($"Unknown event type: '{ev.GetType()}'");
            }
        }

```


## Many-to-many Relationships

Two one to Many Relation Ships
Join Table must be Elevated into an entity

=> Bad SoC

Give It Good Name, Own dedicated primary Key

## One-To-One RelationShip

Primary Id is Fk
Similar to Value Objects
Always prefer Value Object over one to One RelationShip






