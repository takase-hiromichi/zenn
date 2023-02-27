---
title: "AWSのEC2でUbuntu 22.04 LTS を立てて、Zabbix Server 6.2をインストールする"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "Zabbix", "Ubuntu", "EC2"]
published: true
---

# インスタンスを立てる

EC2 > インスタンス > インスタンスを起動

![](/images/install_zabbix_for_ubuntu_22_lts/p1.png)

名前；zabbix

![](/images/install_zabbix_for_ubuntu_22_lts/p2.png)

マシンは Ubuntu

![](/images/install_zabbix_for_ubuntu_22_lts/p3.png)

インスタンスタイプは適当に t3a.small

![](/images/install_zabbix_for_ubuntu_22_lts/p4.png)

新しいキーペアを生成して...

![](/images/install_zabbix_for_ubuntu_22_lts/p5.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p6.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p7.png)

インスタンスを起動

![](/images/install_zabbix_for_ubuntu_22_lts/p8.png)

# インスタンスの接続

接続から

![](/images/install_zabbix_for_ubuntu_22_lts/p9.png)

SSH クライアント

![](/images/install_zabbix_for_ubuntu_22_lts/p10.png)

セキュリティグループはちゃんと絞っておくように

![](/images/install_zabbix_for_ubuntu_22_lts/p11.png)

鍵は~/.ssh/の中に移動

```
ls -la ~/.ssh/zabbix.pem
-rw-r--r--@ 1 takase  staff  1678  2 27 20:14 /Users/takase/.ssh/zabbix.pem
```

もろもろ実行

```
chmod 400 ~/.ssh/zabbix.pem
```

接続

```
ssh -i ~/.ssh/zabbix.pem ubuntu@3.114.6.227
```

入れた

```
ubuntu@ip-172-31-45-243:~$ pwd
/home/ubuntu
ubuntu@ip-172-31-45-243:~$
```

# Zabbix Server を入れる

https://www.zabbix.com/download?zabbix=6.2&os_distribution=ubuntu&os_version=22.04&components=server_frontend_agent&db=mysql&ws=apache

にならう

## zabbix repository のインストール

```
wget https://repo.zabbix.com/zabbix/6.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.2-4%2Bubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.2-4+ubuntu22.04_all.deb
sudo apt update
```

## zabbix server, frontend, agent のインストール

```
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

## MySQL を入れる

```
sudo apt update
sudo apt upgrade -y
sudo apt install mysql-server -y
sudo service mysql status
sudo mysql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'A@123abc#';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Abc445566@';
FLUSH PRIVILEGES;
```

## 初期データベース作成

```
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

## 初期スキーマとデータをインポート

```
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
mysql -uroot -p
set global log_bin_trust_function_creators = 0;
quit;
```

## zabbix server の設定ファイルを変更

```
sudo vi /etc/zabbix/zabbix_server.conf
```

```
DBPassword=password
```

## zabbix server と agent のプロセスを開始

```
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

## Web UI の確認

セキュリティグループで自分の IP だけ port 80 を開けておきます。

http://3.114.6.227/zabbix

![](/images/install_zabbix_for_ubuntu_22_lts/p12.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p13.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p14.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p15.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p16.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p17.png)
![](/images/install_zabbix_for_ubuntu_22_lts/p18.png)

Admin / zabbix

![](/images/install_zabbix_for_ubuntu_22_lts/p19.png)

Zabbix 6.2.7 が入った。

![](/images/install_zabbix_for_ubuntu_22_lts/p20.png)
