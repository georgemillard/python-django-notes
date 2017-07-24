# Hosting a Django project on Linode

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

`pip install uwgsi`

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

`uwsgi --http :8000 --module mysite.wsgi --virtualenv /path/to/virtual/env/`
