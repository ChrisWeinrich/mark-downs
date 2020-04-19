# 6 Avoiding Nulls with the Maybe Type

## The Billion-dollar Mistake

=> Dishonesty of method signature

MyClassOrNull

## Non-nullability on the Language Level

Not Possible on Compiler level due backwards compatibility

## Mitigating the Billion-dollar Mistake

introduce **maybe** class

```C#
    public struct Maybe<T> : IEquatable<Maybe<T>>
        where T : class
    {
        private readonly T _value;
        public T Value
        {
            get
            {
                if (HasNoValue)
                    throw new InvalidOperationException();

                return _value;
            }
        }

        public bool HasValue => _value != null;
        public bool HasNoValue => !HasValue;

        private Maybe([AllowNull] T value)
        {
            _value = value;
        }

        public static implicit operator Maybe<T>([AllowNull] T value)
        {
            return new Maybe<T>(value);
        }

        public static bool operator ==(Maybe<T> maybe, T value)
        {
            if (maybe.HasNoValue)
                return false;

            return maybe.Value.Equals(value);
        }

        public static bool operator !=(Maybe<T> maybe, T value)
        {
            return !(maybe == value);
        }

        public static bool operator ==(Maybe<T> first, Maybe<T> second)
        {
            return first.Equals(second);
        }

        public static bool operator !=(Maybe<T> first, Maybe<T> second)
        {
            return !(first == second);
        }

        public override bool Equals(object obj)
        {
            if (!(obj is Maybe<T>))
                return false;

            var other = (Maybe<T>)obj;
            return Equals(other);
        }

        public bool Equals(Maybe<T> other)
        {
            if (HasNoValue && other.HasNoValue)
                return true;

            if (HasNoValue || other.HasNoValue)
                return false;

            return _value.Equals(other._value);
        }

        public override int GetHashCode()
        {
            return _value.GetHashCode();
        }

        public override string ToString()
        {
            if (HasNoValue)
                return "No value";

            return Value.ToString();
        }

        [return: AllowNull]
        public T Unwrap([AllowNull] T defaultValue = default(T))
        {
            if (HasValue)
                return Value;

            return defaultValue;
        }
    }
```
## Enforcing the Use of the Maybe Type

**Fody.NullGuard**

```Xml
<?xml version="1.0" encoding="utf-8"?>
<Weavers>
  <NullGuard IncludeDebugAssert="false" />
</Weavers>
```

```C#
using NullGuard;

[assembly: NullGuard(ValidationFlags.All)]

namespace Nulls.Logic
{
    public class Initer
    {
    }
}
```

### Allowing Null

```C#
      [return: AllowNull]
        public T Unwrap([AllowNull] T defaultValue = default(T))
        {
            if (HasValue)
                return Value;

            return defaultValue;
        }
    }
```

## Limitations

*   Decide which assemblies should be weaved => Domain Logic






