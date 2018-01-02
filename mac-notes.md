#### Postgres ####

Log on with: `psql -U postgres`

##### When you forget your password! (for version 9.6, update path according to your version) #####

http://cli23.blogspot.co.uk/2013/12/resetting-postgresql-postgres-user.html

Edit the pg_hba.conf file:

`sudo nano /Library/PostgreSQL/9.6/data/pg_hba.conf`

Change the "md5" method for all users to "trust" near the bottom of the file:

```
# "local" is for Unix domain socket connections only
local   all             all                                     trust
```

Find the name of the service:

`ls /Library/LaunchDaemons`

Stop the postgresql service:

`sudo launchctl stop com.edb.launchd.postgresql-9.6`

Start the postgresql service:

`sudo launchctl start com.edb.launchd.postgresql-9.6`

Start psql session as postgres:

`psql -U postgres`

(shouldn't ask for password because of 'trust' setting)

Reset password in psql session by typing:

```
ALTER USER postgres with password 'secure-new-password';
\q
```

Edit the pg_hba.conf file:

Switch it back to 'md5'

Restart services again
