# 9 Turning Optional Call into Calls on Optional Objects (or MayBe)

## Complicating the Requirements That Lead to Use of Nulls

When Null Object And Special Case are not applicable

## Identifying the Problem of a Nonexistent Objects

**Object substitution eliminates branching based on object state**

NullObject => Method on Object which is Null is always Called
Branching => Method on Object which is Null is never Called

## Representing Optional Object as a Collection

Collection which 0 or 1 object

``` C#

    static class EnumerableExtensions
    {
        public static void Do<T>(this IEnumerable<T> sequence, Action<T> action)
        {
            foreach (T obj in sequence)
                action(obj);
        }
    }

    class Option<T> : IEnumerable<T>
    {
        private IEnumerable<T> Content { get; }

        private Option(IEnumerable<T> content)
        {
            this.Content = content;
        }

        public static Option<T> Some(T value) => new Option<T>(new[] {value});

        public static Option<T> None() => new Option<T>(new T[0]);

        public IEnumerator<T> GetEnumerator() => this.Content.GetEnumerator();

        IEnumerator IEnumerable.GetEnumerator() => this.GetEnumerator();
    }
```

## Adding Pattern Matching to Options

Functional Style

```C#
          IOption<string> name = Option<string>.Some("something");

            name
                .When(s => s.Length > 3).Do(s => Console.WriteLine($"{s} long"))
                .WhenSome().Do(s => Console.WriteLine($"{s} short"))
                .WhenNone().Do(() => Console.WriteLine("missing"))
                .Execute();

            int length =
                name
                    .When(s => s.Length > 3).MapTo(s => s.Length)
                    .WhenSome().MapTo(s => 3)
                    .WhenNone().MapTo(() => 0)
                    .Map();

            Console.ReadLine();
```

## Heavyweight Implementation of Options with Pattern Matching

combinatorial explosion => To Heavy