# 8 Leveraging Special Case Objects to Remove Null Checks

## Understanding the Problem of Null

Keeping Operations as near as possible to data !
Beware of Boolean Method arguments, Tell the object what to do and let them decide how !

## Outlining the Design Without Null References

object should know its state

## How to Never Return Null

```C#
    class SoldArticle
    {
        public IWarranty MoneyBackGuarantee { get; private set; }
        public IWarranty ExpressWarranty { get; private set; }
        private IWarranty NotOperationalWarranty { get; }

        public SoldArticle(IWarranty moneyBack, IWarranty express)
        {
            if (moneyBack == null)
                throw new ArgumentNullException(nameof(moneyBack));
            if (express == null)
                throw new ArgumentNullException(nameof(express));

            this.MoneyBackGuarantee = moneyBack;
            this.ExpressWarranty = VoidWarranty.Instance;
            this.NotOperationalWarranty = express;
        }
    }
```
### Null Object

```C#    
    class VoidWarranty : IWarranty
    {    
        public bool IsValidOn(DateTime date)=> false;
    }
```

## Demonstrating the Power of Null Objects

Null OBject has  no state or behavior, all instances are the same.

Implemented as Singleton:

```C#
    class VoidWarranty : IWarranty
    {
        [ThreadStatic]
        private static VoidWarranty instance;

        private VoidWarranty() { }

        public static VoidWarranty Instance
        {
            get
            {
                if (instance == null)
                    instance = new VoidWarranty();
                return instance;
            }
        }

        public void Claim(DateTime onDate, Action onValidClaim) { }
    }
```

## Introducing Special Cases

Providing an object which universally addresses one situation
No Data
Limited amount of behavior

Example: Registered User vs Anonymous User(Special Case Object)


```C#
    class LifetimeWarranty : IWarranty
    {

        private DateTime IssuingDate { get; }

        public LifetimeWarranty(DateTime issuingDate)
        {
            this.IssuingDate = issuingDate.Date;
        }

        public void Claim(DateTime onDate, Action onValidClaim)
        {
            if (!this.IsValidOn(onDate))
                return;
            onValidClaim();
        }

        private bool IsValidOn(DateTime date) =>
            date.Date >= this.IssuingDate;
    }
```

## Turning Boolean Query Methods into Real Operations

Design Interfaces which are forcing implementing classes to provide features

```C#

internal interface IWarranty
{
    void Claim(DateTime onDate, Action onValidClaim);
}

public void Claim(DateTime onDate, Action onValidClaim)
{
    if (!this.IsValidOn(onDate))
        return;
    onValidClaim();
}
```

## Substituting Objects at Run Time

No Branching over boolean conditions => polymorphic calls

```C#
    class SoldArticle
    {
        public IWarranty MoneyBackGuarantee { get; private set; }
        public IWarranty ExpressWarranty { get; private set; }
        private IWarranty NotOperationalWarranty { get; }

        public SoldArticle(IWarranty moneyBack, IWarranty express)
        {
            if (moneyBack == null)
                throw new ArgumentNullException(nameof(moneyBack));
            if (express == null)
                throw new ArgumentNullException(nameof(express));

            this.MoneyBackGuarantee = moneyBack;
            this.ExpressWarranty = VoidWarranty.Instance;
            this.NotOperationalWarranty = express;
        }

        public void VisibleDamage()
        {
            this.MoneyBackGuarantee = VoidWarranty.Instance;
        }

        public void NotOperational()
        {
            this.MoneyBackGuarantee = VoidWarranty.Instance;
            this.ExpressWarranty = this.NotOperationalWarranty;
        }
    }
```




