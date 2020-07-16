# 10 Increasing Flexibility by Avoiding Switch Statements

## Adding Requirements that Lead to Multiway Branching

classical computed jump or nested if-then-elses

**problem => hardcoded !**

## Using the Old-fashioned Switch Instruction and an Enum

Bit Wise Operations

Avoid using enum !

```C#
    [Flags]
    public enum StatusRepresentation
    {
        AllFine = 0,
        NotOperational = 1,
        VisiblyDamaged = 2,
        CircuitryFailed = 4
    }
```


## Encapsulating Representation in a Separate Class

```C#
    sealed class DeviceStatus : IEquatable<DeviceStatus>
    {
        [Flags]
        private enum StatusRepresentation
        {
            AllFine = 0,
            NotOperational = 1,
            VisiblyDamaged = 2,
            CircuitryFailed = 4
        }

        private StatusRepresentation Representation { get; }

        private DeviceStatus(StatusRepresentation representation)
        {
            this.Representation = representation;
        }

        public static DeviceStatus AllFine() =>
            new DeviceStatus(StatusRepresentation.AllFine);

        public DeviceStatus WithVisibleDamage() =>
            new DeviceStatus(this.Representation | StatusRepresentation.VisiblyDamaged);

        public DeviceStatus NotOperational() =>
            new DeviceStatus(this.Representation | StatusRepresentation.NotOperational);

        public DeviceStatus Repaired() =>
            new DeviceStatus(this.Representation & ~StatusRepresentation.NotOperational);

        public DeviceStatus CircuitryFailed() =>
            new DeviceStatus(this.Representation | StatusRepresentation.CircuitryFailed);

        public DeviceStatus CircuitryReplaced() =>
            new DeviceStatus(this.Representation & ~StatusRepresentation.CircuitryFailed);

        public override int GetHashCode() => (int)this.Representation;

        public override bool Equals(object obj) => this.Equals(obj as DeviceStatus);

        public bool Equals(DeviceStatus other) =>
            other != null && this.Representation == other.Representation;

        public static bool operator ==(DeviceStatus a, DeviceStatus b) =>
            (object.ReferenceEquals(a, null) && object.ReferenceEquals(b, null)) ||
            (!object.ReferenceEquals(a, null) && a.Equals(b));

        public static bool operator !=(DeviceStatus a, DeviceStatus b) => !(a == b);
    }
```

# Using Encapsulated Representation as the Key

``` C#
    class SoldArticle
    {

        private IWarranty MoneyBackGuarantee { get; }
        private IWarranty NotOperationalWarranty { get; }
        private IWarranty CircuitryWarranty { get; set; }

        private IOption<Part> Circuitry { get; set; } = Option<Part>.None();

        private DeviceStatus OperationalStatus { get; set; }

        private IReadOnlyDictionary<DeviceStatus, Action<Action>> WarrantyMap { get; }

        public SoldArticle(IWarranty moneyBack, IWarranty express, IWarrantyMapFactory rulesFactory)
        {
            if (moneyBack == null)
                throw new ArgumentNullException(nameof(moneyBack));
            if (express == null)
                throw new ArgumentNullException(nameof(express));

            this.MoneyBackGuarantee = moneyBack;
            this.NotOperationalWarranty = express;
            this.CircuitryWarranty = VoidWarranty.Instance;

            this.OperationalStatus = DeviceStatus.AllFine();

            this.WarrantyMap = rulesFactory.Create(
                this.ClaimMoneyBack, this.ClaimNotOperationalWarranty, this.ClaimCircuitryWarranty);
        }

        private void ClaimMoneyBack(Action action)
        {
            this.MoneyBackGuarantee.Claim(DateTime.Now, action);
        }

        private void ClaimNotOperationalWarranty(Action action)
        {
            this.NotOperationalWarranty.Claim(DateTime.Now, action);
        }

        private void ClaimCircuitryWarranty(Action action)
        {
            this.Circuitry
                .WhenSome()
                .Do(c => this.CircuitryWarranty.Claim(c.DefectDetectedOn, action))
                .Execute();
        }

        public void InstallCircuitry(Part circuitry, IWarranty extendedWarranty)
        {
            this.Circuitry = Option<Part>.Some(circuitry);
            this.CircuitryWarranty = extendedWarranty;
            this.OperationalStatus = this.OperationalStatus.CircuitryReplaced();
        }

        public void CircuitryNotOperational(DateTime detectedOn)
        {
            this.Circuitry
                .WhenSome()
                .Do(c =>
                    {
                        c.MarkDefective(detectedOn);
                        this.OperationalStatus = this.OperationalStatus.CircuitryFailed();
                    })
                .Execute();
        }

        public void VisibleDamage()
        {
            this.OperationalStatus = this.OperationalStatus.WithVisibleDamage();
        }

        public void NotOperational()
        {
            this.OperationalStatus = this.OperationalStatus.NotOperational();
        }

        public void Repaired()
        {
            this.OperationalStatus = this.OperationalStatus.Repaired();
        }

        public void ClaimWarranty(Action onValidClaim)
        {
            this.WarrantyMap[this.OperationalStatus].Invoke(onValidClaim);
        }
    }
}
```

Triple Dispatch

## Turning Multiway Branching into a Dictionary Object

**DeviceStatus has to implement Iequatable to be used as key in dictionary**

# Substituting the Multiway Branching Object at Runtime

``` C#
    class CommonWarrantyRules : IWarrantyMapFactory
    {
        public IReadOnlyDictionary<DeviceStatus, Action<Action>> Create(
            Action<Action> claimMoneyBack, 
            Action<Action> claimNotOperational, 
            Action<Action> claimCircuitry) =>
            new Dictionary<DeviceStatus, Action<Action>>()
            {
                [DeviceStatus.AllFine()] =
                    claimMoneyBack,
                [DeviceStatus.AllFine().NotOperational()] =
                    claimNotOperational,
                [DeviceStatus.AllFine().WithVisibleDamage()] =
                    (action) => { },
                [DeviceStatus.AllFine().NotOperational().WithVisibleDamage()] =
                    claimNotOperational,
                [DeviceStatus.AllFine().CircuitryFailed()] =
                    claimCircuitry,
                [DeviceStatus.AllFine().NotOperational().CircuitryFailed()] =
                    claimNotOperational,
                [DeviceStatus.AllFine().CircuitryFailed().WithVisibleDamage()] =
                    claimCircuitry,
                [DeviceStatus.AllFine().NotOperational().WithVisibleDamage().CircuitryFailed()] =
                    claimNotOperational
            };

    }

    interface IWarrantyMapFactory
    {
        IReadOnlyDictionary<DeviceStatus, Action<Action>> Create(
            Action<Action> claimMoneyBack,
            Action<Action> claimNotOperational,
            Action<Action> claimCircuitry);
    }

```

