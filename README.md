# AWS EC2 + Docker を用いた Web サービス構築手順書

## EC2 へ SSH 接続
```
ssh ec2-user@IPアドレス -i 秘密鍵ファイルのパス
```
---

## 基本ツールのインストール
```
sudo yum install vim -y

sudo yum install screen -y  

screen
```
---

## Docker のインストール
```
sudo yum install -y docker

sudo systemctl start docker

sudo systemctl enable docker

sudo usermod -a -G docker ec2-user  

exit  
```
※ 再ログイン後、再度 screen を実行する

---

## Docker Compose のインストール
```
sudo mkdir -p /usr/local/lib/docker/cli-plugins/  

sudo curl -SL https://github.com/docker/compose/releases/download/v2.36.0/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose  

sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose  

docker compose version  
```
---

## 作業ディレクトリ作成
```
mkdir dockertest

cd dockertest  

sudo chown -R ec2-user:ec2-user ~/dockertest  
```
---

## PHP 設定（php.ini）
```
vim php.ini
```
https://github.com/haru-masu/kouki_Webkakuron_kadai/blob/main/php.ini

---

## Dockerfile（PHP）
```
vim Dockerfile
```
https://github.com/haru-masu/kouki_Webkakuron_kadai/blob/main/Dockerfile

---

## Docker Compose 設定（compose.yml）   
```
vim compose.yml
```
https://github.com/haru-masu/kouki_Webkakuron_kadai/blob/main/compose.yml

---

## コンテナ起動
```
docker compose up -d --build  
```
---

## Nginx 設定
```
mkdir -p nginx/conf.d

sudo chown -R ec2-user:ec2-user nginx  
```
server 設定ファイル（default.conf）を作成する。
```
vim nginx/conf.d/default.conf
```
https://github.com/haru-masu/kouki_Webkakuron_kadai/blob/main/nginx/conf.d/default.conf

---

## public ディレクトリ準備
```
mkdir public  

sudo chown -R ec2-user:ec2-user /home/ec2-user/dockertest/public  
```
※sshでEC2インスタンスに入らず、powershell上で行ってください。
githubにあるリポジトリをzipで圧縮し、解凍する。

---
### PowerShell からファイル転送

scp -i {秘密鍵のファイルパス} -r {publicディレクトリのファイルパス} ec2-user@{IPアドレス}:/home/ec2-user/dockertest



## データベース作成
https://github.com/haru-masu/kouki_Webkakuron_kadai/blob/main/sql

---

## 権限設定
```
chmod 755 public/

chmod 644 public/*.php  

chmod 755 public/setting/

chmod 644 public/setting/*.php  
```
---

## 再ビルド
```
docker compose down

docker compose up --build  
```
---

## 動作確認

以下の URL にアクセスし、画面が表示されれば完了とする。

http://パブリックIPアドレス/login.php  

