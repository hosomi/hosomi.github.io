## TomEE を CentOS7 にインストール

2016.09.27 記入

### TomEE とは

*  Tomcat + Java EE = TomEE  

** Java Enterprise Edition の機能を利用しない場合、通常は Tomcat で十分です。**

https://en.wikipedia.org/wiki/Apache_TomEE  
> Apache TomEE (pronounced "Tommy")  

トミーと呼ぶようです。


### インストール

super ユーザまたは root ユーザで作業してください。  
（適宜 sudo 等でも置き換えてください。）

#### 環境

記入時点全て最新を利用

| #    |  項目     |    内容               | 備考     |
| ---: |:----------| -------------------- | ---------|
| 1    | OS        | CentOS Linux release 7.2.1511 (Core) |  |
| 2    | Java      | jdk1.8.0_101 | tar.gz 版を利用 |
| 3    | TomEE     | 7.0.1 WebProfile (NOT JavaEE7 certified) | tar.gz 版を利用 |


#### ダウンロード

##### Java
最新版の JDK をダウンロードします。

1 http://www.oracle.com/technetwork/java/javase/downloads/index.html  
2 JDK(download) をクリック  
3 Accept License Agreement のラジオボタンを選択。  
4 Linux x64 の tar.gz を をクリック。

wget の場合：
```bash
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz
```

##### TomEE
最新版の TomEE webprofile をダウンロードします。

1 http://tomee.apache.org/downloads.html  
2 Version 7.0.1 の WebProfile (NOT JavaEE7 certified)  Format から tar.gz をクリック。

wget の場合：
```bash
wget http://repo.maven.apache.org/maven2/org/apache/tomee/apache-tomee/7.0.1/apache-tomee-7.0.1-webprofile.tar.gz
```

#### 解凍

それぞれ /opt/ 配下に解凍します。  

```bash
tar zxvf jdk-8u101-linux-x64.tar.gz
tar zxvf apache-tomee-7.0.1-webprofile.tar.gz
```

#### シンボリックリンク作成(optional)

バージョンアップをやりやすいようにしておく。

/opt/ 配下で作業。

```bash
ln -s jdk1.8.0_101 jdk-latest
ln -s apache-tomee-webprofile-7.0.1 tomee
```

#### TomEE 実行ユーザ作成

```bash
useradd tomee
passwd tomee  # Enter 後、パスワードを２回入力
```
** /home フォルダは利用しませんが、実行ユーザのみにする等のオプションはお好みで。**


#### 権限変更

/opt/ 配下で作業。  
シンボリックリンク自体とリンク先の両方の権限を変更。  

```bash
chown -h tomee. /opt/tomee
chown -R tomee. apache-tomee-webprofile-7.0.1
```

#### Systemd

Systemd から起動できるように各種設定ファイルの追加と起動確認を行う。

##### service ファイル追加

```bash
vim /etc/systemd/system/tomee.service
```

/etc/systemd/system/tomee.service:
```bash
[Unit]
Description=Apache TomEE
After=network.target

[Service]
User=tomee
Group=tomee
Type=oneshot
PIDFile=/opt/tomee/tomee.pid
RemainAfterExit=yes

EnvironmentFile=/etc/sysconfig/tomee
ExecStart=/opt/tomee/bin/startup.sh
ExecStop=/opt/tomee/bin/shutdown.sh
ExecReStart=/opt/tomee/bin/shutdown.sh;/opt/tomee/bin/startup.sh

[Install]
WantedBy=multi-user.target
```

##### 環境ファイル追加

```bash
vim /etc/sysconfig/tomee
```

/etc/sysconfig/tomee:
```bash
JAVA_HOME="/opt/jdk-latest/"
JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom"
```

##### service に権限付与

```bash
chmod 755 tomee.service
```

##### Systemd 起動確認

```bash
systemctl start tomee
```

エラーが出ていなければプロセスの確認  
```bash
ps -ef | grep tomee
```

出力例(tomee ユーザで実行されていれば OK です)：
```console
tomee     1516     1  3 10:07 ?        00:01:01 /opt/jdk-latest//bin/java -Djava.util.logging.config.file=/opt/tomee/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -javaagent:/opt/tomee/lib/openejb-javaagent.jar -Djava.security.egd=file:/dev/./urandom -Djdk.tls.ephemeralDHKeySize=2048 -classpath /opt/tomee/bin/bootstrap.jar:/opt/tomee/bin/tomcat-juli.jar -Dcatalina.base=/opt/tomee -Dcatalina.home=/opt/tomee -Djava.io.tmpdir=/opt/tomee/temp org.apache.catalina.startup.Bootstrap start
```

ブラウザで確認(Apache Tomcat (TomEE) バージョンが表示されていば問題なし ):  
http://ServerIP:8080/

その他のオプションも確認する:
```bash
systemctl status tomee
systemctl restart tomee
systemctl stop tome
```

自動起動を有効にする（optional）
```bash
systemctl enable tomee
```
