# ScrollSense Assignment 1 - Run Manual
## unzip the file
## A. One-command verification

Open a terminal in the submission directory and run:

```bash
python run_all.py
```

`run_all.py` recreates the database from scratch. It is therefore safe to use as the final pre-submission check.

On Windows PowerShell:

```powershell
cd .\ScrollSense_da24b005_ready
py .\run_all.py
```

## B. Run each part manually

### 1. Recreate the database

```bash
python generate_data.py --db scrollsense.db
```

The generator uses `DA24B005` as its deterministic seed.

### 2. Load the views

Python one-liner:

```bash
python -c "import sqlite3; c=sqlite3.connect('scrollsense.db'); c.executescript(open('views.sql',encoding='utf-8').read()); c.commit(); c.close()"
```

### 3. Check the schema

```bash
python check_submission.py
```

This checks SQLite version, relation count, foreign keys, views, and core constraints.

### 4. Run individual queries

Use any SQLite client (for example, DB Browser for SQLite) and open `scrollsense.db`.

Then open `queries.sql`.

For parameterized queries use:

- F5: `:tag = 'ai'`
- F6: `:creator_id = 3022` (the generator's highest-impression creator in the latest deterministic run; any valid creator can be used)
- F11: `:session_id = 1`

F3 has two intentionally different variants. Do not combine them: the assignment asks you to report both.

### 5. Transaction demonstrations

`transactions.sql` documents the exact SQL and the expected observation for each test.

T1 uses an intentionally invalid `signal_type` to force a CHECK failure inside a transaction.

T2 requires two open connections to the same `scrollsense.db`. Connection A starts `BEGIN IMMEDIATE` and inserts a moderation decision without committing. Connection B reads the view and should still see the last committed state because the write is not committed.

T3 demonstrates the handle-change trigger and then the single-writer rule. The Python harness reports the SQLite error name `SQLITE_BUSY` when the second connection writes while T2 holds the writer lock.

## C. What to submit

Submit the single assignment PDF plus the repository/ZIP containing the code files listed in `README.md`. The `.db` and test output are included for reproducibility but are not a replacement for the SQL/Python source.

## D. If a run fails

Delete `scrollsense.db` and rerun `python run_all.py`. The script is designed to start from an empty database file.

The implementation requires SQLite 3.44+ because the brief explicitly relies on ordered `group_concat` syntax and other later SQLite features.
