# 6 Turning Algorithms into Strategy Objects

## Identifying the Problem of a Varying Algorithm

Turn algorithm into replaceable component

Composition is more flexible, Inheritance reduces requirements => Favor Composition

```C#
        public CombiningPainter(IEnumerable<TPainter> painters, IPaintingScheduler<TPainter> scheduler)
                : base(painters)
        {
            base.Reduce = this.Combine;
            this.Scheduler = scheduler;
        }


```

## Dissecting the Algorithm

Alternative Implementation as a strategy

Template Method uses Strategies

## Identifying the Moving Parts of the Algorithm


## Analysis of the Template Method with Strategy Object

**Combining Painter Template Method**

```C#
    class CombiningPainter<TPainter> : CompositePainter<TPainter>
        where TPainter : IPainter
    {

        private IPaintingScheduler<TPainter> Scheduler { get; }

        public CombiningPainter(IEnumerable<TPainter> painters, IPaintingScheduler<TPainter> scheduler)
                : base(painters)
        {
            base.Reduce = this.Combine;
            this.Scheduler = scheduler;
        }


        private IPainter Combine(double sqMeters, IEnumerable<TPainter> painters)
        {
            IEnumerable<TPainter> availablePainters = painters.Where(painter => painter.IsAvailable).ToList();

            IEnumerable<PaintingTask<TPainter>> schedule = this.Scheduler.Schedule(sqMeters, availablePainters);

            TimeSpan time = schedule.Max(task => task.Painter.EstimateTimeToPaint(task.SquareMeters));

            double cost = schedule.Sum(task => task.Painter.EstimateCompensation(task.SquareMeters));

            return new ProportionalPainter()
            {
                TimePerSqMeter = TimeSpan.FromHours(time.TotalHours / sqMeters),
                DollarsPerHour = cost / time.TotalHours
            };
        }

    }
```

## Externalizing Strategy to a Separate Class

1. Evolving design
2. Refactor one step at the time
3. Traits of design improvements

Tuples in refactoring phase

**Painting Scheduler is strategy**

``` C#
    interface IPaintingScheduler<TPainter> where TPainter : IPainter
    {
        IEnumerable<PaintingTask<TPainter>> Schedule(double sqMeters, IEnumerable<TPainter> painters);
    }
```


## Implementing a Concrete Strategy Class

With Strategy pattern there is the possibility to add new functionality with new classes

```C#
    class ProportionalPaintingScheduler : IPaintingScheduler<ProportionalPainter>
    {
        public IEnumerable<PaintingTask<ProportionalPainter>> Schedule(double sqMeters,
            IEnumerable<ProportionalPainter> painters)
        {

            IEnumerable<Tuple<ProportionalPainter, double>> velocities =
                    painters
                        .Select(painter => Tuple.Create(painter, sqMeters / painter.EstimateTimeToPaint(sqMeters).TotalHours))
                        .ToList();

            double totalVelocity = velocities.Sum(tuple => tuple.Item2);

            IEnumerable<PaintingTask<ProportionalPainter>> schedule =
                velocities
                    .Select(tuple => new PaintingTask<ProportionalPainter>(tuple.Item1, sqMeters * tuple.Item2 / totalVelocity))
                    .ToList();

            return schedule;
        }
    }

```



