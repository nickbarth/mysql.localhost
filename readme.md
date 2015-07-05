# MySQL Docker Server Setup Notes

Some days you just want to create and destroy a bunch of MySQL Servers.

https://docs.docker.com/compose/

https://registry.hub.docker.com/u/mysql/mysql-server/

```terminal
# Uses mysql/mysql-server:latest

git clone --depth 1 https://github.com/nickbarth/mysql.localhost && cd mysql.localhost

docker-compose up database -d
docker-compose ps database

# Web Scale
docker-compose scale database=5
docker-compose ps -q database | parallel docker exec -t {} mysql --version

# Mass Import
docker-compose ps -q database | parallel 'docker exec -t {} \
  "mysql -h localhost -u root -psecretpw example" < database.sql'

# Mass Export
docker-compose ps -q database | parallel 'docker exec -t {} \
  "mysqldump -h localhost -u root -psecretpw example" > database.{}.sql'

# Local TCP
sed -i 's/\(database:\)/\1\n  ports:\n    - "3306:3306"/' docker-compose.yml
docker-compose up database -d
mysql -h localhost -u root -psecretpw --protocol=TCP example

# Make sure your Firewall Allows it
sed -i 's/\(DEFAULT_FORWARD_POLICY=\)"DROP"/\1"ACCEPT"/' /etc/default/ufw

# Works with PHPMyAdmin
wget https://www.phpmyadmin.net/downloads/

# config.inc.php

/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
/* Server parameters */
$cfg['Servers'][$i]['host'] = '127.0.0.1';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = 'secretpw';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['AllowNoPassword'] = false;

php -S localhost:5001

# Good Bye
docker-compose stop database
docker-compose rm database
```

![Web Scale](http://i.imgur.com/uQDttq2.jpg)

### License
The CC0 License

[![CC0](http://i.creativecommons.org/l/zero/1.0/88x31.png)](http://creativecommons.org/publicdomain/zero/1.0/)
