---
title: Dolt SQL Procedures
---

# Table of Contents

* [Dolt SQL Procedures](#dolt-sql-procedures)
  * [dolt\_add()](#dolt_add)
  * [dolt\_checkout()](#dolt_checkout)
  * [dolt\_commit()](#dolt_commit)
  * [dolt\_fetch()](#dolt_fetch)
  * [dolt\_merge()](#dolt_merge)
  * [dolt\_pull()](#dolt_pull)
  * [dolt\_push()](#dolt_push)
  * [dolt\_reset()](#dolt_reset)

# Dolt SQL Procedures

Dolt provides SQL stored procedures to allow access to `dolt` CLI 
commands from within a SQL session. Each procedure is named after the
`dolt` command line command it matches, and takes arguments in an
identical form.

For example, `dolt checkout -b feature-branch` is equivalent to
executing the following SQL statement:

```sql
CALL DOLT_CHECKOUT('-b', 'feature-branch');
```

SQL procedures are provided for all imperative CLI commands. For
commands that inspect the state of the database and print some
information, (`dolt diff`, `dolt log`, etc.) [system
tables](dolt-system-tables.md) are provided instead.

One important note: all procedures modify state only for the current
session, not for all clients. So for example, whereas running `dolt
checkout feature-branch` will change the working HEAD for anyone who
subsequently runs a command from the same dolt database directory,
running `CALL DOLT_CHECKOUT('feature-branch')` only changes the
working HEAD for that database session. The right way to think of this
is that the command line environment is effectively a session, one
that happens to be shared with whomever runs CLI commands from that
directory.


## `DOLT_ADD()`

Adds working changes to staged for this session. Works exactly like
`dolt add` on the CLI, and takes the same arguments.

After adding tables to the staged area, they can be committed with
`DOLT_COMMIT()`.

```sql
CALL DOLT_ADD('-A');
CALL DOLT_ADD('.');
CALL DOLT_ADD('table1', 'table2');
```

### Options

`table`: Table\(s\) to add to the list tables staged to be
committed. The abbreviation '.' can be used to add all tables.

`-A`: Stages all tables with changes.

### Example

```sql
-- Set the current database for the session
USE mydb;

-- Make modifications
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Stage all changes.
CALL DOLT_ADD('-a');

-- Commit the changes.
CALL DOLT_COMMIT('-m', 'committing all changes');
```

## `DOLT_CHECKOUT()`

Switches this session to a different branch.

With table names as arguments, restores those tables to their contents
in the current HEAD.

When switching to a different branch, your session state must be
clean. `COMMIT` or `ROLLBACK` any changes before switching to a
different branch.

```sql
CALL DOLT_CHECKOUT('-b', 'my-new-branch');
CALL DOLT_CHECKOUT('my-existing-branch');
CALL DOLT_CHECKOUT('my-table');
```

### Options

`-b`: Create a new branch with the given name.

### Example

```sql
-- Set the current database for the session
USE mydb;

-- Create and checkout to a new branch.
CALL DOLT_CHECKOUT('-b', 'feature-branch');

-- Make modifications
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Stage and commit all  changes.
CALL DOLT_COMMIT('-a', '-m', 'committing all changes');

-- Go back to main
CALL DOLT_CHECKOUT('main');
```


## `DOLT_COMMIT()`

Commits staged tables to HEAD. Works exactly like `dolt commit` with
each value directly following the flag.

`DOLT_COMMIT()` also commits the current transaction.

```sql
CALL DOLT_COMMIT('-a', '-m', 'This is a commit');
CALL DOLT_COMMIT('-m', 'This is a commit');
CALL DOLT_COMMIT('-m', 'This is a commit', '--author', 'John Doe <johndoe@example.com>');
```

### Options

`-m`, `--message`: Use the given `<msg>` as the commit message. **Required**

`-a`: Stages all tables with changes before committing

`--allow-empty`: Allow recording a commit that has the exact same data
as its sole parent. This is usually a mistake, so it is disabled by
default. This option bypasses that safety.

`--date`: Specify the date used in the commit. If not specified the
current system time is used.

`--author`: Specify an explicit author using the standard "A U Thor
author@example.com" format.

### Examples

```sql
-- Set the current database for the session
USE mydb;

-- Make modifications
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Stage all changes and commit.
CALL DOLT_COMMIT('-a', '-m', 'This is a commit', '--author', 'John Doe <johndoe@example.com>');
```


## `DOLT_FETCH()`

Fetch refs, along with the objects necessary to complete their histories
and update remote-tracking branches. Works exactly like `dolt fetch` on
the CLI, and takes the same arguments.

```sql
CALL DOLT_FETCH('origin', 'main');
CALL DOLT_FETCH('origin', 'feature-branch');
CALL DOLT_FETCH('origin', 'refs/heads/main:refs/remotes/origin/main');
```

### Options

`--force`: Update refs to remote branches with the current state of the
remote, overwriting any conflicting history

### Example

```sql
-- Get remote main
CALL DOLT_FETCH('origin', 'main');

-- Inspect the hash of the fetched remote branch
SELECT HASHOF('origin/main');

-- Merge remote main with current branch
CALL DOLT_MERGE('origin/main');
```


## `DOLT_MERGE()`

Incorporates changes from the named commits \(since the time their
histories diverged from the current branch\) into the current
branch. Works exactly like `dolt merge` on the CLI, and takes the same
arguments.

Any resulting merge conflicts must be resolved before the transaction
can be committed or a new Dolt commit created.

```sql
CALL DOLT_MERGE('feature-branch'); -- Optional --squash parameter
CALL DOLT_MERGE('feature-branch', '-no-ff', '-m', 'This is a msg for a non fast forward merge');
CALL DOLT_MERGE('--abort');
```

### Options

`--no-ff`: Create a merge commit even when the merge resolves as a fast-forward.

`--squash`: Merges changes to the working set without updating the
commit history

`-m <msg>, --message=<msg>`: Use the given as the commit message. This
is only useful for --non-ff commits.

`--abort`: Abort the current conflict resolution process, and try to
reconstruct the pre-merge state.

When merging a branch, your session state must be clean. `COMMIT`
or`ROLLBACK` any changes, then `DOLT_COMMIT()` to create a new dolt
commit on the target branch.

If the merge causes conflicts or constraint violations, you must
resolve them using the `dolt_conflicts` system tables before the
transaction can be committed. See [Dolt system
tables](dolt-system-tables.md##dolt_conflicts_usdtablename) for
details.

### Example

```sql
-- Set the current database for the session
USE mydb;

-- Create and checkout to a new branch.
CALL DOLT_CHECKOUT('-b', 'feature-branch');

-- Make modifications
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Stage and commit all  changes.
CALL DOLT_COMMIT('-a', '-m', 'committing all changes');

-- Go back to main
CALL DOLT_MERGE('feature-branch');
```


## `DOLT_RESET()`

Resets staged tables to their HEAD state. Works exactly like `dolt reset` on the CLI, and takes the same arguments.

Like other data modifications, after a reset you must `COMMIT` the
transaction for any changes to affected tables to be visible to other
clients.

```sql
CALL DOLT_RESET('--hard');
CALL DOLT_RESET('my-table'); -- soft reset
```

### Options

`--hard`: Resets the working tables and staged tables. Any changes to
tracked tables in the working tree since <commit> are discarded.

`--soft`: Does not touch the working tables, but removes all tables
staged to be committed. This is the default behavior.

### Example

```sql
-- Set the current database for the session
USE mydb;

-- Make modifications
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Reset the changes permanently.
CALL DOLT_RESET('--hard');

-- Makes some more changes.
UPDATE table
SET column = "new value"
WHERE pk = "key";

-- Stage the table.
CALL DOLT_ADD('table')

-- Unstage the table.
CALL DOLT_RESET('table')
```

## `DOLT_PUSH()`

Updates remote refs using local refs, while sending objects necessary to
complete the given refs. Works exactly like `dolt push` on the CLI, and
takes the same arguments.

```sql
CALL DOLT_PUSH('origin', 'main');
CALL DOLT_PUSH('--force', 'origin', 'main');
```

### Options

`--force`: Update the remote with local history, overwriting any conflicting history in the remote.

### Example

```sql
-- Checkout new branch
CALL DOLT_CHECKOUT('-b', 'feature-branch');

-- Add a table
CREATE TABLE test (a int primary key);

-- Create commit
CALL DOLT_COMMIT('-a', '-m', 'create table test');

-- Push to remote
CALL DOLT_PUSH('origin', 'feature-branch');
```


## `DOLT_PULL()`

Fetch from and integrate with another database or a local branch. In
its default mode, `dolt pull` is shorthand for `dolt fetch` followed by
`dolt merge <remote>/<branch>`. Works exactly like `dolt pull` on the
CLI, and takes the same arguments.

Any resulting merge conflicts must be resolved before the transaction
can be committed or a new Dolt commit created.

```sql
CALL DOLT_PULL('origin');
CALL DOLT_PULL('feature-branch', '--force');
```

### Options

`--no-ff`: Create a merge commit even when the merge resolves as a fast-forward.

`--squash`: Merges changes to the working set without updating the
commit history

`--force`: Ignores any foreign key warnings and proceeds with the commit.

When merging a branch, your session state must be clean. `COMMIT`
or`ROLLBACK` any changes, then `DOLT_COMMIT()` to create a new dolt
commit on the target branch.

If the merge causes conflicts or constraint violations, you must
resolve them using the `dolt_conflicts` system tables before the
transaction can be committed. See [Dolt system
tables](dolt-system-tables.md##dolt_conflicts_usdtablename) for
details.

### Example

```sql
-- Update local working set with remote changes
CALL DOLT_PULL('origin');

-- View a log of new commits
SELECT * FROM dolt_log LIMIT 5;
```