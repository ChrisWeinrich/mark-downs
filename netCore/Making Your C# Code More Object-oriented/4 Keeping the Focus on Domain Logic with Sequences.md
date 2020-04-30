# 4 Keeping the Focus on Domain Logic with Sequences

## Outlining the Desired Solution

=> similar to natural language

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

Predicates Function which receives an object and returning a boolean

## Aggregating the Sequence, Improving the Readability, Improving Performance of Infrastructural Operations                                                       

Sorting not good for picking a element => O(NlogN) vs O(N)  
**Outsource Hard to understand code to Extension Methods !**


```C#
    internal static class EnumerableExtensions
    {
        public static T WithMinimum<T, TKey>(this IEnumerable<T> sequence, Func<T, TKey> criterion)
            where T : class
            where TKey : IComparable<TKey> =>
                sequence
                    .Select(obj => Tuple.Create(obj, criterion(obj)))
                    .Aggregate((Tuple<T, TKey>)null, (best, cur) =>
                            best == null || cur.Item2.CompareTo(best.Item2) < 0 ? cur : best)
                    .Item1;
    }
```


## Summary

Let Sequence loop through itself

