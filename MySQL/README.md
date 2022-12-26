# MySQLの導入  
```bash
 $ sudo apt install mysql-server
```

## パスワードポリシーの確認
現在のパスワードポリシーの確認  
```sql
mysql> show variables like 'validate_password%';
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 6     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | LOW   |
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
7 rows in set (0.00 sec)
```
パスワードポリシーの変更  
```sql
mysql> set global validate_password.policy=0; # ポリシーレベルをLOWに変更
mysql> set global validate_password.length=6; # パスワードの必要文字数を6文字に変更
```

## ユーザの作成
ユーザ「root」でログイン  
```sql
 $ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 8.0.21-0ubuntu0.20.04.4 (Ubuntu)
<snip>
```

rootユーザのパスワード変更
```sql
mysql> set password for 'root'@'localhost' = '********';
```

USEコマンドでmysqlを指定  
```sql
mysql> USE mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> CREATE DATABASE test;
Query OK, 1 row affected (0.00 sec)
```

ユーザ「ubuntu」の作成  
IDENTIFIED BY 'ubuntu'の「ubuntu」がパスワードとなる  
```sql
mysql> CREATE USER 'ubuntu'@'localhost' IDENTIFIED BY 'ubuntu';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON test.* TO 'ubuntu'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT user,plugin FROM user;
+------------------+-----------------------+
| user             | plugin                |
+------------------+-----------------------+
| debian-sys-maint | caching_sha2_password |
| mysql.infoschema | caching_sha2_password |
| mysql.session    | caching_sha2_password |
| mysql.sys        | caching_sha2_password |
| root             | mysql_native_password |
| ubuntu           | caching_sha2_password |
+------------------+-----------------------+
6 rows in set (0.00 sec)

mysql> quit
Bye
```
## MySQLを使ったテーブルの作成
ユーザ「ubuntu」でログイン
```sql
 $ mysql -u ubuntu -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
<snip>
```

CREATE TABLEを用いたテーブルの作成  
```sql
CREATE TABLE テーブル名 (列名１ 型, 列名２ 型, 列名３ 型 );
```

実行例  
```sql
mysql> USE test;
Database changed
mysql> CREATE TABLE table1 (time DATETIME, value CHAR(10), number INTEGER );
Query OK, 0 rows affected (0.01 sec)
```

作成されたテーブルの確認  
```sql
mysql> DESCRIBE table1;
+--------+----------+------+-----+---------+-------+
| Field  | Type     | Null | Key | Default | Extra |
+--------+----------+------+-----+---------+-------+
| time   | datetime | YES  |     | NULL    |       |
| value  | char(10) | YES  |     | NULL    |       |
| number | int      | YES  |     | NULL    |       |
+--------+----------+------+-----+---------+-------+
3 rows in set (0.01 sec)
```

テーブルに値の挿入
```
mysql> INSERT INTO table1 VALUE ('2020-09-25 00:51:20','test-char',777);
Query OK, 1 row affected (0.02 sec)
```

テーブルに挿入した値の確認
```sql
mysql> SELECT * FROM table1;
+---------------------+-----------+--------+
| time                | value     | number |
+---------------------+-----------+--------+
| 2020-09-25 00:51:20 | test-char |    777 |
+---------------------+-----------+--------+
1 row in set (0.00 sec)
```

## Javaによるデータベース接続

DataManagerを使ったデータベース接続確認用プログラム
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectDatabase {
	public static void main(String[] args) {
		String path = "jdbc:mysql://localhost/";
		String databaseName = "データベース名";
		String jdbcUrl = path + databaseName;
		String userId = "root";
		String password = "パスワード";

		Connection con = null;
		try {
			con = DriverManager.getConnection(jdbcUrl, userId, password);
			System.out.println("接続");

		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				if (con != null) {
					con.close();
					System.out.println("切断");
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}
```

## MySQL Workbench の導入
[参考](https://codepre.com/ja/como-instalar-mysql-workbench-en-ubuntu-20-04-lts.html)  

とりあえずアップデート＆グレード
```bash 
~$ sudo apt update
~$ sudo apt upgrade
```

MySQLリポジトリを追加
```bash 
~$ wget https://dev.mysql.com/get/mysql-apt-config_0.8.15-1_all.deb
~$ sudo apt install ./mysql-apt-config_0.8.15-1_all.deb
```

アップデートすると失敗する(パブリックキーが期限切れのため更新できない)  
```
~$ sudo apt update
ヒット:12 http://jp.archive.ubuntu.com/ubuntu focal-backports InRelease
パッケージリストを読み込んでいます... 完了
W: GPG エラー: http://repo.mysql.com/apt/ubuntu focal InRelease: 公開鍵を利用できないため、以下の署名は検証できませんでした: NO_PUBKEY 467B942D3A79BD29
E: リポジトリ http://repo.mysql.com/apt/ubuntu focal InRelease は署名されていません。
N: このようなリポジトリから更新を安全に行うことができないので、デフォルトでは更新が無効になっています。
N: リポジトリの作成とユーザ設定の詳細は、apt-secure(8) man ページを参照してください。
```
**「上のNO_PUBKEY 467B942D3A79BD29」**を覚えておく  
apt-key-listでMySQLのパブリックキーを確認すると「期限切れ」とある  
```
~$ apt-key list
dsa1024 2003-02-03 [SCA] [期限切れ: 2022-02-16]
      A4A9 4068 76FC BD3C 4567  70C8 8C71 8D3B 5072 E1F5
uid           [期限切れ] MySQL Release Engineering <mysql-build@oss.oracle.com>
```
上で覚えた467B942D3A79BD29を使ってパブリックキーをアップデート（apt-key listで確認すると「期限切れ」から「不明」に変わる）
```
~$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6A030B21BA07F4FB
Executing: /tmp/apt-key-gpghome.iqv8eozhE8/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29
gpg: 鍵467B942D3A79BD29: 公開鍵"MySQL Release Engineering <mysql-build@oss.oracle.com>"をインポートしました
gpg: 処理数の合計: 1
gpg:               インポート: 1
```
再度アップデート
```
~$ sudo apt update
取得:1 file:/var/cuda-repo-ubuntu2004-11-6-local  InRelease
無視:1 file:/var/cuda-repo-ubuntu2004-11-6-local  InRelease
取得:2 file:/var/cuda-repo-ubuntu2004-11-6-local  Release [564 B]
取得:2 file:/var/cuda-repo-ubuntu2004-11-6-local  Release [564 B]
取得:3 http://repo.mysql.com/apt/ubuntu focal InRelease [12.9 kB]
ヒット:4 http://archive.ubuntulinux.jp/ubuntu focal InRelease                  
ヒット:5 http://archive.ubuntulinux.jp/ubuntu-ja-non-free focal InRelease      
ヒット:7 https://dl.google.com/linux/chrome/deb stable InRelease               
ヒット:8 http://packages.microsoft.com/repos/code stable InRelease             
取得:9 http://repo.mysql.com/apt/ubuntu focal/mysql-8.0 Sources [963 B]        
ヒット:10 http://security.ubuntu.com/ubuntu focal-security InRelease           
ヒット:11 http://jp.archive.ubuntu.com/ubuntu focal InRelease 
ヒット:12 http://jp.archive.ubuntu.com/ubuntu focal-updates InRelease
取得:13 http://repo.mysql.com/apt/ubuntu focal/mysql-apt-config i386 Packages [567 B]
ヒット:14 http://jp.archive.ubuntu.com/ubuntu focal-backports InRelease        
取得:15 http://repo.mysql.com/apt/ubuntu focal/mysql-apt-config amd64 Packages [567 B]
取得:16 http://repo.mysql.com/apt/ubuntu focal/mysql-8.0 amd64 Packages [8,524 B]
取得:17 http://repo.mysql.com/apt/ubuntu focal/mysql-tools i386 Packages [455 B]
取得:18 http://repo.mysql.com/apt/ubuntu focal/mysql-tools amd64 Packages [6,126 B]
30.1 kB を 3秒 で取得しました (9,613 B/s)
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています       
状態情報を読み取っています... 完了
アップグレードできるパッケージが 4 個あります。表示するには 'apt list --upgradable' を実行してください。
```
aptでインストールしようと試みると怒られる  
```
~$ sudo apt install mysql-workbench-community
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています                
状態情報を読み取っています... 完了

No apt package "mysql-workbench-community", but there is a snap with that name.
Try "snap install mysql-workbench-community"

E: パッケージ mysql-workbench-community が見つかりません

```
代わりに「snap install mysql-workbench-community」しろと言われるのでやると入る  
```
~$ snap install mysql-workbench-community
mysql-workbench-community 8.0.29 from Tonin Bolzan (tonybolzan) installed
```
アプリ検索結果にworkbenchがあればOK

## OKじゃなかった（Workbenchにアクセスできない問題の解決方法）
Workbenchにアクセスしようとすると、「Cannot Connect to Database Server」やらなんやらで怒られる  
- 「Ubuntu Softwareセンター」を開く  
- 「mysql-workbench-community」を検索して開く（インストール済みとなっているはず）  
- 「Permissions」を選択して全部許可  
- Workbenchが開けばOK

テスト用SQL  
SQLを全選択しQueryの「Execute」で実行可能
```sql
CREATE TABLE `スキーマ名`.`user` (
  `id` BIGINT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(100) NOT NULL,
  `address` VARCHAR(255) NULL,
  `phone` VARCHAR(50) NULL,
  `update_date` DATETIME NOT NULL,
  `create_date` DATETIME NOT NULL,
  `delete_date` DATETIME NULL,
  PRIMARY KEY (`id`));

INSERT INTO `スキーマ名`.`user` (`id`, `name`, `address`, `phone`, `update_date`, `create_date`) 
VALUES ('1', 'テスト太郎', '東京都サンプル区1-1', '080-0000-0000', '2019-05-06 12:00:00', '2019-05-01 12:00:00');
```
