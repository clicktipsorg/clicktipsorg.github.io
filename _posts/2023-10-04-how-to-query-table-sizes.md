---
date: 2023-10-04 12:00:00
layout: post
title: How large are your ClickHouse tables?
subtitle: Learn how to query for ClickHouse table sizes!
description: How do you find out how large your ClickHouse tables are ?
image: /assets/img/ch_size.png
optimized_image: assets/img/ch_size.png"
category: clickhouse
tags:
  - clickhouse
  - operations
  - howto
author: bharatnc
---

How do you find out how large your ClickHouse tables are?

The `system.parts` table in ClickHouse contains all information regarding table sizes and more.
 
To find out the size of your tables, you can query for that information in the following manner:

```sql
SELECT table,
    formatReadableSize(sum(bytes)) as size,
    min(min_date) as min_date,
    max(max_date) as max_date
    FROM system.parts
    WHERE active
    GROUP BY table;
```

Sample output:
```sql
┌─table───────────────────┬─size───────┬───min_date─┬───max_date─┐
│ metric_log              │ 558.95 KiB │ 2023-10-05 │ 2023-10-05 │
│ trace_log               │ 22.57 KiB  │ 2023-10-05 │ 2023-10-05 │
│ query_log               │ 16.36 KiB  │ 2023-10-05 │ 2023-10-05 │
│ asynchronous_metric_log │ 315.40 KiB │ 2023-10-05 │ 2023-10-05 │
└─────────────────────────┴────────────┴────────────┴────────────┘
```

For more details about each table including information regarding bytes allocated for primary keys in memory (vs on disk), total bytes on disk, compressed bytes, etc. you can do the following:

```sql
SELECT 
    table,
    formatReadableSize(min(primary_key_bytes_in_memory_allocated)) AS min,
    formatReadableSize(median(primary_key_bytes_in_memory_allocated)) AS median,
    formatReadableSize(max(primary_key_bytes_in_memory_allocated)) AS max,
    formatReadableSize(sum(primary_key_bytes_in_memory)) AS in_memory,
    formatReadableSize(sum(primary_key_bytes_in_memory_allocated)) AS allocated,
    formatReadableSize(sum(bytes_on_disk)) AS total_size_on_disk,
    formatReadableSize(sum(data_compressed_bytes)) AS data_size_on_disk,
    count()
FROM system.parts 
GROUP BY table;
```

Sample output:

```sql
┌─table───────────────────┬─min───────┬─median────┬─max───────┬─in_memory─┬─allocated──┬─total_size_on_disk─┬─data_size_on_disk─┬─count()─┐
│ metric_log              │ 8.00 KiB  │ 8.00 KiB  │ 8.00 KiB  │ 966.00 B  │ 632.00 KiB │ 8.62 MiB           │ 3.65 MiB          │      79 │
│ trace_log               │ 8.00 KiB  │ 8.00 KiB  │ 8.00 KiB  │ 396.00 B  │ 264.00 KiB │ 162.10 KiB         │ 131.60 KiB        │      33 │
│ query_log               │ 8.00 KiB  │ 8.00 KiB  │ 8.00 KiB  │ 120.00 B  │ 80.00 KiB  │ 82.25 KiB          │ 46.10 KiB         │      10 │
│ asynchronous_metric_log │ 14.52 KiB │ 14.52 KiB │ 17.06 KiB │ 9.81 KiB  │ 1.24 MiB   │ 5.07 MiB           │ 5.01 MiB          │      85 │
└─────────────────────────┴───────────┴───────────┴───────────┴───────────┴────────────┴────────────────────┴───────────────────┴─────────┘
```


This system table is quite powerful and contains many important statistics. For more information, check out the ClickHouse [documentation](https://clickhouse.com/docs/en/operations/system-tables/parts) for the `system.parts` table.