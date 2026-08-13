---
layout: post
author: author1_en
tags: [posts-en]
title: What Happened When We Dropped the Wrong Partitions (Global Index UNUSABLE)
description: >
  Notes on recovering from a pre-launch partition-drop test that wiped out the current month's partition and broke login testing, and the global index UNUSABLE issue and REBUILD fix that followed.
---

Notes on something that happened while working with partitioned tables on an OAM (integrated authentication) project. **This occurred during pre-launch verification (production-DB testing), so real end users were never affected** — but since it's a login-processing system, login itself broke during verification testing.

## 1. What Happened

The table in question wasn't one we'd designed ourselves — it was **a table provided by the OAM solution itself**. Tables like this typically don't have their future partitions created manually ahead of time; instead, a scheduler registered via `DBMS_JOB` periodically creates the next partition automatically.

A junior developer was assigned to run a drop test to clean up old partitions, but the target range was specified incorrectly, and **partitions from the current month through future months were wiped out entirely.** The intended target was past partitions that were no longer needed, but the current month's partition — where incoming data needed to land — and the future partitions `DBMS_JOB` had pre-created were swept away along with them.

## 2. The Error Changed Twice

The first thing I found in the login failure logs was this:

```
ORA-14400: inserted partition key does not map to any partition
```

Since there was no partition left to hold the current month's data, the INSERT was rejected outright with "can't find a partition for this value." The cause was clear, so I re-created the partitions spanning past through future via script.

But even after re-creating the partitions, login still didn't work. Checking the logs again, the error had changed to something else:

```
ORA-01502: index 'SCHEMA.IDX_NAME' or partition of such index is in unusable state
```

This time the index was in an UNUSABLE state. The partition-drop operation had broken the global index sitting on that table. Re-creating the partitions didn't automatically fix an index that had already been broken.

A global index is a single index spanning the whole table regardless of its partition structure, so a partition-level operation like `DROP PARTITION` (without the `UPDATE INDEXES` clause) drops the entire index into an UNUSABLE state — no matter how many partitions were actually dropped, the whole index breaks at once.

## 3. The Fix: Rebuild the Index

```sql
ALTER INDEX idx_name REBUILD ONLINE PARALLEL NOLOGGING;
```

Only after rebuilding the index did login start working normally again. I'd been focused on the partition issue alone, and as soon as I fixed that, a completely different error (the index) popped up right away — I was briefly stumped wondering "I fixed the partitions, why is this still broken?" It was a very direct way to learn that dropping partitions reaches all the way into the index too.

## 4. What Changed Afterward

Actually, before this incident, during a DB acceptance review, I'd already been advised to use a local index instead of a global index on this table. But an Oracle engineer had said, "tables created by the solution vendor may have their own reasons for being the way they are — better not to touch them casually," so at the time I left the recommendation unapplied. It was only after going through this incident myself that I truly understood, viscerally, why that recommendation existed.

Since this incident, I now default to considering a local index first whenever I'm indexing a partitioned table. A local index maps 1:1 to the table's partitions, so when a specific partition is dropped, only the corresponding slice of the index disappears with it — the rest stays intact. If a global index is unavoidable, adding the `UPDATE INDEXES` clause to partition operations is another way to keep it from ever dropping into an UNUSABLE state in the first place.

I also took away a broader lesson: for any operation that changes the data structure itself, like partitioning, testing needs one more explicit review pass over the blast radius beforehand — in this case, "does the drop range include the current month's or future partitions?"
