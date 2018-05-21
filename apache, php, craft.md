
#### Git

install git:
`apt-get install git-core`

setup user config (to identify commits)
`git config --global user.name "testuser"`
`git config --global user.email "testuser@example.com"`

#### PHP

enable Ondrej's PPA:
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```

install PHP:
`sudo apt-get install php7.1`

list all available modules
`apt-cache pkgnames | grep php7.1` or `apt-cache search php7.1`

install mysql module
sudo apt-get install php7.1-mysql

enable mysql module
sudo phpenmod pdo_mysql
