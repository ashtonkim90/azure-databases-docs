---
title: Run parameterized queries
description: Run PostgreSQL queries with named and positional placeholders from the query editor.
author: mmcfarland
ms.author: mmcfarland
ms.reviewer: maghan
ms.date: 07/22/2026
ms.service: azure-database-postgresql
ms.subservice: extensions
ms.topic: how-to
---

# Run parameterized queries

Parameterized queries let you keep placeholders in SQL and provide values at run time. The PostgreSQL extension binds those values as query parameters; it doesn't paste values into the SQL text.

Use this page when you want to run SQL copied from tools or application code that use placeholders such as `:name`, `$1`, or `?`.

## Supported placeholder syntaxes

The query editor detects these placeholder styles outside strings, comments, casts, array slices, dollar-quoted bodies, and PostgreSQL JSON operators.

### Named placeholders

```sql
select id, email
from users
where id = :user_id;
```

Named placeholders are case-sensitive. Repeated occurrences of the same name share one grid row.

### PostgreSQL positional placeholders

```sql
select id, email
from users
where id = $1;
```

`$N` placeholders are positional within the statement that contains them.

### Qmark positional placeholders

```sql
select id, email
from users
where active = ?;
```

`?` placeholders work in left-to-right order. A `?` in any value
position acts as a parameter, including after comparison operators
(`>=`, `<=`, `<>`), in `CASE` branches, and in `LIMIT`/`OFFSET`. PostgreSQL's
JSONB operators `?`, `?|`, and `?&` and the JSON path operator `@?` are
recognized as operators, not parameters.

> [!IMPORTANT]  
> Use one placeholder style per statement. A statement that mixes `:name` with `$N`, or mixes `$N` with `?`, is rejected before execution.

## Open and use the Parameters tab

1. Open or create a `.sql` file and connect it to a database.
1. Run **Execute Query (PostgreSQL)**, **Execute Current Statement (PostgreSQL)**, or run a selected SQL range.
1. If the SQL contains placeholders, the **Parameters** tab opens in the bottom panel.
1. Enter a value for each row, choose a type if needed, and select **Run query**.
1. After the first run, edit values and select **Run again** to repeat the query.

# [Visual Studio Code](#tab/vscode)

:::image type="content" source="media/parameterized-queries/default-query-editor-parameters-tab.png" alt-text="Screenshot of parameters tab with placeholder rows, type dropdowns, NULL controls, and the Run query button." lightbox="media/parameterized-queries/default-query-editor-parameters-tab.png":::

# [Cursor](#tab/cursor)

:::image type="content" source="media/parameterized-queries/cursor-editor-query-editor-parameters-tab.png" alt-text="Screenshot of parameters tab with placeholder rows, type dropdowns, NULL controls, and the Run query button." lightbox="media/parameterized-queries/cursor-editor-query-editor-parameters-tab.png":::

---

The tab shows one row for each unique named placeholder and one row for each positional placeholder. Each row includes the placeholder name or index, a value input, a **NULL** checkbox, a type dropdown list, and row actions when available.

## Multi-statement scripts

**Note (May 2026):** earlier versions of this article incorrectly described positional indexes as per-statement-independent. The behavior didn't change; only the documentation is corrected.

Positional parameters (`$N`, `?`) share a single value array across the executed script. `$1` (or the first `?`) in any statement always binds to the same value as `$1` in any other statement. Reusing the same positional index across statements doesn't give them independent values. If you need different values for the same index in different statements, use named parameters (`:name`) instead.

If a shared named value isn't compatible with one of the statements that uses it, PostgreSQL returns the error and the grid keeps your values so you can adjust and run again.

## NULL values

Use the **NULL** checkbox to bind SQL `NULL`. When checked, the value field is ignored for that row.

If you type the literal text `NULL` while the **NULL** checkbox is off, the grid warns you that the value binds as the text `NULL`, not SQL `NULL`.

## Choose parameter types

The type dropdown list defaults to `auto`, which lets PostgreSQL infer the parameter type. Choose a type when you want client-side validation or clearer binding:

- `text`
- `integer`
- `bigint`
- `numeric`
- `boolean`
- `date`
- `timestamp`
- `timestamptz`
- `uuid`
- `json`
- `jsonb`

Validation is soft. A warning doesn't block submission; PostgreSQL remains the final validator at execution time.

## Generate a query plan with parameters

When you visualize a query plan for SQL that contains placeholders, the Parameters tab drives the query plan visualizer instead of returning rows. The run button reads **Visualize Query Plan**, and after the first run it reads **Visualize again**. Enter values and select the button to run `EXPLAIN` and open the [query plan visualizer](query-plan-visualizer.md). This path doesn't return query results.

## Use Ignore

Use **Ignore** when the grid shows a token that should stay in SQL, such as a valid PostgreSQL operator. Ignore is enabled only when the token remains valid SQL without binding.

## Edit SQL and run again

When you open the **Parameters** tab, you can edit the SQL and select **Run again**. The extension re-extracts placeholders and compares the new templated SQL with the previous fingerprint.

If the placeholder set changed, a drift banner summarizes what changed, such as placeholders added or removed. The extension merges values forward when the placeholder still matches by name or by positional index. If all placeholders are removed, the grid closes and the query runs normally.

## Cancel and recover transactions

While a parameterized run is active, the run button changes to a **stop** control (labelled **Cancel**). Cancel interrupts the in-flight batch, skips later batches, and leaves the **Parameters** tab open with values intact. A canceled run shows the **canceled** batch status rather than a failure, so its rows aren't highlighted as errors.

The extension doesn't auto-rollback user-started transactions. If cancellation leaves the connection in an aborted transaction state, the **Parameters** tab shows a recovery notice with **Execute `ROLLBACK`**. Select it to issue one explicit `ROLLBACK` on the same connection, then run the script again.

## Review failures and retry

When a parameterized run fails, the **Parameters** tab keeps your values and shows the failed status with the database error summary. Select **See Messages** to open the full message details.

Canceled runs show canceled status separately from failed runs, and later batches that didn't run are marked as skipped.

After you fix a value or type, select **Run again**. The tab clears stale failure, cancellation, and row-highlight state for the new attempt. If the connection is still in an aborted transaction, the recovery notice appears again.

## Query history value retention

The setting `pgsql.queryPlaceholders.historyValueRetention` controls whether parameter values are retained in the current session's in-memory query history:

| Value | Behavior |
| --- | --- |
| `ask` | Ask after each successful parameterized run. |
| `always` | Retain values for in-session history entries without prompting. |
| `never` | Retain templated SQL only. |

When `ask` is active, the prompt shown after a successful run offers **Save once** (retain this entry only), **Always save** (also switch the setting to `always`), **Skip** (templated SQL only), and **Don't ask again** (also switch the setting to `never`).

Values are kept in memory only and are cleared when VS Code reloads or the workspace changes. Parameter values are redacted from telemetry and logs.

## PREPARE caveat

`PREPARE ... AS SELECT $1` uses PostgreSQL server-side positional syntax. The extension detects `PREPARE` statements and leaves placeholders inside the `PREPARE` body for PostgreSQL instead of binding them on the client. Other statements in the same script are parsed normally.

## Unsupported MVP cases

The MVP doesn't include:

- Persistent disk-backed history of values.
- Named or saved parameter sets across editor sessions.
- Server-side `PREPARE`/`EXECUTE` reuse as client-side parameterized execution.
- Composite, array, bytea, range, interval, enum, or other type binding beyond the supported dropdown list types.

## Related content

- [Query editor and IntelliSense](query-editor-intellisense.md)
- [Query plan visualizer](query-plan-visualizer.md)
- [Quickstart: Connect and query PostgreSQL](quickstart-connect-query.md)
- [Settings reference](reference/settings.md)
