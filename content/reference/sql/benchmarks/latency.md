---
title: Latency
---

# Latency and Throughput

Our approach to SQL performance benchmarking is to use `sysbench`, an
industry standard benchmarking tool.

## Performance Roadmap

Dolt is slower than MySQL. The goal is to get Dolt to within 2-4 times
the speed of MySQL for common operations. If a query takes MySQL 1
second, we expect it to take Dolt 2-4 seconds. Or, if MySQL can run 8
queries in 10 seconds, then we want Dolt to run 2-4 queries in 10
seconds. The `multiple` column represents this relationship with
regard to a particular benchmark.

It's important recognize that these are industry standard tests, and
are OLTP oriented. Many Dolt use-cases are non-OLTP, and Dolt is fast
for bulk operations common in, for example, data pipeline contexts.

## Benchmark Data

### Current Default Format (`__DOLT__`)

Below are the results of running `sysbench` MySQL tests against Dolt
SQL Server for the most recent release of Dolt in the current default 
storage format. We will update this with every release. The tests 
attempt to run as many queries as possible in a fixed 2 minute time 
window. The `Dolt` and `MySQL` columns show the median latency in 
milliseconds (ms) of each query during that 2 minute time window.

The Dolt version is `0.51.8`.

<!-- START___DOLT___LATENCY_RESULTS_TABLE -->
|       Read Tests        | MySQL |  Dolt  | Multiple |
|-------------------------|-------|--------|----------|
| covering\_index\_scan   |  2.03 |   2.91 |      1.4 |
| groupby\_scan           | 12.75 |  17.95 |      1.4 |
| index\_join             |  1.25 |    5.0 |      4.0 |
| index\_join\_scan       |  1.21 |   4.25 |      3.5 |
| index\_scan             | 31.94 |  56.84 |      1.8 |
| oltp\_point\_select     |  0.16 |   0.53 |      3.3 |
| oltp\_read\_only        |  3.13 |   9.56 |      3.1 |
| select\_random\_points  |  0.32 |   0.83 |      2.6 |
| select\_random\_ranges  |  0.38 |   1.21 |      3.2 |
| table\_scan             | 32.53 |  65.65 |      2.0 |
| types\_table\_scan      | 73.13 | 200.47 |      2.7 |
| reads\_mean\_multiplier |       |        |      2.6 |

|       Write Tests        | MySQL | Dolt  | Multiple |
|--------------------------|-------|-------|----------|
| bulk\_insert             | 0.001 | 0.001 |      1.0 |
| oltp\_delete\_insert     |  2.81 | 12.08 |      4.3 |
| oltp\_insert             |  1.55 |  3.13 |      2.0 |
| oltp\_read\_write        |  5.47 | 18.61 |      3.4 |
| oltp\_update\_index      |  1.52 |  5.88 |      3.9 |
| oltp\_update\_non\_index |  1.47 |  6.32 |      4.3 |
| oltp\_write\_only        |  2.35 |   8.9 |      3.8 |
| types\_delete\_insert    |  2.91 |  13.7 |      4.7 |
| writes\_mean\_multiplier |       |       |      3.4 |

| Overall Mean Multiple | 3.0 |
|-----------------------|-----|
<!-- END___DOLT___LATENCY_RESULTS_TABLE -->
<br/>
