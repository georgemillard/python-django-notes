# Hosting a Django project on Linode

This guide has been written for Ubuntu 16.04

#### Configure hostname

```
echo "hostname" > /etc/hostname
hostname -F /etc/hostname

nano /etc/hosts
```
Add:
`ip-address   hostname.example.com  hostname`

#### Configure timezone

`dpkg-reconfigure tzdata`

#### Add normal user

`adduser example_user`

Add to sudo group: `adduser example_user sudo`

#### Setup SSH keys

On Mac:

`ssh-keygen -b 4096`

On Linux:

`mkdir ~/.ssh`
`nano ~/.ssh/authorized_keys`

On Mac:

`scp ~/.ssh/id_rsa.pub example_user@ip_address:~/.ssh/authorized_keys`

Then you can log in without having to enter a password!

#### SSH Demon settings

`sudo nano /etc/ssh/sshd_config`

Set `PermitRootLogin` and `PasswordAuthentication` to `no`

Then `sudo service ssh restart`

#### Check running processes

`sudo netstat -tulpn`

`ps aux | less`

`pgrep <process_name>`

End a process with `sudo apt-get purge program_name`

#### Fail2Ban

Default settings `/etc/fail2ban/fail2ban.conf`

Override by copying and editing to `/etc/fail2ban/fail2ban.local`

Default logs `/var/log/fail2ban.log`

To run commands in a client `sudo fail2ban-client -i`

Commands include `status, start, reload, stop, get logtarget, exit...`

Can also override jail.conf:

`awk '{ printf "# "; print; }' /etc/fail2ban/jail.conf | sudo tee /etc/fail2ban/jail.local`

`sudo apt-get install sendmail`

#### Python and PostgresQL

`sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib`

```
sudo -u postgres psql

postgres=# CREATE DATABASE myproject;
postgres=# CREATE USER myprojectuser WITH PASSWORD 'password';
postgres=# ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
postgres=# ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
postgres=# ALTER ROLE myprojectuser SET timezone TO 'UTC';
postgres=# GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
postgres=# \q
```
##### Sidenote
On certain systems postgres gives this error: "FATAL:  Peer authentication failed for user". 
This is caused by postgres trying to authenticate via your local user password, to get around this, modify the config file at `/etc/postgresql/9.5/main/pg_hba.conf` from:

`local   all         postgres                          peer`

to:

`local   all         postgres                          md5`

Save the pg_hba_conf file, restart the service with `sudo service postgresql restart` and it should be good to go.

#### Useful postgres commands:

ALTER USER Postgres WITH PASSWORD '<newpassword>';

`\q` - quit

`\c <database_name>` - connect to database <database_name>

`\d <table_name>` - show <table_name> definition

`\dt` - list tables

`\l` - list databases

`\du` - list users


See:
https://gist.githubusercontent.com/Kartones/dd3ff5ec5ea238d4c546/raw/4f3d0ae1188128168758291b07382294f7f9922a/postgres-cheatsheet.md


#### Venv + Django

Install virtualenv `sudo pip3 install virtualenv`
Create a new venv `virtualenv myprojectenv`
Install Django `pip install django psycopg2`
Start a new Django project `django-admin.py startproject myproject`
Configure DB in settings.py:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject',
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```
And add IP address to ALLOWED_HOSTS

Migrate DB and you're away:

```
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser ...
python manage.py runserver 0.0.0.0:8000
```

#### uWSGI

##### (install system wide - we will need this later to run uwsgi with sudo)
`sudo pip install uwgsi`

Create a test file:

```
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
```

Test with `uwsgi --http :8000 --wsgi-file test.py`

Test Django Project:

`python manage.py runserver 0.0.0.0:8000`

If that works, then test wtih uWSGI:

`uwsgi --http :8000 --module myproject.wsgi --virtualenv /path/to/virtual/env/`

(note - run this from django project root!)

### nginx

```
sudo apt-get install nginx
sudo /etc/init.d/nginx start
```

Test by visiting `<ip_address>:80` and you should see Welcome to nginx!

Create uwsgi_params in Django project root:

```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

Create mysite_nginx.conf:

```
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name .example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        root /path/to/your/mysite;  # path to folder containing /media folder
    }

    location /static {
        root /path/to/your/mysite; # path to folder containing /static folder
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

Symlink to this file to nginx can see it:

`sudo ln -s ~/path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/`

Restart nginx:

`sudo /etc/init.d/nginx restart`

Set static files setting in Django settings.py:

`STATIC_ROOT = os.path.join(BASE_DIR, "static/")`

Run `python manage.py collectstatic`

Upload a picture to your Django project media directory:

`/path/to/Django/Project/media/pic.jpg`

Restart nginx, then test if you can access it through nginx:

`sudo /etc/init.d/nginx restart` or `stop`, `start`

`<ip_address>:8000/media/pic.jpg`

Now we can run nginx with uwsgi:

`ct$ uwsgi --socket :8001 --module test_project.wsgi --virtualenv /home/gbmillard/projects/virtualenvs/asset_manager_venv/`

Visiting our django app on `<ip_address>:8000` and our test_project.wsgi on `<ip_address>:8001`. This will not be visible in the browser but the uwsgi terminal will show output for the request.

#### Flipping sockets!

Edit `mysitenginx.conf`:

```
server unix:///path/to/your/mysite/mysite.sock; # for a file socket
# server 127.0.0.1:8001; # for a web port socket (we'll use this first)
```

Then try:

`uwsgi --socket mysite.sock --wsgi-file test.py`

#### 502 Bad Gateway Error!!!

Using uwsgi with a .ini file (see below):

`uwsgi --ini mysite_uwsgi.ini`

Need to set permissions on test_project to 777, and on mysite.sock to 777 (within the ini file):

```
# mysite_uwsgi.ini file
[uwsgi]
#set DJANGO_SETTINGS_MODULE here if needed
env=DJANGO_SETTINGS_MODULE=test_project/settings_base
# Django-related settings
# the base directory (full path)
chdir           = /home/gbmillard/projects/test_project
# Django's wsgi file
wsgi-file       = /home/gbmillard/projects/test_project/test_project/wsgi.py
# the virtualenv (full path)
home            = /home/gbmillard/projects/virtualenvs/asset_manager_venv

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 10
# the socket (use the full path to be safe
socket          = /home/gbmillard/projects/test_project/sockets/mysite.sock
# ... with appropriate permissions - may be needed
chmod-socket    = 664
uid = gbmillard
gid = gbmillard
# clear environment on exit
vacuum          = true
```

Then we can run with `uwsgi --ini mysite_uwsgi.ini`

__Solution!!!__

The uid/gid flags from the .ini file were not being applied (verify by checking the permissions on mysite.sock while uwsgi is running)

To get the flags to apply, uwsgi must be run with SUDO!

`sudo uwsgi --ini mysite_uwsgi.ini`

The .ini can be updated: `chmod = 664` and all is well :-)

#### Emperor Mode

Create a folder to store config files: `sudo mkdir /etc/uwsgi/vassals/`

Create a simlink: `sudo ln -s /path/to/your/mysite/mysite_uwsgi.ini /etc/uwsgi/vassals/`

Run uwsgi in emperor mode: `sudo uwsgi --emperor /etc/uwsgi/vassals`

If your .ini does not specify uid and gid per project, add: `--uid www-data --gid www-data`

#### With Supervisor

`sudo apt-get install supervisor`

To restart: `sudo service supervisor restart`

Create a .conf file:

`/etc/supervisor/conf.d/uwsgi.conf`:

```
[program:uwsgi]
command=sudo uwsgi --emperor /etc/uwsgi/vassals
autostart=true
autorestart=true
stderr_logfile=/var/log/uwsgi.err.log
stdout_logfile=/var/log/uwsgi.out.log
```

After making any changes to a config file:

```
sudo supervisorctl reread
sudo supervisorctl update
```

Managing processes:

```
sudo supervisorctl
>start/stop/restart <process_name>
exit
```


#### UFW

`sudo apt-get install ufw`

https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw
https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands

```
sudo ufw default allow outgoing
sudo ufw default deny incoming

sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

sudo ufw status verbose
sudo ufw disable/enable
```
