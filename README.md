# Start at 2023-03-04

# ShareM14XR2

## 検証動機

M14xR2 この名前、アルファベットと数字の羅列にピンときた貴方はきっと Dell ユーザなのでしょう。5 年近くこのマシンは自宅で放置されていた。様々な理由はあるが、大きくは次の2点だろう。
ヒンジの破損と汚れ、特に汚れ。M15 R7の購入をきっかけに、M14xR2もついでに修理とクリーニングをやろうと思った。業者を巡り、自身でも新たにPCを購入したので、現在のマシンスペック
がいかなるものか、そして、修理・クリーニングしようとしているマシンとのスペック差がどれほどのものか、痛感した。最低価格の廉価版マシンにボコられる、CPU差なのだと。Intel Core i7は
現在12世代で、このこは第3世代だ。比べるのも馬鹿らしいと思われた。自分が聞いたのが悪いが「三倍遅いですね :)」といわれた。修理にも、最大で6Mかかるし、やりたくない旨も聞かせていただいた。

しかし、そこで、大変貴重な情報ももらえた。「個人で DIY やられてる方がいますよ :)」

ここまで来たのだ、いくところまで行こう。そして、M14xR2は見事復活をはたした。この少し前から、GCCには興味があった。
Intel 12th 世代 CPU 対 Intel 3rd 世代 CPU 無論、前者の圧勝だ。なので、私なりのハンデを設けた。
Intel 12th GEN CPU は Java、Intel 3rd GEN CPU は C++ 。
俄然楽しくなってきた。

## 検証内容

RDBMS への接続、任意のレコードの取得までのラウンドトリップ時間。それを各マシンで計測、速い方が勝ち。

# Docker
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
## Ubuntu でも SqlDeveloper 使いたい
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

## M14xR2 戦いの舞台は整ったようだな
- C++ でMySQLに接続（spring_demo)。
- 任意のレコードの取得（person）。
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
.
.
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
## ホストと Docker 内の MySQL との共有ディレクトリ
この後に行うC++の接続のために必要だったこと。
```
sudo mkdir -p -v /tmp/ubuntu-mysql-2/data
```
## 共有ディレクトリを指定した docker run の実行
MySQL側の　/var/run/mysqld　が非常に重要。

ここの配下に作られる mysqlx.sock を C++ のソースファイルから Read する必要がある。
```
sudo docker run --name ubuntu-mysql-2 -e MYSQL_ROOT_PASSWORD=jack1234 -p 127.0.0.1:3306:3306 -v /tmp/ubuntu-mysql-2/data:/var/run/mysqld -d mysql:8.0.32-debian
```
## C++ で MySQL に接続できたよ
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
## M14xR2 Intel CPU 3rd GEN And C++ これが初めての計測結果だ。勝ったのか？ あってるのか？
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
#include "jdbc/mysql_driver.h"
#include "jdbc/mysql_connection.h"
#include "jdbc/mysql_error.h"
#include "jdbc/cppconn/statement.h"
#include "jdbc/cppconn/resultset.h"

#include "mysqlx/xdevapi.h"
#include "mysqlx/devapi/collection_crud.h"
#include <time.h>

using namespace std;

int main() {
    clock_t start = clock();
    mysqlx::Session sess("mysqlx://user:password@%2Ftmp%2Fubuntu-mysql-2%2Fdata%2Fmysqlx.sock");
    cout << "If you read this message, you did connected by C++. Huge Congraturations !!" << endl;

    mysqlx::Schema schema = sess.getSchema("spring_demo");
    string schemaName = schema.getName();
    cout << "your schema is " << schemaName << endl;
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
# g++ コンパイル
これで、ヘッダファイルの場所を指定しているのだから、ソース内でフルパス指定していると、問題あるんじゃないのかな。
（はい、もうやめました。Visual Studio Code にしかられたからさ、最初はね。）
```
 g++ -std=c++11 -I/usr/include/mysql-cppconn-8/ -L/usr/lib/x86_64-linux-gnu m14xr2-conn-mysql.cpp -lmysqlcppconn8 -o m14xr2-conn-mysql
```
