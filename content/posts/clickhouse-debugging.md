+++
title = "The Belgian Malinois of databases"
date = "2026-09-19"
draft = false
author = "Manav"
+++

When I first setup and started working with ClickHouse all I knew was that it's an OLAP Database which understands SQL so it's OLAP and easier to talk to.

What nobody told me though, was that ClickHouse is the Belgian Malinois of databases. It will definitely do things Postgres (a mere golden retriever) simply cannot. It will also find a way to hurt itself if you stop paying attention for three weeks.

So let's learn to tame the Belgian Malinois of databases

## A tiny(ish) primer on ClickHouse
If you understand parts and merges and have a mental model of how keeper coordination works, feel free to skip this section

### The shape of Data
The most important thing which you first learn on working with any OLAP database is that they're columnar in nature.

Your query console shows you rows like any other database, but on disk each column lives together instead of each row.

So if you're storing customer ID, dosas ordered, and time taken to devour a plate of paniyarams, one customer's record doesn't live in one place. All the dosa numbers live together, all the paniyaram times live together. whereas in postgres, a single customer's info would've lived together

### Parts and Merges

In ClickHouse a table is an abstract concept, it's just a bunch of folders (parts) scattered in various places and when you ask ClickHouse for the number of Dosas sold last year or the average time taken to devour a plate of paniyarams, it loads up that data for you by looking inside each part. The more number of parts, the longer your dashboard takes to load

Every insert that you do, creates a part. Doesn't matter how many rows it has, 10 rows create a single part and so does 1 row.

ClickHouse also has a background merge queue, wherein it keeps merging smaller and smaller parts together but your rate of part production should always be way less than the merge rate.

### What if one instance goes down?

Now once you have a functioning ClickHouse setup, second step is reliability and durability, if you anticipate a huge read load on your instance or you cannot afford a single minute of downtime on your read or write path, you go for a replicated setup wherein you run an odd number of ClickHouse keepers and more than one ClickHouse instances which replicate your data.

Now even if one of your ClickHouse instances goes down or stops responding, your users can still access their data and continue writing into the functioning instance still.

ClickHouse keeper here simply acts as an append only log of information between your ClickHouse instances. When instance A receives a write, it tells keeper that part_0_0_1 exists, keeper writes it in something called a znode. Instance B wakes up, sees that it doesn't have this part and then goes and asks A for it. The actual data in that part is never transmitted to keeper

Also this is useful for co-ordination and leader election, When A says I am going to merge parts 10 to 25 it appends to keeper - 10 to 25 merged to 26. B then picks that up and performs the same operation.

---

Now that we have a decent understanding of clickhouse internals needed for this post, we can go ahead and see the various failure modes that come with it 

## Taming a single ClickHouse instance

As we know every insert creates a tiny little folder which ClickHouse needs to look into at query time. So if we create a bunch of little parts, ClickHouse will slowly open each part look inside and then do the aggregate.

So as the common wisdom goes kids, always batch your inserts. In any OLAP DB making a bunch of small inserts is a recipe for impending disaster

ClickHouse makes your life much easier here by allowing you to batch server side. Just use async_insert=1 and ClickHouse will batch your inserts for you, It even lets you tune the batch size via rows, or data size and whatnot. You can even block on inserts if you need it to be super durable.

Now as every good engineer knows, async flows bring a whole bunch of idiosyncrasies and failure modes with them.

### Debugging / Recon

Debugging async inserts can be a pain, lucky for you ClickHouse always maintains a nice running log of all async_insert failures queryable with all the info you'll need to narrow it down

#### Find errors in the overall pipeline
```sql
SELECT
    event_time,
    table,
    status,
    exception,
    rows,
    query
FROM system.asynchronous_insert_log
WHERE table IN ('<your_table>')
    AND exception != ''
ORDER BY event_time DESC
LIMIT 50;
```
Just substitute your table name and find whatever async insert errors have been plaguing your Belgian

This btw will also have all info of insert failures into any materialized views that you may have configured downstream of a table.

#### Am I creating too many parts
```sql
SELECT
    database,
    table,
    count() AS active_parts,
    avg(rows) AS avg_rows_per_part
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY active_parts DESC
```

ClickHouse's own warning threshold is 1,000 parts per partition

#### Are my queries finishing

```sql
SELECT
    toDate(event_time) AS day,
    count() AS attempts
FROM system.query_log
WHERE query LIKE '%<your_table>%'
    AND type = 'QueryFinish'
    AND event_time BETWEEN '<start_date>' AND '<end_date>'
GROUP BY day
ORDER BY day
```

#### Look at merges and parts both

```sql
SELECT
    database,
    table,
    toDate(event_time) AS day,
    countIf(event_type = 'NewPart') AS parts_created,
    countIf(event_type = 'MergeParts') AS merges
FROM system.part_log
WHERE table IN ('<table_a>', '<table_b>', '<table_c>')
    AND event_time BETWEEN '<start_date>' AND '<end_date>'
GROUP BY database, table, day
ORDER BY table, day
```

## Taming one or more ClickHouse instances

Now begins the fun part, once you decide to move from MergeTree to ReplicatingMergeTree, things have the potential to get messier

* The most important thing I've learnt working with Keeper is, it needs good fast local storage. Not network attached disks. The entire purpose of keeper is to be a fast key value store to maintain quorum and for that it needs to efficiently and quickly do I/O on its filesystem.

* Keeper also needs a lot of love and proper monitoring, a Keeper instance being overwhelmed with work means your tables' replication slowing down best case and the table going fully read only in the worst case.

### Debugging / Recon

#### Keeper system logs
Goldmine of information on connects/disconnects. Just check the syslogs for whatever is plaguing keeper

#### Keeper connection logs from ClickHouse

```sql
SELECT
    toDate(event_time) AS day,
    type,
    count() AS events
FROM system.zookeeper_connection_log
GROUP BY day, type
ORDER BY day ASC
```

#### Keeper client
clickhouse-keeper-client has a bunch of smaller useful commands like ls, cd, get, get_stat to navigate znodes - the way keeper stores information.


## Setting up a monitoring system so you don't need to go on debugging rabbit holes

Simplest and the most comprehensive way to setup monitoring is to enable the ClickHouse's built in metrics exporter which gives you almost every piece of information out of the box. Everything mentioned above and bit more.

Setup graphs and alerts and make sure your Malinois doesn't hurt itself

### Good things to alert on
*  **Rate of part creation** - Higher rate or increasing rate here is a decent warning of either higher incoming load or an application bug
*  **Rate of merges complete** - Low number here means something might be up
*  **Keeper znode count and memory use** - Keeper is a fairly resilient piece of software and before fully dying it gives you a lot of signs like high RAM usage. Alert on that
*  Alert on a table showing zero new parts alongside a high error rate

---

### Sane config to have when you deploy a clickhouse instance

ClickHouse stores a lottttt of logs in internal tables which fill up exponentially and by default there is absolutely no TTL on these tables.

Within a few months this can get to 100s of GBs just hogging storage.

Set a retention limit on ClickHouse's internal log tables by default, in server configuration.

Sane default tables to put ttls on text_log, part_log, trace_log, query_log, metric_log.

The exact TTL value depends on which all logs you want to preserve.


## Conclusion

So after all this debugging, it's important to realize that all of this isn't ClickHouse being bad at its job. It's doing something genuinely harder than Postgres does, quicker aggregations, much more amount of data being stored and processed.

But a Malinois that nobody walks is a Malinois that eats your sofa, and a ClickHouse cluster that nobody watches will go quiet for several weeks and not tell you.

Walk the dog.

---
The Rabokki (Ramen weds Tteokbokki) and Chicken Cheese dumplings at Kalsang Cafe might just be the the most delicious thing you can devour while maintaining your ClickHouse instance.