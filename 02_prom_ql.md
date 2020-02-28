# The expression language

With more metrics collected by your Prometheus server, it is time to
familiarize yourself a bit more with the expression language. For comprehensive
documentation, check out the
[querying chapter](http://prometheus.io/docs/querying/basics/). The following
is meant as an inspiration for how to play with the metrics currently collected
by your server. Evaluate them in the *Console* and *Graph* tab. For the latter,
try different time ranges and the *stacked* option.

## The `rate()` function
Node exporter exposes CPU information for all of the installed CPUs on the
target system. The metric is named `node_cpu_seconds_total` since version
0.16; it was called `node_cpu` in the earlier versions. It is comprised
of multiple counters, one per each combination of mode and cpu_id.

These counters are an ever-increasing count of seconds that the CPU spent
in the particular mode. To see a rate of the time spent in each mode every
seocnd, use the rate function over a time range that should cover at least
a handful of scrape intervals.

```
rate(node_cpu_seconds_total[1m])
```

Now you can see the rate for each mode.  Note that the rate
function handles counter resets (for example if a binary is restarted).
Whenever a counter goes down, the function assumes that a counter reset has
happened and the counter has started counting from `0`.

## The `sum` aggregation operator
If you want to get the total rate for all modes, you need to sum up the
rates:

```
sum(rate(node_cpu_seconds_total[1m]))
```

Note that you need to take the sum of the rate, and not the rate of the sum.
(Exercise for the reader: Why?)

This query tells you the number of CPUs present, albeit by a rather
contrived way.

## Select by label
If you want to look only at the `user` mode, you can filter by label with
curly braces:

```
rate(node_cpu_seconds_total{mode="user"}[1m])
```

You can use multiple label pairs within the curly braces (comma-separated), and
the match can be inverted (with `!=`) or performed with a regular expression
(with `=~`, or `!~` for the inverted match).

## Aggregate by label
If you are only interested in which mode is being used the most regardless
of which CPU it uses, you can let the sum operator aggregate by mode.

```
sum(rate(node_cpu_seconds_total[1m])) by (mode)
```

A combination of label pairs is possible, too. You can aggregate by mode and
instance (which is interesting if you have added an additional node exporter to
your config):

```
sum(rate(node_cpu_seconds_total[1m])) by (mode, instance)
```

Note that there is an alternative syntax with the `by` clause following
directly the aggregation operator. This syntax is particularly useful in
complex nested expressions, where it otherwise becomes difficult to spot which
`by` clause belongs to which operator.

```
sum by (mode, instance) (rate(node_cpu_seconds_total[1m]))
```

## Arithmetic
If you want to show percentages of the time spent in different CPU modes,
and don't care about the number of CPUs in the machine, you can divide the
per-second values by the number of distinct CPUs on the machine.

```
sum by (mode, instance) (rate(node_cpu_seconds_total[1m]))
 / count(node_cpu_seconds_total) by (instance, mode)
```

Things become more interesting if the labels do not match perfectly
between two instant vectors or you want to match vector elements in a
many-to-one or one-to-many fashion. See the
[vector-matching section](http://prometheus.io/docs/querying/operators/#vector-matching)
in the documentation for details.

## Recording rules
In your practical work with Prometheus at scale, you will pretty soon run into
expressions that are very expensive and slow to evaluate. The remedy is
*recording* rules, a way to tell Prometheus to pre-calculate expressions,
saving the result in a new time series, which can then be used instead of the
expensive expression. See the documentation for details:
* [General documentation about rules](http://prometheus.io/docs/querying/rules/).
* [Best practices for naming rules](http://prometheus.io/docs/practices/).
