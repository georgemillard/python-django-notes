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

`sudo apt-get -y install supervisor`

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

NAME="hive_mvp"
DIR=/home/django_mvp/hive_mvp
USER=django_mvp
GROUP=django_mvp
WORKERS=3
TIMEOUT=120
BIND_SOCKET=unix:/home/django_mvp/hive_mvp/run/gunicorn.sock
BIND_TCP=188.166.159.218:8000
DJANGO_SETTINGS_MODULE=hive_mvp.settings
DJANGO_WSGI_MODULE=hive_mvp.wsgi
LOG_LEVEL=info

echo "Starting $NAME"

cd $DIR
source ../bin/activate

export DJANGO_SETTINGS_MODULE=$DJANGO_SETTINGS_MODULE
export PYTHONPATH=$DIR:$PYTHONPATH

# exec ../bin/gunicorn hive_mvp.wsgi:application --bind 0.0.0.0:8000
# exec ../bin/gunicorn hive_mvp.wsgi:application --bind=unix:/home/django_mvp/hive_mvp/run/gunicorn.sock

exec ../bin/gunicorn ${DJANGO_WSGI_MODULE}:application \
  --name $NAME \
  --workers $WORKERS \
  --user=$USER \
  --group=$GROUP \
  --bind=$BIND_TCP \
  --log-level=$LOG_LEVEL \
  --log-file=-
```
