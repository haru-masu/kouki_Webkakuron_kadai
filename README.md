# AWS EC2 + Docker を用いた Web サービス構築手順書

本ドキュメントは、Web概論後期課題として作成した  
AWS EC2 上に Docker 環境を構築し、PHP を用いた Web アプリケーションを動作させるまでの手順をまとめたものである。

---

## 使用技術

- AWS EC2（Amazon Linux）
- Docker
- Docker Compose
- Nginx
- PHP 8.4（php-fpm）
- MySQL 8.4
- Redis
- PowerShell（Windows）

---

## EC2 初期設定

### キーペア（秘密鍵）の権限設定

1. 秘密鍵ファイルを右クリックし「プロパティ」を開く  
2. 「セキュリティ」タブを選択  
3. 「継承の無効化」をクリック  
4.  
   継承されたアクセス許可をこのオブジェクトの明示的なアクセス許可に変換します  
   を選択  
5. ktc 以外のプリンシパルを削除し、適用する  

---

## EC2 へ SSH 接続

ssh ec2-user@IPアドレス -i 秘密鍵ファイルのパス

---

## 基本ツールのインストール

sudo yum install vim -y  
sudo yum install screen -y  

screen

---

## Docker のインストール

sudo yum install -y docker  
sudo systemctl start docker  
sudo systemctl enable docker  

sudo usermod -a -G docker ec2-user  

exit  

※ 再ログイン後、再度 screen を実行する

---

## Docker Compose のインストール

sudo mkdir -p /usr/local/lib/docker/cli-plugins/  

sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose  

sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose  

docker compose version  

---

## 作業ディレクトリ作成

mkdir dockertest  
cd dockertest  

sudo chown -R ec2-user:ec2-user ~/dockertest  

---

## PHP 設定（php.ini）

post_max_size = 5M  
upload_max_filesize = 5M  

session.save_handler = redis  
session.save_path = "tcp://redis:6379"  
session.gc_maxlifetime = 86400  

---

## Dockerfile（PHP）

FROM php:8.4-fpm-alpine AS php  

RUN apk add --no-cache autoconf build-base \  
    && yes '' | pecl install redis \  
    && docker-php-ext-enable redis  

RUN docker-php-ext-install pdo_mysql  

RUN install -o www-data -g www-data -d /var/www/upload/image/  

COPY ./php.ini ${PHP_INI_DIR}/php.ini  

---

## Docker Compose 設定（compose.yml）

services:  
  web:  
    image: nginx:latest  
    ports:  
      - 80:80  
    volumes:  
      - ./nginx/conf.d/:/etc/nginx/conf.d/  
      - ./public/:/var/www/public/  
      - image:/var/www/upload/image/  
    depends_on:  
      - php  

  php:  
    container_name: php  
    build:  
      context: .  
      target: php  
    volumes:  
      - ./public/:/var/www/public/  
      - image:/var/www/upload/image/  

  mysql:  
    container_name: mysql  
    image: mysql:8.4  
    environment:  
      MYSQL_DATABASE: example_db  
      MYSQL_ALLOW_EMPTY_PASSWORD: 1  
      TZ: Asia/Tokyo  
    volumes:  
      - mysql:/var/lib/mysql  
    command: >  
      mysqld  
      --character-set-server=utf8mb4  
      --collation-server=utf8mb4_unicode_ci  
      --max_allowed_packet=4MB  

  redis:  
    container_name: redis  
    image: redis:latest  
    ports:  
      - 6379:6379  

volumes:  
  mysql:  
  image:  

---

## コンテナ起動

docker compose up -d --build  

---

## Nginx 設定

mkdir -p nginx/conf.d  
sudo chown -R ec2-user:ec2-user nginx  

server 設定ファイル（default.conf）を作成する。

---

## public ディレクトリ準備

mkdir public  

sudo chown -R ec2-user:ec2-user /home/ec2-user/dockertest/public  

### ※sshでEC2インスタンスに入らず、powershell上で行ってください。
#### githubにあるリポジトリをzipで圧縮し、解凍する。

### PowerShell からファイル転送

```bash
scp -i {秘密鍵のファイルパス} -r {publicディレクトリのファイルパス} ec2-user@{IPアドレス}:/home/ec2-user/dockertest
```


## データベース作成

docker compose exec mysql mysql -u root example_db  

USE example_db;  

＄docker compose exec mysql mysql -u root example_db
＄USE example_db;
＄CREATE TABLE `access_logs` (
  `id` INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
  `user_agent` TEXT NOT NULL,
  `remote_ip` TEXT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);

＄CREATE TABLE `bbs_entries` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT UNSIGNED NOT NULL,
  `body` TEXT NOT NULL,
  `image_filename` TEXT DEFAULT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);

＄CREATE TABLE `user_relationships` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `followee_user_id` INT UNSIGNED NOT NULL,
  `follower_user_id` INT UNSIGNED NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);

＄CREATE TABLE `users` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `name` TEXT NOT NULL,
  `email` TEXT NOT NULL,
  `password` TEXT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE `users` ADD COLUMN icon_filename TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN introduction TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN cover_filename TEXT DEFAULT NULL;

ALTER TABLE `users` ADD COLUMN birthday DATE DEFAULT NULL;


---

## 権限設定

chmod 755 public/  
chmod 644 public/*.php  
chmod 755 public/setting/  
chmod 644 public/setting/*.php  

---

## 再ビルド

docker compose down  
docker compose up --build  

---

## 動作確認

以下の URL にアクセスし、画面が表示されれば完了とする。

http://パブリックIPアドレス/login.php  

## 今回のやつ  
http://18.207.250.198/login.php  
