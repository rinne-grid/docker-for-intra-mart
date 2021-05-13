
## docker for intra-mart

* intra-martの検証環境をDockerコンテナ上に作成し、環境構築にかかる時間を削減

### バージョンについて

* AP（Resin）のコンテナについては、Gitタグで管理しています。
* 利用したいバージョンのタグ指定の上cloneを実施してください。

```sh
$ git clone https://github.com/rinne-grid/docker-for-intra-mart im -b <tag_name> 
```

|tag|OS|Javaバージョン|確認したResinバージョン|確認したiAPバージョン|
|---|---|-----------|----|----|
|v1.0-openjdk8|centos:7.5.1804(※1)|OpenJDK8|4.0.56より前のバージョンで確認|iAP 2019 Summerで確認|
|v1.1-openjdk11|centos:7.5.1804(※1)|OpenJDK11|4.0.64で確認|iAP 2020 Summerで確認|
|v2.0-openjdk11-rhel7.x|RHEL7.x|OpenJDK11|4.0.64で確認|iAP 2020 Summerで確認|
|v2.0-openjdk11-rhel8.x|RHEL8.x(※1)|OpenJDK11|4.0.65で確認|iAP 2020 Summerで確認(※1)|

（※1 iAP2019Summer, iAP2020Summerのサーバ要件は、RHEL6.x, RHEL7.xが前提となります。

http://accel-archives.intra-mart.jp/2020-summer/document/iap/public/iap_release_note/texts/support_environment/index.html

本リポジトリではRHEL8.xとiAP 2020 Summerの組み合わせで確認していますが、公式による動作保証がされているわけではありませんのでご注意ください。）

### Dockerで作成するintra-martのシステム構成

#### Docker上の環境


|ホスト名|コンテナイメージ|目的|
|--------|------------------------------|-----------|
|ap|RHEL7.x(RedHat Universal Base Image7 ubi7/ubi:latest)(※2)|resin-proの実行及びwarデプロイ|
|db|PostgreSQL 10(library/postgres)|intra-martに関するデータの保存|
|adminer|Adminer(library/adminer)|dbのデータ参照用のアプリ|


（※2 本Docker関連ファイルを利用する場合は、あくまでも動作検証用の環境に留めておくことをおすすめします。本リポジトリ内容の利用によって発生した障害等について、一切責任を負いません）



### 必要なPC環境

* 下記のいずれか

|OS|バージョン|Docker|
|--|----------|------|
|Windows7以降|64bit|Docker Toolbox|
|Windows10|64bit|Docker for Windows|
|macOS|-|Docker for Mac|


### 事象別のコマンドリファレンス

|事象|コマンド|
|----|--------|
|Dockerコンテナを開始したい|docker-compose up|
|Dockerコンテナをビルドしたい|docker-compose build --no-cache|
|Dockerコンテナを終了させたい|docker-compose down|
|DBデータやストレージを削除して、<br>新しくテナント環境セットアップから始めたい（永続化しているコンテナのがデータ全部消えるので要注意）|docker-compose up <br>docker-compose down -v<br>docker-compose up|
|warファイルをアンデプロイしたい|docker exec im_ap /ap-server/bin/resinctl undeploy imart|
|warファイルをデプロイしたい|docker exec im_ap /ap-server/bin/resinctl deploy /war/imart.war|


### コンテナごとの接続情報

* APサーバー（CentOS）

|コンテナ名|ホスト名|ポート番号(ホスト)|ポート番号(コンテナ)|
|----------|--------|----------|------|
|im_ap|ap|8888|8080|

* PostgreSQL

|コンテナ名|ホスト名|DB名|ユーザ名|パスワード|ポート番号(ホスト)|ポート番号(コンテナ)|
|----------|--------|----|--------|----------|----------|----|
|im_db|db|imart|imart|imart|5432|5432|

* Adminer

|コンテナ名|ホスト名|ポート番号(ホスト)|ポート番号(ゲスト)|
|----------|--------|----------|----|
|im_ap|adminer|8889|8080|





### 利用手順

本Dockerプロジェクトを利用して、intra-martの環境を構築する手順を記述いたします。
（この手順は、Windows10 + Docker Toolboxを前提にしています。必要に応じて適宜読み替えをお願いします。）

#### [1] Docker Toolboxのインストール

* 下記のURLより、Docker Toolboxをダウンロードし、インストールします

https://docs.docker.com/toolbox/toolbox_install_windows/


#### [2] Dockerのメモリを増やす

* Dockerで利用するメモリを確保するため、VirtualBoxから、メモリを増やしておきます。
* 下記のURLの記事にメモリを増やす方法が詳しく記載されています  
https://qiita.com/niisan-tokyo/items/2d7d21aeb4e25f7a7bbe




* Docker for Windowsの場合は、下記のURLが参考になります

https://qiita.com/fkooo/items/d2fddef9091b906675ca



> タスクトレイのDockerアイコン右クリック->Settingsを開く



* Docker for Macの場合の場合、下記のURLが参考になります

> Preferences → Advanced にあるMemoryで使用量を調節

https://qiita.com/mks1412/items/9356187a3bcb20b64e82




#### [2] Gitのインストール

* 下記のURLより、Git for Windowsをダウンロードし、インストールします

https://gitforwindows.org/



#### [3] Dockerプロジェクトのダウンロードと初期設定

* 任意のフォルダで、以下のコマンドを実行し、dockerプロジェクトをダウンロードします

```sh
> git clone https://github.com/rinne-grid/docker-for-intra-mart im
> cd im
```

* warファイルの配置用フォルダを作成します

```sh
> mkdir .\ap\war
```

#### [4] Jugglingでwarファイルを作成

プロジェクト名をimartにして、必要なモジュールを選択し、設定を行います。
今回のDocker環境をそのまま利用するためには、下記ファイルの設定を変更する必要があります

* storage-config.xmlを設定する
* resin-web.xmlを設定する
* 出力するwarファイル名をimart.warとする

（Dockerプロジェクトのap/.envファイルを変更することで、別の値を指定することも可能です）

##### storage-config.xmlの設定

imart/config/storage-config.xml の 19行目付近を以下のとおりに変更します

```xml
<root-path-name>/im-data/storage</root-path-name>
```

##### resin-web.xmlの設定

imart/resin-web.xml 内容を下記のとおりにします

```xml
<web-app xmlns="http://caucho.com/ns/resin" xmlns:resin="urn:java:com.caucho.resin">
    <character-encoding>UTF-8</character-encoding>

    <log-handler name="" class="jp.co.intra_mart.common.platform.log.handler.JDKLoggingOverIntramartLoggerHandler"/>
    <logger name="debug.com.sun.portal" level="warning" />

    <!-- im_service(im_asynchronous) -->
    <resource jndi-name="jca/work" type="jp.co.intra_mart.system.asynchronous.impl.executor.work.resin.ResinResourceAdapter" />
    <jsp>
        <recycle-tags>false</recycle-tags>
    </jsp>
    <database jndi-name="jdbc/default">
        <driver>
            <type>org.postgresql.Driver</type>
            <url>jdbc:postgresql://db:5432/imart</url>
            <user>imart</user>
            <password>imart</password>
            <init-param>
                <param-name>preparedStatementCacheQueries</param-name>
                <param-value>0</param-value>
            </init-param>
        </driver>
        <max-connections>20</max-connections>
        <prepared-statement-cache-size>8</prepared-statement-cache-size>
    </database>
    <session-config>
        <reuse-session-id>false</reuse-session-id>
        <session-timeout>30</session-timeout>
    </session-config>

    <mime-mapping extension=".json" mime-type="application/json"/>
</web-app>
```


##### warファイルの出力

imart.warという名称でwarファイルを出力したら、
プロジェクトのim/ap/warフォルダの中に、warファイルをコピーします



#### [5] intra-martのサイトからLinuxのresin-proをダウンロード

intr-martのサイトにアクセスし、プロダクトファイルダウンロードボタンを押下します。

https://www.intra-mart.jp/download/library/

ライセンスキーを入力すると、ダウンロード可能なファイル一覧が表示されます。

なお、intra-martサイトにも書いているとおり、.tar.gzがLinux用のresin-proになります。

https://www.intra-mart.jp/download/product/iap/setup/iap_setup_guide/texts/install/linux/resin_linux.html

最新のResin<b>resin-pro-4.0.xx.tar.gz</b>を入手します。


#### [6] 7zipをダウンロード、インストール

tar.gz形式のファイルを展開するため、この記事では7zipを利用します。

https://sevenzip.osdn.jp/


1. resin-pro.4.0.xx.tar.gzを展開します
2. resin-pro.4.0.xx.tarファイルが作成されます
3. resin-pro.4.0.xx.tarを展開します
4. resin-pro.4.0.xxフォルダが作成されます
5. resin-pro.4.0.xxフォルダの直下に、automake, binといったフォルダが存在することを確認します

![resin-proフォルダ](http://www.rinsymbol.sakura.ne.jp/github_images/docker/blog.PNG)

#### [7] Dockerプロジェクトのフォルダにresin-proをコピー

* 上記の[6]の5のフォルダ「resin-pro.4.0.xx」の名称をresin-proに変更します
* resin-proフォルダをim/apフォルダにコピーします

#### [8] プロジェクトのフォルダ構成の確認

* フォルダを確認し、以下の構成と同じになっていることを確認します
* ポイント
  * im/ap/resin-proフォルダがあり、直下にautomake等のファイルが存在する
  * im/ap/warフォルダがあり、imart.warファイルが存在する


```
im
│  .env
│  .gitignore
│  docker-compose.yml
│  README.md
│
└─ap
    │  Dockerfile
    │
    ├─resin-pro
    │  ├─automake  など
    │
    └─war
            imart.war
```

#### [9] 必要に応じて、設定ファイルを変更する

* im/ap/resin-pro/conf/resin.properties の 82行目付近 - jvm_args

-Xmx, -Xmsの値が、初期状態だと8192m(8GB)が設定されているため、自分のPCのメモリ状況に合わせて変更します

```ini
jvm_args : -Dfile.encoding=UTF-8 -Djava.io.tmpdir=tmp -Xmx1500m -Xms1500m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=30 -XX:NewSize=512m -XX:MaxNewSize=512m -XX:+CMSClassUnloadingEnabled -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+HeapDumpOnOutOfMemoryError -Xloggc:log/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=10M
```

* HTTPプロキシの設定 im/.env

社内ネットワーク等で、プロキシサーバーを経由する必要がある場合、.envのHTTP_PROXY、HTTPS_PROXYに値を設定します

```env
HTTP_PROXY=http://user:password@server:port/
HTTPS_PROXY=http://user:password@server:port/
```

#### [10] Docker Machineを起動

* Docker Machineを起動していない場合、下記コマンドで起動します
  * Docker Toolbeltという青いアイコンのショートカットを起動すると、自動でDocker Machineを起動してくれます

![docker-toolbelt](http://www.rinsymbol.sakura.ne.jp/github_images/docker/docker-toolbelt.PNG)

```sh
> docker-machine start
```


#### [11] Dockerコンテナの起動

* プロジェクトフォルダに移動します

```sh
> cd any_folder\im
```

* docker-composeを利用し、コンテナを起動します

```sh
> docker-compose build --no-cache
> docker-compose up -d
```

ネットワーク接続環境にもよりますが、早くて5分、おそくとも30分程度で完了します

* resinのindexページに接続します

http://192.168.99.100:8888

![docker-toolbelt](http://www.rinsymbol.sakura.ne.jp/github_images/docker/resin-top.PNG)

（Docker for WindowsやDocker for Macの場合は、上記IPアドレスではなく  http://localhost:8888  もしくは自分で設定しているホスト名やIPアドレスにアクセスします。）

* resinのページが開けることが確認できたら、warファイルをデプロイします

```sh
> docker exec im_ap /ap-server/bin/resinctl deploy /war/imart.war
```


コンテナ内のresin-proの場所は、/ap-serverです。
im/.envファイル内の変数を変更することで、お好きな場所を指定できます。

* デプロイのコマンドが終了して、数分経ったらintra-martのセットアップページにアクセスします
  * 503 Service Temporarily Unavailableが発生する場合は、もう少しだけ待ってあげてください。

http://192.168.99.100:8888/imart/system/login



* 無事にテナント設定画面が表示されるので、テナント環境セットアップを実行します

![tenant1](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant.PNG)


* テナントIDはimartを指定します

![tenant2](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant2.PNG)
* リソース参照名は一覧に表示されたものを選択します

![tenant3](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant3.PNG)

* テナント登録を行い、しばらく待ちます

![tenant4](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant4.PNG)

* テナント環境セットアップが適切に動作しているかどうかについては、Adminerからテーブル作成状況を参照することで確認できます
  * http://192.168.99.100:8889にアクセスします
  * Adminerが表示されるので、下記のとおり情報を入力します

|情報名|入力情報|
|----|--------|
|データベース情報|PostgreSQL|
|サーバ|db|
|ユーザ名|imart|
|パスワード|imart|
|データベース|imart|

  * テーブルの作成状況が確認できます(だいたい500テーブルくらいができたら、処理完了です)

  ![adminer2](http://www.rinsymbol.sakura.ne.jp/github_images/docker/adminer12.PNG)

  ![tenant5](http://www.rinsymbol.sakura.ne.jp/github_images/docker/tenant5.PNG)


* データベースやストレージ情報はDocker Volumeに保存しているため、データは永続化されています
  * 一度、docker-compose downで終了し、もう一度docker-compose upを試して、システムログイン画面にアクセスすると、ダッシュボードが表示されることがわかります

![dashboard](http://www.rinsymbol.sakura.ne.jp/github_images/docker/dashboard.PNG)

