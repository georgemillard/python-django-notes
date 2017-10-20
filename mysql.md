#### Login

`mysql -u <username> -p`

#### List all users

`select User, Host from mysql.user;`

#### Change user password

`set password for '<username>'@'<host>' = password('<password>');
