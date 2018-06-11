#### Login

`mysql -u <username> -p`

#### List all users

`select User, Host from mysql.user;`

#### Show databases

`show databases;`

#### Change user password

`set password for '<username>'@'<host>' = password('<password>');`

#### Grant user privileges

`mysql> GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';`
