# Hosting notes - Ubuntu 16.04

### Create a new user

`adduser hive_user`

### Pyhton setup

```
sudo apt-get update
sudo apt-get -y upgrade
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

