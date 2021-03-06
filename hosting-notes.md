# Hosting notes - Ubuntu 16.04

### Create a new user

```
adduser hive_user
gpasswd -a hive_user sudo
su - hive_user
```

### Pyhton setup

```
sudo apt-get update
sudo apt-get -y upgrade
```

some suggested packages:

```
sudo apt-get install 
  build-essential
  libssl-dev
  libffi-dev
  libpq-dev
  python-dev
```

check python3 version:
`python3 -V`

install pip for python3:
`sudo apt-get install -y python3-pip`

install venv for python3:
`sudo apt-get install -y python3-venv`

make a venv directory:
```
mkdir virtualenvironments
cd virtualenvironments
```

create new virtual environment:

`python3 -m venv hive_venv`

activate environment:

`source hive_venv/bin/activate`

within a virtual environment you can use `pip` and `python` instead of `pip3` and `python3`

to capture a virtual environment:

`pip freeze > requirements.txt`

to install it:

`pip install -r requirements.txt`

### NGINX

`sudo apt-get -y install nginx`

### Supervisor

```
sudo apt-get -y install supervisor
cd ~
mkdir logs
touch logs/gunicorn-error.log
sudo nano /etc/supervisor/conf.d/myproject.conf
```

myproject.conf:

```
[program:myproject]
command=/home/user/bin/gunicorn_start
user=user
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/user/logs/gunicorn-error.log
```

supervisor commands:

```
sudo supervisorctl reread
sudo supervisorctl update
sudo supercisorctl status

sudo supervisorctl start/stop/restart [project]
```

### Postgres

```
su - postgres
createuser hive_user
createdb hive_db
psql -c "ALTER USER hive_user WITH PASSWORD 'password'"
```

### Gunicorn

`pip install gunicorn`

/home/user/bin/gunicor_start:

nb. TCP works, socket doesn't (502 error) I haven't yet worked out why this is...

```
#!/bin/bash

NAME="project_name"
DIR=/home/user/project_name
USER=user
GROUP=group
WORKERS=3
TIMEOUT=120
BIND_SOCKET=unix:/home/user/run/gunicorn.sock
BIND_TCP=ip_address:8000
DJANGO_SETTINGS_MODULE=project.settings
DJANGO_WSGI_MODULE=project.wsgi
LOG_LEVEL=info

echo "Starting $NAME"

cd $DIR
source ../path_to_virtual_env/bin/activate

export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DIR:$PYTHONPATH

# exec ../path_to_virtual_env/bin/gunicorn project.wsgi:application --bind 0.0.0.0:8000
# exec ../path_to_virtual_env/bin/gunicorn project.wsgi:application --bind=unix:/home/user/project/run/gunicorn.sock

exec ../path_to_virtual_env/bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $WORKERS \
  --user=$USER \
  --group=$GROUP \
  --bind=$BIND_TCP \
  --log-level=$LOG_LEVEL \
  --log-file=-
```

make the gunicorn_start file executable:

`chmod u+x /home/user/bin/gunicorn_start`

### Add ip address to django settings ALLOWED_HOSTS['...']

### Install Nodejs
```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version
```

### Add node process to Supervisor
```
/etc/supervisor/conf.d/node-project.conf

[program:node-project]
directory=/path/to/node/project/dir #containing package.json
command=npm start --production
autostart=true
autorestart=true
stderr_logfile=/var/log/node-project.err.log
```

`sudo supervisorctl`
`reread`
`update`
