# docker-study3

## Java（Springboot）を使った開発環境

このサンプルは Java(Springboot) を起動させるためのサンプルです。

### Dockerfile

<pre>
# Javaの公式イメージをDockerHubからダウンロード
FROM openjdk:21-slim-bullseye

ENV JAVA_HOME /usr/local/openjdk-21
</pre>

Java を使う場合、環境変数の JAVA_HOME を設定しておきましょう。

### docker-compose.yml

<pre>
version: '3.8'

# データベースのデータを保存しておくたのめのvolume定義
volumes:
  springboot-app-data:

services:
  app:
    container_name: springboot-app
    build: . # Dockerfileのパスを指定
    # ttyとstdin_openはコンテナを起動させておくための設定
    tty: true
    stdin_open: true
    volumes:
      - ..:/workspaces:cached
    environment:
      # NOTE: POSTGRES_DB/USER/PASSWORD should match values in db container
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
      POSTGRES_HOSTNAME: springboot-db

  # データベースはPostgreSQLを私用
  db:
    container_name: springboot-db
    image: postgres:latest
    # データベースのデータを保存しておくたのめのvolume定義
    volumes:
      - springboot-app-data:/var/lib/postgresql/data
    environment:
      # NOTE: POSTGRES_DB/USER/PASSWORD should match values in app container
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
      TZ: 'Asia/Tokyo'
    # 外部ツールからメンテナンスするために接続用ポート番号を指定
    ports:
      - 5432:5432
</pre>

アプリケーションのコンテナ（springboot-app）とデータベースのコンテナ（springboot-db）を定義しています。
データベースの接続

### devcontainer.js

<pre>
{
  "name": "Java(SpringBoot) Development Environment",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces",
  "customizations": {
    "vscode": {
      "extensions": [
        "vscjava.vscode-spring-initializr",
        "vscjava.vscode-java-pack"
      ]
    }
  }
}
</pre>

SpringBoot で開発を行うために VSCode の拡張機能"spring-initializr"をインストールしておきます。

### install springboot

表示 → コマンドパレット → 開発コンテナー:コンテナーで再度開く  
コンテナを起動させたらコマンドパレットから「Spring Initalizr: Create Maven Project...」を選択  
・Spring Boot のバージョンを選択（3.4）  
・プロジェクトの言語を選択（Java）  
・プロジェクトのパッケージ名を入力（com.example）  
・パッケージタイプを選択（jar）  
・Java のバージョンを選択（21）  
・依存ライブラリを選択（Spring Web）  
※その他 Lombok、Thymeleaf など必要なライブラリを選択してください  
・プロジェクトを作成するフォルダを選択（デフォルトのままで良い）

### 既にプロジェクトのファイルがインストールされている場合

git clone でソースコードをダウンロードしたら
VSCode の MAVEN メニューからプロジェクトを右クリックして「install」を選択

### SpringBoot のデバッグ実行

VScode の実行とデバッグから「launch.json ファイルを作成します」をクリックして、  
「Java」を選択する  
作成された launch.json を下記の様に書き換える

<pre>
{
  // IntelliSense を使用して利用可能な属性を学べます。
  // 既存の属性の説明をホバーして表示します。
  // 詳細情報は次を確認してください: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "DemoApplication",
      "request": "launch",
      "mainClass": "com.example.demo.DemoApplication",
      "projectName": "demo"
    }
  ]
}</pre>
