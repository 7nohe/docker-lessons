## Docker研修

## ToC



- 仮想環境
  - 仮想環境はなぜ必要か？
  - 仮想マシン(VirtualBox/VMware)
  - Vagrant
  - Docker
  - Docker vs Vagrant
- Dockerについて
  - Dockerのメリット
  - Dockerのデメリット
- まとめ1
- ハンズオン準備
- Containers
- Images
- Volumes
- Networks
- まとめ2
- コンテナオーケストレーションについて
- Docker Compose
- docker-compose.yml
- その他のオーケストレーションツール
  - Docker Swarm
  - Kubernetes (k8s)
- まとめ3



## 仮想環境はなぜ必要か？



- ローカル環境を汚さないするため
- 複数人で同じ環境化で開発するため
- 本番環境と同じ(または近い)環境で開発するため



## 仮想マシン (Virtual Machine/VM)

ホストOS型の仮想化技術。VirtualBox/VMwareなど。

ホストOS上にゲストOSを動かす。(例: ローカルのmacOS上にWindowsやLinuxのVMを立てる)



## Vagrant

VirtualBoxなどの仮想環境を設定ファイル(Vagrantfile)に基づいて管理/構築できるツール。

boxというVMイメージからVMを作成する。



## Dockerとは

コンテナ型の仮想化技術。

ゲストOSを使わずにDocker Engine上のコンテナでアプリケーションを動かす。

ホストOS型よりも軽量で、起動が早いのが特徴。

基本「１コンテナ=１関心事」が良い。

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/



## Docker vs Vagrant

|                  | Docker     | Vagrant              |
| ---------------- | ---------- | -------------------- |
| 役割             | 仮想環境   | 仮想マシン(VM)の管理 |
| プラットフォーム | Linux      | Windows/macOS/Linux  |
| 起動時間         | 数秒       | 数分                 |
| 環境の分離       | 一部       | 完全                 |
| 設定ファイル     | Dockerfile | Vagrantfile          |





## Dockerのメリット

- 非常に軽量
- 環境構築のコード化
- 冪等性
- ポータビリティ



## Dockerのデメリット

- ホストOSのような完全なふるまいはできない (例: ハードウェアエミュレート)
- 非Linux環境の動作はできない



## まとめ1



- 仮想化技術にはホストOS型とコンテナ型がある
- ホストOS型ではVagrantというツールでVMを管理できる
- コンテナ型の一つにDockerがある
- ホストOS型とコンテナ型にそれぞれ得意/不得意がある



------



## ハンズオン準備

```bash
$ git clone https://github.com/7nohe/docker-lessons.git
$ cd docker-lessons
```



## Containers



アプリケーションを動かすための箱(コンテナ)を作成します。

コンテナはイメージを元に作成できる。



よく使うコマンド　

- `docker container run [イメージ名]` : コンテナの起動
- `docker container stop [コンテナ名/ID]` : コンテナの停止
- `docker container rm [コンテナ名/ID]` : コンテナ削除
- `docker container ls` : コンテナ一覧
- `docker container exec [コンテナ名/ID] [コマンド]` : コンテナ上でコマンドの実行



リファレンス

https://docs.docker.com/engine/reference/commandline/container/





## Lesson1: まずはコンテナを動かしてみよう



nginxイメージを使って、静的ファイル配信用サーバーを立ててみます。



```
$ cd lesson0

$ docker run --name my-nginx -v $(pwd):/usr/share/nginx/html:ro -d -p 8000:80 nginx
```



オプションの解説

- `--name` : コンテナ名
- `-v` : ボリュームマウント(ローカル側: コンテナ側)。roはRead Only
- `-d` : バックグラウンドで実行
- `-p` : ポートマッピング: (ローカルポート: コンテナポート)



http://localhost:8000にアクセスしてみましょう。



## Docker Hub

Dockerレジストリサーバー。

ruby/postgres/nginxなど公式イメージが使える。

<https://hub.docker.com/>



Docker HubにあるDocker Registryイメージを使えばプライベートなレジストリサーバーを構築することも可能。



*野良イメージは使わないように注意しましょう。



## Challenge1-1: PostgreSQLのDBコンテナを作ってみよう



postgresイメージを使って以下のDBコンテナを作ってみよう

- ユーザー名: docker
- データベース名: dockerdb

*ヒント: runコマンドの際に環境変数(-eオプション)を指定してあげる*

参考: <https://hub.docker.com/_/postgres>



確認方法:

```bash
$ docker container exec -it my-postgres psql -U docker -d dockerdb 
dockerdb=# \du
dockerdb=# \l
```







## Challenge1-2: 初期データを投入してみよう



コンテナ起動時にDBに初期データを投入してみましょう。

- POSTテーブルをCREATE(id, title, bodyカラム)
- 適当な１レコードをINSERT



```sql
-- create-db.sql
CREATE TABLE POST (
  ID INT PRIMARY KEY NOT NULL,
  TITLE CHAR(50) NOT NULL,
  BODY TEXT NOT NULL
);

INSERT INTO POST (
  ID, TITLE, BODY
)
VALUES (
  1, 'Dockerの学習', 'コンテナを使って効率の良い開発を行えるよう学習しました。'
);
```



*ヒント: postgresコンテナの `/docker-entrypoint-initdb.d`  以下に`.sql.gz` 、`.sql` 、`.sh` ファイルを置くと起動時に実行されます。*

参考: <https://hub.docker.com/_/postgres>



確認方法:

```bash
$ docker container exec -it my-postgres psql -U docker -d dockerdb 
dockerdb=# SELECT * FROM POST;
```



## Images

コンテナを作るための元となるもの。

Dockerfileを定義することで、自分でカスタマイズしたイメージが作成できる。



よく使うコマンド

- `docker images build -t [イメージ名(:タグ名)] [Dockerfileのパス]` : イメージのビルド
- `docker image rm [イメージ名]` : イメージの削除
- `docker image ls` : イメージの一覧表示
- `docker image tag [ソースイメージ名(:タグ名)] [ターゲットイメージ名(:タグ名)]` : イメージのタグ付け
- `docker image push [イメージ名]` : レジストリへプッシュ



リファレンス

<https://docs.docker.com/engine/reference/commandline/image/>



## Lesson2: イメージを作ってみよう

Rubyイメージを元にコンテナ起動時にスクリプトが実行されるイメージを作成してみましょう。



まずはRuby動作確認用スクリプトを用意します。

```
# hello.rb
p 'Hello World'
```



次にイメージを作成するためのDockerfileを定義します。

```
# Dockerfile
FROM ruby:2.5

WORKDIR /usr/src/app

COPY hello.rb .

CMD ["ruby", "hello.rb"]

```



ビルド、イメージの確認、実行してみましょう。

```
$ docker image build -t hello_ruby:latest . 
$ docker image ls
$ docker container run hello_ruby:latest
"Hello World!"
```



## Lesson3: 作ったイメージをDocker Hubで共有しよう



1. Docker Hubアカウントからサインアップする。

[Sign up for Docker Hub](https://hub.docker.com/signup)



2. ターミナルでコマンドを打ってログインする

```bash
$ docker login
```





3. Docker Hubの[Create Repository] からレポジトリを作成する



4. ローカルのイメージをタグ付けする



```bash
$ docker image tag hello_ruby:latest [ユーザー名]/hello_ruby:[タグ名]
```



5. タグ付けしたイメージをプッシュする



```bash
$ docker image push [ユーザー名]/hello_ruby:[タグ名]
```



→プッシュ後は、 `docker image pull [ユーザー名]/hello_ruby:[タグ名]` でイメージをローカルに取得することができます。



[おまけ]

Docker Hubに公開されているイメージを探すコマンド

```bash
$ docker search [探したいイメージ名]
```



## Volumes

ボリュームは永続化したいデータをDockerで管理できるもの。

用途としては、

- バックアップ
- コンテナ間でのファイル共有

など。



よく使うコマンド

- `docker volume create [ボリューム名]` : ボリュームの作成
- `docekr volume ls` : ボリュームの一覧表示
- `docker volume rm [ボリューム名]` : ボリュームの削除



## Lesson4: ボリュームなしの場合でコンテナを削除してみよう

Challenge1-2で作成したコンテナ(ボリュームなし)を削除して、再度初期データを投入せずに作成してみましょう。

```
$ docker container rm my-postgres -f
$ docker container run --name my-postgres -e POSTGRES_USER=docker -e POSTGRES_DB=dockerdb -d postgres
$ docker container exec -it my-postgres psql -U docker -d dockerdb 
dockerdb=# SELECT * FROM POST; # 失敗する
```



コンテナにはデータが残っていないはずです。



## Lesson5: ボリュームを作成してデータを永続化してみよう

それではボリュームをつくってデータベース情報を永続化してみます。

```bash
$ docker volume create postgres-data
$ docker volume ls
$ cd challenge1-2
$ docker container run --name my-postgres \ 
	-v $(pwd)/:/docker-entrypoint-initdb.d \
	-v postgres-data:/var/lib/postgresql/data \
	-e POSTGRES_USER=docker -e POSTGRES_DB=dockerdb -d postgres
```



これで、ボリュームをマウントしたコンテナが起動しました。

試しにコンテナを削除して、(初期データなしで)再作成してみましょう。

```bash
$ docker container rm my-postgres -f
$ docker container run --name my-postgres \ 
	-v postgres-data:/var/lib/postgresql/data \
	-e POSTGRES_USER=docker -e POSTGRES_DB=dockerdb -d postgres
$ docker container exec -it my-postgres psql -U docker -d dockerdb 
dockerdb=# SELECT * FROM POST; # 成功！
```



これでデータの永続化することができました。



## ボリュームマウントとバインドマウント







## Networks

コンテナ間で通信を行うためのもの。

例として、

`アプリケーションコンテナ(Rails) + DBコンテナ(Postgres)`

の構成だった場合、RailsからPostgresへ接続して読み書きしなければなりません。

そこでコンテナ間で通信できるネットワークを用意してあげる必要があります。



よく使うコマンド

- `docker network create [ネットワーク名]` : ネットワーク作成
- `docker network connect [ネットワーク名] [コンテナ名]` : コンテナをネットワークへ接続



## Lesson6: コンテナ間で通信しよう



まずはネットワークを作ります。

```bash
$ docker network create busybox-network
$ docker network ls
```



次に作成したネットワークに接続するコンテナ二つ用意します。



```bash
$ docker run --name busybox1 --network busybox-network -it --rm busybox
# 別タブで実行
$ docker run --name busybox2 --network busybox-network -it --rm busybox
```



```
# (busybox1)
$ ping busybox2

# (busybox2)
$ ping busybox1
```



通信が成功するはずです。



ネットワーク確認

```bash
$ docker network inspect busybox-network
```



## [補足]ネットワークドライバの種類



Dockerのネットワークドライバは複数あります。



- `bridge` : デフォルトドライバ。今回使用したものです。スタンドアローンでDockerを使う場合はこれ。
- `host` : Docker Swarmでのみ使える。
- `overlay` : 複数のDockerホストでの通信で使います。Swarmなどで使う。
- `macvlan` : MACアドレスをアサインして、物理デバイスに接続できるようにする。
- `none` : 全てのネットワークを無効化する。





## まとめ２



- コンテナはイメージを元に作られる
- イメージはDockerfileからビルドして作られる
- ボリュームを使ってデータを永続化できる
- ネットワークを使ってコンテナ間で通信できる



## コンテナオーケストレーションについて



実際のWebアプリケーションでは、

複数コンテナ(Rails + Postgres + Redis + nginxなど)で運用する場合がほとんどです。

そういった複数コンテナの管理に役立つのがコンテナオーケストレーションツールです。



## Docker Compose

コンテナオーケストレーションツールの一つで、単一Dockerホストでの複数コンテナを扱う際に便利です。



よく使うコマンド



- `docker-compose up -d` : docker-composeで定義されたコンテナの作成/バックグラウンド実行
- `docker-compose ps` : docker-composeで定義されたコンテナの一覧表示



## docker-compose.yml

Docker Composeで扱うためのコンテナ定義ファイル。

使うイメージ、ポート、ボリュームマウント、ネットワークなどの設定ができる。



```
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```



- `build` : コンテナ作成のためのDockerfileのパス 
- `ports` : ポートフォワーディング
- `volumes` : ボリュームマウント、バインドマウント
- `image` : コンテナ作成のためのイメージ名



## Lesson7: Rails開発環境を作ってみよう

<https://github.com/7nohe/rails-docker-example>



## そのほかのオーケストレーションツール



- Docker Swarm:
- Kubernetes (k8s): 





## まとめ３



