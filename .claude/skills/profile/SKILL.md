---
name: profile
description: Use when a new source, schema or table arrives in SQL Server, to profile it before any pipeline code, view or report is built on it. Covers schema, row counts, nulls, duplicates, keys, bad values, relationships, gaps and personal data.
argument-hint: "[schema, table, or schema.table]"
---

# Profile

The job: understand the data before anything is built on it.
It writes nothing except the report.

Input: $ARGUMENTS, the name of a source. Look it up in the table below
to find where its tables are, and profile all of them.

| Source | Server      | Database | Tables     |
|--------|-------------|----------|------------|
| Moed   | BG-SQL-TEST | MRR_SOLD | all tables |

If the source is not in the table, stop and ask which database it is in.
Never guess a database.

## Steps

1. Count everything with real queries through sqlcmd. Never guess a number.
2. Connect with sqlcmd to the database of the source (-d from the table above).
   Schema: list all the tables, columns, data types, max lengths and nullability
   (INFORMATION_SCHEMA.COLUMNS, sys.tables). Get the row count of each table.
3. Keys: the declared primary keys and foreign keys (sys.key_constraints,
   sys.foreign_keys). For tables without a primary key, test which columns
   are unique.
4. Data quality per column: null count and %, distinct count, min and max,
   empty strings, columns that are always empty or always the same value.
   Count fully duplicated rows.
5. Bad values: numbers out of a sensible range, dates in the future or
   before 1900, placeholder values such as 9999-12-31, 0, -1, 'N/A', 'UNKNOWN'.
   Text stored in the wrong type, e.g. dates or numbers kept as VARCHAR.
6. Relationships: for every foreign key, declared or likely by name
   (e.g. CustomerID), count the orphans: child rows with no matching parent.
7. Gaps: for date columns, the days with no rows, and the latest date loaded.
8. Personal data: columns that look like names, emails, phones, ID numbers,
   addresses or bank details. Show their values masked, never in full.
9. Rank the findings by severity, each with the number of rows it affects
   and the decision it needs.
10. Write the result as one HTML file in reports/profile/,
    named <schema>_<date>.html: the key numbers on top, one section per check
    with a table or a chart, the ranked findings, and the open decisions last.

## Reads

- The tables and system views in SQL Server, read-only.

## Writes

- One HTML file in reports/profile/. Nothing else.

## Never

- Never runs anything but SELECT queries.
- Never changes data, tables or files of the project.
- Never guesses a type or a format: it's what the values show.
- Never shows personal data unmasked.
- Never pulls whole large tables into the chat: aggregate in SQL.
