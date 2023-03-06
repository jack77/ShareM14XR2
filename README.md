# Start at 2023-03-04

# ShareM14XR2
情報共有だよ。

```
docker run --name win-mysql-2 -e MYSQL_ROOT_PASSWORD=jack1234 -p 127.0.0.1:3306:3306 -d mysql:8.0.32-debian
```
```
docker exec -it win-mysql-2 /bin/bash
```
## Docker で mysqldump したファイルをオレにもよこせ。
```
docker exec -it win-mysql-2 /bin/bash
mysqldump -u root -p spring_demo > spring_demo.txt
docker cp win-mysql-2:/spring_demo.txt .
```
## Thanks 俺、Docker で mysqldump してもらった借りだ。
```
sudo docker exec -it ubuntu-mysql-2 /bin/bash
mysql -u root -p
create database spring_demo;
exit

sudo docker cp $HOME/downloads/spring_demo.txt ubuntu-mysql-2:/spring_demo.txt
mysql -u root -p spring_demo < spring_demo.txt
```
## Ubuntu でも　SqlDeveloper　使いたい
```
mkdir dev
unzip -d ~/dev ~/downloads/sqldeveloper-22.2.1.234.1810-no-jre.zip
dpkg-deb -x ~/downloads/mysql-connector-j_8.0.32-1ubuntu22.10_all.deb ~/dev
// SQL Developer
/home/jack/dev/sqldeveloper/sqldeveloper.sh

// JDBC Connector
dpkg-deb -x ~/downloads/mysql-connector-j_8.0.32-1ubuntu22.10_all.deb ~/dev
ls -al $HOME/dev/usr/share/java/mysql-connector-j-8.0.32.jar

// Cpp Connector
dpkg-deb -x $HOME/downloads/libmysqlcppconn9_8.0.31-1ubuntu22.04_amd64.deb ~/dev
```
## M14xr2 戦いの舞台は整ったようだな
- C++ でMySQLに接続（spring_demo)。
- 任意のレコードの登録（person）。
- タイム計測。

## 全然整ってなかったね :)
次のコマンドを実行して、C++のMySQLドライバとヘッダが利用可能になるはず。
```
sudo apt --fix-broken install ./mysql-community-client-plugins_8.0.32-1ubuntu22.04_amd64.deb
sudo apt --fix-broken install ./libmysqlcppconn8-2_8.0.32-1ubuntu22.04_amd64.deb
$ dpkg -L libmysqlcppconn8-2
.
.
.
/usr/lib/x86_64-linux-gnu/libmysqlcppconn8.so.2
jack@jack-M14xR2:~/downloads$ 
sudo apt --fix-broken install ./libmysqlcppconn-dev_8.0.32-1ubuntu22.04_amd64.deb
$ dpkg -L libmysqlcppconn-dev
```
