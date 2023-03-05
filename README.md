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

