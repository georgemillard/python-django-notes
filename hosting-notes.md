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

