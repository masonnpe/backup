yum install libaio
mysql -uroot -p
/etc/mysql/mysql.conf/mysqld.conf
bind-address

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' identified by 'root' WITH GRANT OPTION;

FLUSH PRIVILEGES;