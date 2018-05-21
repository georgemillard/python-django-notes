
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
`sudo apt-get install php7.1-mysql`

enable mysql module
`sudo phpenmod pdo_mysql`

modules can also be enabled with:
`a2enmod <module_name>`

Some sites suggest removing the `;` before the extension in the php.ini files (/etc/php/7.1/apache/php.ini) but this causes problems as the dll's are compiled for Windows - not 100% sure about this one :-)
