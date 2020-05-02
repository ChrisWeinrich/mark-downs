# 5 Untangling Structure from Operations on Business Data

## Identifying the Problem of Selecting an Object

Collection of Object is another object !
Problems of using IEnumerable => Code Duplication

## Identifying the Problem of Synthesizing an Object

IPainter class could represent a **real person**  
IPainter class can also represent a **group of people**

=> Composite Pattern

```C#
   public interface IPainter
    {
        bool IsAvailable { get; }
        TimeSpan EstimateTimeToPaint(double sqMeters);
        double EstimateCompensation(double sqMeters);
    }

        class ProportionalPainter : IPainter
    {
        public TimeSpan TimePerSqMeter { get; set; }
        public double DollarsPerHour { get; set; }

        public bool IsAvailable => true;

        public TimeSpan EstimateTimeToPaint(double sqMeters) =>
            TimeSpan.FromHours(this.TimePerSqMeter.TotalHours*sqMeters);

        public double EstimateCompensation(double sqMeters) =>
            this.EstimateTimeToPaint(sqMeters).TotalHours * this.DollarsPerHour;
    }
```

## Understanding the Problems

Client wants to control everything and gets complex

**Tell don't Ask !**

## Treating Collection of Objects as an Object

Design the consuming end first
Exposed Methods should be atomic/primitive


## Implementing the Collection of Objects

```C#
    class Painters
    {

        private IEnumerable<IPainter> ContainedPainters { get; }

        public Painters(IEnumerable<IPainter> painters)
        {
            this.ContainedPainters = painters.ToList();
        }

        public Painters GetAvailable() => 
            new Painters(this.ContainedPainters.Where(painter => painter.IsAvailable));

        public IPainter GetCheapestOne(double sqMeters) =>
            this.ContainedPainters.WithMinimum(painter => painter.EstimateCompensation(sqMeters));

        public IPainter GetFastestOne(double sqMeters) =>
            this.ContainedPainters.WithMinimum(painter => painter.EstimateTimeToPaint(sqMeters));
    }
```

## Introducing the Compositional Function Idea

Idea of a composite:

Hides the Collection and exposes the **same** interface as the containing type

Goal: Encapsulate Concept or a collection. Callers don't have to know how to manipulate objects

Collection has been encapsulated with belonging methods


| Func/Action Delegate |  Abstract Method |
|-----|---|
|  Can be supplied from outside   |  Requires a derived class |
|  Can extend implementation without adding new class   | Needs new derived classes   |


```C#
    class CompositePainter<TPainter>: IPainter
        where TPainter : IPainter
    {
        public bool IsAvailable => this.Painters.Any(painter => painter.IsAvailable);

        private IEnumerable<TPainter> Painters { get; }

        private Func<double, IEnumerable<TPainter>, IPainter> Reduce { get; }

        public CompositePainter(IEnumerable<TPainter> painters,
                                Func<double, IEnumerable<TPainter>, IPainter> reduce)
        {
            this.Painters = painters.ToList();
            this.Reduce = reduce;
        }

        public TimeSpan EstimateTimeToPaint(double sqMeters) =>
            this.Reduce(sqMeters, this.Painters).EstimateTimeToPaint(sqMeters);

        public double EstimateCompensation(double sqMeters) =>
            this.Reduce(sqMeters, this.Painters).EstimateCompensation(sqMeters);
    }

        static class CompositePainterFactories
    {
        public static IPainter CreateCheapestSelector(IEnumerable<IPainter> painters) =>
            new CompositePainter<IPainter>(painters,
                (sqMeters, sequence) => new Painters(sequence).GetAvailable().GetCheapestOne(sqMeters));

        public static IPainter CreateFastestSelector(IEnumerable<IPainter> painters) =>
            new CompositePainter<IPainter>(painters,
                (sqMeters, sequence) => new Painters(sequence).GetAvailable().GetFastestOne(sqMeters));

        public static IPainter CreateGroup(IEnumerable<ProportionalPainter> painters) =>
            new CompositePainter<ProportionalPainter>(painters, (sqMeters, sequence) =>
            {
                TimeSpan time =
                    TimeSpan.FromHours(
                        1/
                        sequence
                            .Where(painter => painter.IsAvailable)
                            .Select(painter => 1/painter.EstimateTimeToPaint(sqMeters).TotalHours)
                            .Sum());
                double cost =
                    sequence
                        .Where(painter => painter.IsAvailable)
                        .Select(painter =>
                            painter.EstimateCompensation(sqMeters)/
                            painter.EstimateTimeToPaint(sqMeters).TotalHours*
                            time.TotalHours)
                        .Sum();
                return new ProportionalPainter()
                {
                    TimePerSqMeter = TimeSpan.FromHours(time.TotalHours/sqMeters),
                    DollarsPerHour = cost/time.TotalHours
                };

            });
    }
```