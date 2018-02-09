### Python and PostgresQL ###

```
sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib

sudo -u postgres psql

postgres=# CREATE DATABASE myproject;
postgres=# CREATE USER myprojectuser WITH PASSWORD 'password';
postgres=# ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
postgres=# ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
postgres=# ALTER ROLE myprojectuser SET timezone TO 'UTC';
postgres=# GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
postgres=# \q
```

Sidenote
On certain systems postgres gives this error: "FATAL: Peer authentication failed for user". This is caused by postgres
trying to authenticate via your local user password, to get around this, modify the config file at 
/etc/postgresql/9.5/main/pg_hba.conf from:

`local all postgres peer`

to:

`local all postgres md5`

Save the pg_hba_conf file, restart the service with sudo service postgresql restart and it should be good to go.

#### Useful postgres commands ####

```
ALTER USER Postgres WITH PASSWORD '';

\q - quit

\c <database_name> - connect to database <database_name>

\d <table_name> - show <table_name> definition

\dt - list tables

\l - list databases

\du - list users
```

See: https://gist.githubusercontent.com/Kartones/dd3ff5ec5ea238d4c546/raw/4f3d0ae1188128168758291b07382294f7f9922a/postgres-cheatsheet.md

#### Backup and restore postgres db ####

`pg_dump database_name -U database_user > backup.pgsql (if running over ssh add -h localhost for authentication to work)`

Then login to psql:

```
DROP DATABASE <database_name>;
CREATE DATABASE <database_name>;
GRANT ALL PRIVILEGES ON DATABASE <database_name> to <database_user>;
```

Back on the command line:

`psql <database_name> -U <database_user> -h localhost < backup.pgsql`

