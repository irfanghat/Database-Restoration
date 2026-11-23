## Restoring Postgres Data

See:
- [Environment Variables](/postgres/.env.template)
- [Docker Compose Configuration](/postgres/docker-compose.yaml)

### Working with `pg_dump`

We can dump the database contents as a **plain text** file that can be used to restore a Postgres database at a given time.

Example:

```sh
pg_dump -U <USER> -d <DATABASE> -f ./dumps/"$(date +'%Y%m%d_%H%M%S').sql"

# --------------------------------------------------
# Sample output:
# --------------------------------------------------
# abd966002a8a:/# ls dumps/
# 20261123_115451_.sql  20261123_115504.sql
# abd966002a8a:/# 
# --------------------------------------------------
```

We can also backup specified tables by providing a pattern for the table/s we would like to back up:

```sh
pg_dump -U <USER> -d <DATABASE> -t users -f ./dumps/"$(date +'%Y%m%d_%H%M%S').sql"
```

Restoring the database/ a database is therefore as straightforward as running the script via `psql` e.g. `cat ./dumps/2026.._.sql | psql -U <USER> -d <DATABASE>`

### Working with `pg_restore`

In order to use `pg_restore` with `pg_dump` we need to use *dump* files and not the **default plain text** sql files.
We can do this by specifying the desired output format as a **tar** file:


```sh
pg_dump -U <USER> -d <DATABASE> -F tar -f ./dumps/"$(date +'%Y%m%d_%H%M%S').tar"
```

This allows us to restore a database from an **archive** file.

## Restoring Corrupted Table Data

The following script will update the email column to `NULL` for emails ending with `@company.com`.

```sql
UPDATE users
SET email = NULL
WHERE email LIKE '%@company.com%';


-- Sample output:
--
--  id  | first_name  | last_name  | email 
-- -----+-------------+------------+-------
--    1 | John        | Doe        | 
--    2 | Jane        | Smith      | 
--    3 | Michael     | Johnson    | 
--    4 | Emily       | Brown      | 
--    5 | David       | Wilson     | 
--    6 | Sarah       | Taylor     | 
--    7 | James       | Anderson   | 
--    8 | Jessica     | Thomas     | 
--    9 | Robert      | Jackson    | 
--   10 | Ashley      | White      | 
--   11 | William     | Harris     | 
--   12 | Amanda      | Martin     | 
--   13 | Daniel      | Thompson   | 
--   14 | Jennifer    | Garcia     | 
--   15 | Matthew     | Martinez   | 
--   16 | Elizabeth   | Robinson   | 
--   17 | Christopher | Clark      | 
--   18 | Megan       | Rodriguez  | 
--   19 | Andrew      | Lewis      | 
--   20 | Rachel      | Lee        | 
--   21 | Joseph      | Walker     | 
-- --More-- 
```

We can now restore by running the following:

```sh
pg_restore -U user -c -C -d <DATABASE> e.g. postgres -v <BACKUP>.tar

# -c: Short for clean, this drops all objects before restoring the database before performing a restore. This is useful for overriding a database.
# -C: Short for create. This creates the database before restoring it.
# -d: The database to issue the initial DROP DATABASE  CREATE DATABASE command.
# -v: Verbose logging.

# ---------------------------------------------------
# Sample output:
# ---------------------------------------------------
# pg_restore: connecting to database for restore
# pg_restore: creating TABLE "public.users"
# pg_restore: creating SEQUENCE "public.users_id_seq"
# pg_restore: creating SEQUENCE OWNED BY "public.users_id_seq"
# pg_restore: creating DEFAULT "public.users id"
# pg_restore: processing data for table "public.users"
# pg_restore: executing SEQUENCE SET users_id_seq
# pg_restore: creating CONSTRAINT "public.users users_email_key"
# pg_restore: creating CONSTRAINT "public.users users_pkey"
# ---------------------------------------------------
```