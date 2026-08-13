---
layout: post
tags: [posts-en]
title: Aggregating Account Codes with Oracle Hierarchical Queries (CONNECT BY)
description: >
  Notes on using Oracle hierarchical queries (START WITH, CONNECT BY, PRIOR, LEVEL) to roll up amounts by parent/child account code in a management accounting system.
---

In a management accounting system, account codes are usually organized as a tree: sub-categories hang off a top-level account, and finer accounts hang off those. On screen, the amount shown for a parent account needs to be the sum of all its child accounts. You could solve this parent/child rollup with recursive calls in application code, but in Oracle it's much cleaner to handle it in one shot with a hierarchical query.

## 1. Basic Hierarchical Query Syntax

```sql
SELECT LEVEL,
       LPAD(' ', (LEVEL - 1) * 2) || ACNT_NM AS ACNT_NM,
       ACNT_CODE,
       PARENT_CODE
FROM   ACNT_MST
START WITH PARENT_CODE IS NULL     -- start from the top-level accounts
CONNECT BY PRIOR ACNT_CODE = PARENT_CODE  -- parent-child join condition
ORDER SIBLINGS BY ACNT_CODE;
```

- **START WITH**: the condition for the root of the tree — here, top-level accounts with no parent.
- **CONNECT BY PRIOR**: defines the parent-child relationship. The side with `PRIOR` refers to the "previous level's (parent) row."
- **LEVEL**: a pseudo column starting at 1 for the root and incrementing as you go down. Often used for indentation.
- **ORDER SIBLINGS BY**: sorts within siblings at the same level while preserving the hierarchy. A plain `ORDER BY` would break the hierarchical ordering.

## 2. Rolling Up Amounts by Parent/Child Account Code

The core requirement is: "child account amounts need to be summed up into the parent account." I solved this by using a hierarchical query to get the list of descendant accounts for each account, then summing the transaction amounts belonging to that list.

```sql
SELECT M.ACNT_CODE,
       M.ACNT_NM,
       (SELECT NVL(SUM(T.AMOUNT), 0)
        FROM   TRAN_DTL T
        WHERE  T.ACNT_CODE IN (
                 SELECT ACNT_CODE
                 FROM   ACNT_MST
                 START WITH ACNT_CODE = M.ACNT_CODE
                 CONNECT BY PRIOR ACNT_CODE = PARENT_CODE
               )
       ) AS TOTAL_AMOUNT
FROM   ACNT_MST M
ORDER  BY M.ACNT_CODE;
```

For each account (M.ACNT_CODE), I get itself plus all of its descendants via `START WITH ACNT_CODE = M.ACNT_CODE ... CONNECT BY PRIOR`, then sum the transaction amounts belonging to that set of codes. A leaf account (with no children) ends up with just its own amount, while mid-level and top-level accounts automatically get the sum of all their descendants rolled up.

## 3. Points From Real-World Use

- **When aggregation conditions need to change dynamically on screen**: when users pick conditions like "this month only" or "a specific department" on screen, I bound those conditions into the `WHERE` clause of the inner `TRAN_DTL` subquery. Keeping the hierarchy (account tree) separate from the aggregation conditions (period/department) meant the hierarchical-query part could be reused unchanged whenever conditions changed, which made maintenance easier.
- **CONNECT BY LOOP error**: if data integrity breaks and account A's parent is B while B's parent is A again, you get `ORA-01436: CONNECT BY LOOP`. It's safer to add validation logic at the point where a parent code is entered, so this kind of circular reference can't happen in production data.
- **Performance**: the account tree itself is usually only a few hundred to a few thousand rows, so the hierarchical query is cheap on its own — but since the inner subquery scans the (large) transaction detail table for every single account, this can slow down as the number of accounts grows. The first things to check are whether the account code column is indexed, and whether the transaction-period condition allows for partition pruning.

## 4. Pivoting Into an Account × Department Matrix (PIVOT)

Since management accounting is fundamentally about "splitting P&L by business unit," the screen often needs a matrix layout — account codes as rows, departments as columns. The period gets filtered as a query condition, and spreading the data within that period across departments is handled with `PIVOT`.

```sql
SELECT *
FROM (
    SELECT ACNT_CODE,
           DEPT_CODE,
           AMOUNT
    FROM   TRAN_DTL
    WHERE  TRAN_DATE BETWEEN :FROM_DATE AND :TO_DATE   -- period is a query filter
)
PIVOT (
    SUM(AMOUNT)
    FOR DEPT_CODE IN ('D01' AS "Sales 1", 'D02' AS "Sales 2", 'D03' AS "Admin")
)
ORDER BY ACNT_CODE;
```

Each value listed after `FOR ... IN` becomes its own column, turning what was stacked as rows-per-department into a single row per account code. To go the other way — unfolding a matrix back into rows — you'd use `UNPIVOT`.

The practical snag here is that the department list in the `IN` clause isn't fixed — **users dynamically change which department or period they want to view on screen.** Since `PIVOT`'s `IN` clause has to be written out ahead of time in static SQL, I handled this with dynamic SQL that assembles the `IN` clause string to match the query conditions at runtime. If the number of departments itself varies screen to screen, it's also worth considering handling the pivot at the application layer instead (e.g., letting the screen's grid component transpose rows and columns itself from the query result).

## 5. Wrap-Up

The syntax for hierarchical queries is short, but once you nail down exactly which column defines the parent-child relationship, you can collapse logic that would otherwise need recursive calls at the application layer into a single SQL statement. It's worth knowing as a fundamental in any domain — like management accounting — that deals with account trees.
