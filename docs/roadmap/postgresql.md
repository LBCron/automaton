# PostgreSQL

Updated: 3 November 2026. Status: in progress. Work and status: parent issue [#584](https://github.com/rzvdaniel/automaton/issues/584) and its sub-issues; this file holds the design.

Automaton moves from MySQL to PostgreSQL before the [public launch](public-launch.md), while only our own two servers (test and production) need their data migrated. Decided 3 November 2026.

## Why

- Pomelo, the MySQL provider for EF Core, has no EF Core 10 release and looks abandoned. Oracle's provider breaks CAP's transactional outbox (CAP's MySQL storage needs MySqlConnector). The remaining MySQL option is a single-maintainer fork.
- Npgsql (EF Core provider), `DotNetCore.CAP.PostgreSql` and `Hangfire.PostgreSql` are mature and supported, and Npgsql follows EF Core releases closely.
- After the launch every database change would have to be migrated on servers we do not control.

## Design

- One PostgreSQL baseline migration replaces the MySQL migration history. New installations start on it; existing servers are moved once with the data migration below.
- Case sensitivity: MySQL compares text case-insensitively by default, PostgreSQL does not. Every lookup by email, user name, app name, slug, domain or similar is checked and keeps its current behavior (case-insensitive collation or normalized columns), with tests.
- MySQL-specific SQL (for example `GET_LOCK` for app deletion) becomes the PostgreSQL equivalent (advisory locks).
- Install, update, backup and restore use PostgreSQL (`pg_dump`/`pg_restore`). The encrypted backup download (#484) is built on the new format.
- Data migration: a tested, repeatable procedure that copies all tables from MySQL to PostgreSQL and verifies row counts, keys, logins, apps, secrets (still decryptable) and schedules. Test server first; production only with the owner's go-ahead and a backup.

## Plan

Each step depends on the previous one.

1. **Application on PostgreSQL** (#585) (on .NET 9): providers, baseline migration, raw SQL, case-insensitive lookups, test fixtures. *Acceptance:* all tests pass on PostgreSQL; a fresh database works end to end locally.
2. **Server installation on PostgreSQL** (#586): Compose, install, update, backup and restore scripts, install guide. *Acceptance:* a clean install, update, backup and restore work on PostgreSQL.
3. **Data migration of existing servers** (#587): procedure, verification, test server, then production. *Acceptance:* both servers run on PostgreSQL with all data, users can sign in, apps run, secrets decrypt.
4. **.NET 10** (#493, #494): retarget with Npgsql 10 after step 1.
