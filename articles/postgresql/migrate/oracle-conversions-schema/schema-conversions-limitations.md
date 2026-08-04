---
title: Oracle to Azure Database for PostgreSQL flexible server Schema Conversion Limitations
description: Known limitations, unsupported objects, and constraints when using the Oracle to Azure Database for PostgreSQL schema conversion feature in Visual Studio Code with Microsoft Foundry integration.
author: apduvuri
ms.author: adityaduvuri
ms.reviewer: maghan
ms.date: 08/04/2026
ms.service: azure-database-postgresql
ms.topic: concept-article
ms.collection:
  - ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Oracle to Azure Database for PostgreSQL flexible server schema conversion limitations

This article summarizes the known limitations, unsupported objects, and migration considerations when you use the Oracle to Azure Database for PostgreSQL schema conversion feature in Visual Studio Code.

## Platform support limitations

- **ARM64**: Not supported on Windows or Linux.

## Unsupported and partially supported objects

The following sections list Oracle objects that the tool doesn't convert, or that it converts with a behavior difference you need to handle. Objects that convert cleanly aren't listed here.

These tables describe the typical outcome for each object class. An individual object can still need attention when it uses a construct that doesn't convert, such as a function that calls a Java routine, so the per-object report from your own conversion run is authoritative for your schema.

Oracle system and built-in schemas aren't extracted or converted. This includes schemas such as `SYS`, `SYSTEM`, `XDB`, `MDSYS`, `CTXSYS`, and `WMSYS`.

> [!NOTE]  
> Objects that the tool extracts but can't fully convert *are* added to the schema conversion report as **review tasks**. You can resolve those tasks manually or with GitHub Copilot agent mode assistance.

## Placeholder output

When the tool can't produce a working conversion for an object, it emits best-effort or placeholder DDL instead of dropping the object. The placeholder keeps the name and signature of the original so that objects that depend on it still convert and deploy. This approach prevents a small number of problem objects from blocking the rest of the schema.

A placeholder isn't functional. The schema conversion report flags every object that receives one as needing review before you deploy. Objects that depend on a placeholder can also be held back from the generated deployment script, so a single unconverted routine can affect several other objects.

The most common causes are:

- **Autonomous transactions.** Routines that use `PRAGMA AUTONOMOUS_TRANSACTION` are the largest single cause. The conversion routes the autonomous work through `dblink`, so if `dblink` isn't installed on the target when you convert, these routines fall back to placeholder output. Routines whose body isn't a simple `INSERT` followed by `COMMIT`, or that contain an exception handler or a `MERGE` statement, also need manual review.
- **A reference the tool couldn't resolve.** An object that references another object the tool didn't convert, or that it converted in a different unit of work, can't be compile-verified.
- **Transaction control statements.** Constructs such as `ROLLBACK TO SAVEPOINT` don't always convert cleanly.
- **Wrapped PL/SQL.** These always produce a placeholder. For more information, see [Wrapped PL/SQL](#wrapped-plsql).

## Programmable objects

| Object type | Limitation | What to do |
| --- | --- | --- |
| Package body | Package state maps to PostgreSQL session variables, so state that persists across calls doesn't behave the same as an Oracle package instance. | Test that shared state behaves as your application expects. |
| Routine that uses `PRAGMA AUTONOMOUS_TRANSACTION` | The autonomous work runs on a separate connection through `dblink`, so it commits independently of the caller and isn't rolled back if the caller fails. Several body shapes need manual review. | Install `dblink` before you convert, then review each routine to confirm that this difference is acceptable. |
| Trigger (system event) | The tool recognizes only `LOGON` and `LOGOFF` as system-event sub-types. `STARTUP`, `SHUTDOWN`, and `SERVERERROR` triggers aren't classified as system-event triggers, so they can pass into conversion and produce output that doesn't work on PostgreSQL. | Review the source for these triggers and reimplement the logic outside the database. |
| Trigger (cross-edition) | Edition-based redefinition isn't detected. A cross-edition trigger converts to an ordinary trigger, so it fires for every session instead of only for data manipulation that crosses editions. | Convert a single edition, and handle version rollout in your release process. |
| Type body | Not extracted. | Reimplement object-type methods as PostgreSQL functions. |
| Java source, Java stored procedure, and Java class | Not extracted. Azure Database for PostgreSQL flexible server can't run Java in the database. | Rewrite the logic in PL/pgSQL or move it to the application layer. |
| External C library (`CREATE LIBRARY`) | Not extracted. | Rewrite the logic in PL/pgSQL or move it to the application layer. |
| Wrapped PL/SQL | The body can't be read, so only a placeholder is produced. For more information, see [Wrapped PL/SQL](#wrapped-plsql). | Supply the original unwrapped source and convert again. |

### Wrapped PL/SQL

The tool can't read the source of PL/SQL units that are stored in wrapped (obfuscated) form by using the Oracle `wrap` utility or `DBMS_DDL.WRAP`. The tool detects these units, reads their signature metadata from the Oracle data dictionary, and generates a PostgreSQL routine that has the same name and signature and a body that raises an error when it's called.

This placeholder lets objects that depend on the wrapped unit convert and deploy. The generated routine contains no logic, and the schema conversion report lists every wrapped object as a review task that you must resolve before you go live.

To convert a wrapped unit properly, supply the original unwrapped source and run the conversion again.

## Data and storage objects

| Object type | Limitation | What to do |
| --- | --- | --- |
| User-defined operator (`CREATE OPERATOR`) | Conversion isn't guaranteed and can produce placeholder output. | Review the generated operator and test it. |
| Index (bitmap, bitmap join, global partitioned, or domain) | Not extracted. PostgreSQL doesn't use Oracle bitmap or domain indexes. | There's usually no functional action. Tune performance-sensitive queries on the target. |
| Materialized view log (`MLOG$`) | Not extracted. | Nothing. These are Oracle refresh support objects that PostgreSQL doesn't need. |
| External table | Not extracted. Azure Database for PostgreSQL flexible server has no server filesystem access, so `file_fdw` isn't available. | Load the file into a table with a client-side `\copy` or `COPY ... FROM STDIN`, with the `azure_storage` extension, or through your application. |
| Advanced Queuing (AQ) table | Not extracted. Oracle Advanced Queuing carries application logic. | Redesign it with an application queue, a PostgreSQL extension, or an Azure messaging service. |
| Blockchain table | Not extracted. | Reimplement the integrity requirement in the application layer. |
| Cluster and hash cluster | Not extracted. | Recreate as a plain or partitioned table. |
| Directory object (`CREATE DIRECTORY`) | Not extracted. | Handle file access in the application layer. |
| User-defined index type (`CREATE INDEXTYPE`) | Not extracted. | Recreate the access method with a PostgreSQL extension index where one exists. |
| Database link | Not converted. For more information, see [Database links](#database-links). | Recreate with a foreign data wrapper. |
| Application context (`CREATE CONTEXT`, `DBMS_SESSION.SET_CONTEXT`) | Not extracted. | Map the context to PostgreSQL session settings, and read them with `current_setting()`. |
| Analytic view, attribute dimension, hierarchy, OLAP dimension, data mining model, and materialized zone map | Not extracted. These have no PostgreSQL target. | Rebuild the required output as views or functions, or drop them. |
| LOB storage clause, registered XML schema (`DBMS_XMLSCHEMA.REGISTERSCHEMA`), and XML schema-bound index | Not extracted. | Remodel the storage or validation requirement on the target. |
| Resource Manager plan and consumer group, SQL profile, SQL plan baseline, edition, SQL Translation profile, Information Lifecycle Management policy, lockdown profile, and Workspace Manager object | Not extracted. These are Oracle instance, optimizer, or lifecycle artifacts that are out of scope. | Recreate them at the application or infrastructure layer, or drop them on Azure Database for PostgreSQL. |

### Database links

The tool doesn't automatically convert private and public database links (`CREATE DATABASE LINK`, `CREATE PUBLIC DATABASE LINK`). Database links store credentials and remote endpoint information that don't translate directly to PostgreSQL.

Recreate these links on the target by using a PostgreSQL foreign data wrapper such as `postgres_fdw` or `oracle_fdw`, or refactor to application-level connections. Search your Oracle source for `@link` references so that you find every dependency, including references inside synonyms and views.

## Security and identity objects

Users, roles, privileges, and data-protection policies are the largest gap in the conversion. The tool converts schema objects, not the security layer around them. Treat this area as a separate manual workstream, and plan it from your Oracle source inventory rather than from the schema conversion report.

| Object type | Limitation | What to do |
| --- | --- | --- |
| User | Not extracted. | Provision logins separately on Azure. Recreate each user as a PostgreSQL role, and validate the resulting access before you go live. |
| Role | Not extracted. | Recreate each role on the target. |
| System privilege | Not extracted. | Recreate as PostgreSQL `GRANT` statements. |
| Role membership | Not extracted. | Recreate as `GRANT <role> TO <role>` statements. |
| Object privilege and grant | Grant extraction is turned off by default, and covers object grants only. | Recreate grants from your source inventory, then validate them. |
| Row-level security and Virtual Private Database (`DBMS_RLS`) | Not converted. | Recreate the policies as PostgreSQL row-level security policies. |
| Oracle Label Security | Not converted. | Redesign the data-access boundaries on the target. |
| Data redaction (`DBMS_REDACT`) | Not converted. | Reapply masking on the target, in the application layer or with a PostgreSQL equivalent. |
| Profile, password policy, and unified or fine-grained audit policy (`DBMS_FGA`, `AUDIT POLICY`) | Not extracted. | Configure the equivalent Azure server parameters and auditing, and manage them as infrastructure as code. |

> [!IMPORTANT]  
> The tool doesn't carry over row-level security, Virtual Private Database, Oracle Label Security, and data redaction policies. A converted schema deploys and returns data without them, so rows that Oracle filtered are visible and columns that Oracle masked are returned in clear text. Recreate these policies and verify them before you route production traffic to the target.

## Scheduler objects

The tool doesn't convert Oracle scheduler metadata into working PostgreSQL schedules.

| Object type | Limitation | What to do |
| --- | --- | --- |
| `DBMS_SCHEDULER` job and schedule | Not converted. | Recreate the schedule on the target with `pg_cron`, or use an external scheduler such as Azure Logic Apps or Azure Functions. |
| Legacy `DBMS_JOB` job | Not detected. | Recreate the schedule with `pg_cron` or an application scheduler. |
| Program, chain, credential, file watcher, job class, group, window, and event-based job | Not converted. | Rebuild the orchestration in an external scheduler. |

## Feature-level gaps

The following items are Oracle features rather than object types. The tool either doesn't detect these features or converts them in a way that loses part of the original behavior, so they don't always appear as a review task.

| Feature | Limitation | What to do |
| --- | --- | --- |
| Hybrid partitioned table (Oracle 19.3 and later) | Converts as an ordinary table, and the external partitions are lost. | Identify hybrid partitioned tables in the source, and load the external data separately. |
| Editioning view and cross-edition trigger | Edition-based redefinition isn't detected. An editioning view converts as a plain view. | Convert one edition, then handle version rollout manually. |
| Immutable table (Oracle 19.11 and later) | Not detected, and there's no PostgreSQL equivalent. | Enforce the requirement in the application layer or with triggers. |
| Flashback Data Archive | Temporal history isn't modeled. | Use a history table pattern or a temporal extension on the target. |
| Flashback query (`AS OF TIMESTAMP`, `AS OF SCN`) | Not converted. | Rewrite queries that read historical versions of a row. |
| Oracle Sharding | Not modeled, and there's no single-database PostgreSQL equivalent. | Redesign the distribution strategy. |
| `BFILE` and SecureFile LOB specifics | Handled only as a generic note about exotic types. | Review the storage and access pattern for each column. |

Identity columns, virtual and generated columns, invisible columns, and global temporary tables are handled by the converter and aren't limitations. Private temporary tables exist only at runtime, so they don't appear in the source inventory.

## Get help

When you encounter limitations:

1. **Use GitHub Copilot agent mode** for guided assistance with review tasks.
1. **Consult PostgreSQL documentation** for alternative implementations.
1. **Review best practices** for Oracle to Azure Database for PostgreSQL migration patterns.
1. **Test in a scratch environment** before deploying to production.

## Related content

- [What is Oracle to Azure Database for PostgreSQL flexible server schema conversion?](schema-conversions-overview.md)
- [Best practices for Oracle to Azure Database for PostgreSQL flexible server schema conversion](schema-conversions-best-practices.md)
