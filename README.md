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
## Alienware M15 R7 And Java
```
// 12th Gen Intel(R) Core(TM) i7-12700H   2.30 GHz And Java
test_MockService_GetPerson_2nd_Time()	0.042s	passed
test_MockService_GetPerson_First_Time()	0.256s	passed
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
## C++ で MySQL に接続

とてつもなく、参考になったサイト、まんまやりました。

https://stackoverflow.com/questions/65748942/x-devapi-mysqlxsession-over-linux-socket-fails-with-cdk-error-unexpected-m
```
// MySQL で次のコマンドを行った。
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'user'@'localhost';
FLUSH PRIVILEGES;
mysql> show variables like 'mysqlx_socket';
```
## C++ でMySQLに接続できたよ
```
jack@jack-M14xR2:~/dev$ g++ -std=c++11 -I/usr/include/mysql-cppconn-8/ -L/usr/lib/x86_64-linux-gnu m14xr2-conn-mysql.cpp -lmysqlcppconn8 -o m14xr2-conn-mysql
jack@jack-M14xR2:~/dev$ ./m14xr2-conn-mysql 
If you read this message, you did connected by C++. Huge Congraturations !!
jack@jack-M14xR2:~/dev$

// 以下が、そのソースだよ。
/**
 * 最初はRDBMS、MySQLに接続することからだよね。
*/
#include <iostream>
#include <sstream>
#include <memory>
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_driver.h"
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_connection.h"
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_error.h"
#include "/usr/include/mysql-cppconn-8/jdbc/cppconn/statement.h"
#include "/usr/include/mysql-cppconn-8/jdbc/cppconn/resultset.h"

 #include "/usr/include/mysql-cppconn-8/mysqlx/xdevapi.h"

using namespace std;

int main() {
        mysqlx::Session sess("mysqlx://user:password@%2Ftmp%2Fubuntu-mysql-2%2Fdata%2Fmysqlx.sock");
        cout << "If you read this message, you did connected by C++. Huge Congraturations !!" << endl;
        return 0;
}
```
## M14xr2 Intel CPU 3rd GEN And C++ これが初めての計測結果だ。勝ったのか？ あってるのか？
```
jack@jack-M14xR2:~/dev$ ./m14xr2-conn-mysql 
If you read this message, you did connected by C++. Huge Congraturations !!
your schema is spring_demo
person count is 4
row is null ? 0
jack@loki.1677824762327.org
passed 0.002161sec.
jack@jack-M14xR2:~/dev$ 
```
## これが、最初の計測時のソースだ。
```
/**
 * 最初はRDBMS、MySQLに接続することからだよね。
*/
#include <iostream>
#include <sstream>
#include <memory>
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_driver.h"
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_connection.h"
#include "/usr/include/mysql-cppconn-8/jdbc/mysql_error.h"
#include "/usr/include/mysql-cppconn-8/jdbc/cppconn/statement.h"
#include "/usr/include/mysql-cppconn-8/jdbc/cppconn/resultset.h"

#include "/usr/include/mysql-cppconn-8/mysqlx/xdevapi.h"
#include "/usr/include/mysql-cppconn-8/mysqlx/devapi/collection_crud.h"
#include <time.h>

using namespace std;

int main() {
    clock_t start = clock();
    mysqlx::Session sess("mysqlx://user:password@%2Ftmp%2Fubuntu-mysql-2%2Fdata%2Fmysqlx.sock");
    cout << "If you read this message, you did connected by C++. Huge Congraturations !!" << endl;

    mysqlx::Schema schema = sess.getSchema("spring_demo");
    string schemaName = schema.getName();
    cout << "your schema is " << schemaName << endl;
    // mysqlx::Collection person = schema.getCollection("person");
    // mysqlx::Table person = schema.getCollectionAsTable("person");
    mysqlx::Table person = schema.getTable("person");
    cout << "person count is " << person.count() << endl;
    mysqlx::TableSelect selectOpe = person.select("email").where("pid=933");
    mysqlx::Row row = selectOpe.execute().fetchOne();
    cout << "row is null ? " << row.isNull() << endl;
    mysqlx::Value val = row.get(0);
    cout << val << endl;    
    sess.close();
    clock_t end = clock();
    cout << "passed " << (double)(end-start)/CLOCKS_PER_SEC << "sec." << endl;
    return 0;
}
```
