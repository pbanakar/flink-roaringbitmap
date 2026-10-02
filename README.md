# Flink RoaringBitmap UDFs

A collection of [RoaringBitmap](https://roaringbitmap.org/) user-defined functions (UDFs) for [Apache Flink](https://flink.apache.org/).

This project provides a temporary solution for RoaringBitmap support in Flink SQL before [FLIP-556](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=399278679) is formally supported in Flink. Before that, you can use the published UDF JARs with Flink 1.18 through Flink 2.x.

## Getting Started

### Prerequisites

- Apache Flink 1.18 or later (including Flink 2.x)
- Java 8 or later

### Installation

Download the latest release JAR from the [Releases](https://github.com/flink-extended/flink-roaringbitmap/releases) page, and add it to your Flink `lib/` directory or include it in your job's classpath.

### Register the UDFs

Run these statements in your Flink SQL environment before using the functions:

```sql
CREATE TEMPORARY FUNCTION rb_build_agg
    AS 'org.apache.flink.udfs.bitmap.RbBuildAggFunction';

CREATE TEMPORARY FUNCTION rb_cardinality
    AS 'org.apache.flink.udfs.bitmap.RbCardinalityFunction';

CREATE TEMPORARY FUNCTION rb_or_agg
    AS 'org.apache.flink.udfs.bitmap.RbOrAggFunction';
```

### Usage

```sql
-- Build a bitmap from a column of integer user IDs
SELECT rb_build_agg(user_id) FROM events GROUP BY dimension;

-- Get the unique visitor count from a stored bitmap
SELECT rb_cardinality(bitmap) FROM bitmaps;

-- Roll-up: union per-minute bitmaps into an hourly unique visitor count
SELECT
    hour_start,
    rb_cardinality(rb_or_agg(minute_bitmap)) AS unique_visitors
FROM minute_stats
GROUP BY hour_start;
```

## Building from Source

```bash
mvn clean package
```

The shaded JAR will be at `target/flink-roaringbitmap-0.1.0-SNAPSHOT.jar`.

## Using Fluss? Native Functions Available

If you are using [Apache Fluss](https://fluss.apache.org/) 1.0 or later, these functions are 
already built into `FlussCatalog`. No JAR download or `CREATE TEMPORARY FUNCTION` statement needed:

```sql
USE CATALOG fluss_catalog;

-- All rb_ functions are available immediately
SELECT rb_cardinality(rb_build_agg(user_id)) FROM events GROUP BY dimension;
```

See the [Fluss SQL Functions documentation](https://fluss.apache.org/docs/engine-flink/sql-functions/) 
for the full reference including `rb_xor_agg`, `rb_xor`, `rb_andnot`, and 9 other functions.

## License

This project is licensed under the [Apache License 2.0](LICENSE).
