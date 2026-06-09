---
title: "CDC با Debezium و Kafka: راهنمای عملی"
date: 2026-05-10
description: "Change Data Capture یکی از قوی‌ترین الگوهای مهندسی داده است. این‌جا می‌نویسم چطور راه‌اندازیش کردم و چطور در پروداکشن نگهش می‌دارم."
tags: ["kafka", "debezium", "cdc", "postgresql", "clickhouse", "data-engineering"]
categories: ["مهندسی داده"]
author: "مصطفی قدیمی"
showToc: true
---

Change Data Capture (CDC) lets you treat your database's write-ahead log as a stream of events. Instead of polling tables or running batch ETL jobs, you get every insert, update, and delete in near real-time.

Debezium + Kafka is the most battle-tested CDC stack in the open-source world. Here's what actually running it looks like.

---

## The Architecture

```
PostgreSQL WAL
     │
     ▼
Debezium Connector (Kafka Connect)
     │
     ▼
Kafka Topic  (e.g. postgres.public.orders)
     │
     ▼
ClickHouse Sink Connector
     │
     ▼
ClickHouse Table (ReplacingMergeTree)
```

Each table gets its own Kafka topic. The topic name follows `{server}.{schema}.{table}`.

## PostgreSQL Setup

Debezium reads from the WAL using the `pgoutput` logical replication plugin (built into Postgres 10+).

```sql
ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM SET max_replication_slots = 10;

CREATE ROLE debezium REPLICATION LOGIN PASSWORD 'secret';
GRANT USAGE ON SCHEMA public TO debezium;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO debezium;

CREATE PUBLICATION dbz_publication FOR ALL TABLES;
```

## The Connector Config

```json
{
  "name": "postgres-source",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "secret",
    "database.dbname": "mydb",
    "database.server.name": "postgres",
    "plugin.name": "pgoutput",
    "snapshot.mode": "initial",
    "transforms": "unwrap",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.add.fields": "op,ts_ms",
    "transforms.unwrap.delete.handling.mode": "rewrite",
    "transforms.unwrap.drop.tombstones": "false"
  }
}
```

The `ExtractNewRecordState` transform is critical — without it every message wraps the row data in a nested `before`/`after` envelope.

## Handling Deletes in ClickHouse

ClickHouse's `ReplacingMergeTree` replaces rows with the same primary key. For deletes, I add a `_deleted` column and filter it in views:

```sql
CREATE TABLE orders_raw
(
    id           UInt64,
    status       String,
    amount       Decimal(18, 2),
    created_at   DateTime64(3),
    _deleted     UInt8 DEFAULT 0,
    _updated_at  DateTime64(3)
)
ENGINE = ReplacingMergeTree(_updated_at)
ORDER BY id;

CREATE VIEW orders AS
SELECT * FROM orders_raw FINAL
WHERE _deleted = 0;
```

## The Gotcha I Hit Most Often

**Replication slot lag.** If Kafka Connect goes down or falls behind, the PostgreSQL replication slot holds WAL segments on disk. On a busy database this fills your disk in hours.

Monitor it and alert aggressively:

```sql
SELECT slot_name,
       active,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS lag
FROM pg_replication_slots;
```

Alert if lag exceeds ~1 GB and have a playbook for safely dropping and recreating the slot when needed.
