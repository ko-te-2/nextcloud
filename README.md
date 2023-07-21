# nextcloud
オープンソースのNextcloudを使用して自作クラウドストレージを初期構築するためのテンプレート用リポジトリ

# 注意事項
- **テンプレートをご利用する際は、同梱されている.envファイルにはパスワード等を記載した状態で、リポジトリには保存しないようにご注意ください。**
- **テンプレートを利用した場合に起きたことに対する保証は一切いたしませんので、自己責任となることをご理解の上、ご利用ください。**

# テンプレート作成環境
- AWS EC2
- Docker
- docker-compose
- Ubuntu 22.04 LTS (GNU/Linux 5.15.0-1011-aws x86_64)

# Nextcloud構築手順
作成したEC2上でNextcloudを構築する
   1. EC2へログイン
   2. Dockerとdocker-composeをインストールするための事前準備
      ```
      sudo apt update
      ```
      ```
      sudo apt install -y \
          ca-certificates \
          curl \
          gnupg \
          lsb-release \
          git
      ```
      ```
      sudo mkdir -p /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      ```
      ```
      echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
          $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      ```
      ```
      sudo apt update
      ```
      ```
      sudo apt policy docker-ce
      ```
   3. Dockerをインストール
      ```
      sudo apt install -y \
          docker-ce \
          docker-ce-cli \
          containerd.io \
          docker-compose-plugin
      ```
      ```
      sudo systemctl status docker
      ```
      ```
      docker version
      ```
   4. docker-composeをインストール
      ```
      sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
      ```
      ```
      sudo chmod +x /usr/local/bin/docker-compose
      ```
      ```
      docker-compose version
      ```
   5. Nextcloudを構築する
      1. リポジトリからクローンする
        ```
        mkdir -p ~/nextcloud
        cd ~/nextcloud
        ```
        ```
        git clone <自身のリポジトリ>
        ```
      2. 環境変数ファイルを修正する
        ```
        vi .env
        ```
      3. Nginxの設定ファイルを作成する
        ```
        DOMAIN=$(grep "ENV_DOMAIN_NAME" .env | grep -v "#" | awk -F= '{ print $2 }')
        mkdir -p proxy/conf
        sudo sed -e "s/{DOMAIN}/$DOMAIN/" proxy.conf > proxy/conf/proxy.conf
        ```
      4. DockerでNextcloudを起動する
        ```bash
        sudo docker-compose up -d
        ```
        ```
        sudo docker-compose logs --follow
        ```
   6. ブラウザでサインアップする
        ```
        https://<YOUR_DOMAIN>
        ```
        - サインアップ時の情報は下記の通り
          - ユーザー名は .env ファイルの ENV_NEXTCLOUD_ADMIN_USER を使用
          - パスワードは .env ファイルの ENV_NEXTCLOUD_ADMIN_USER_PASSWORD を使用
   7. Nextcloudを停止する（ターミナルへ戻る）
      ```
      sudo docker-compose down
      ```
   8. Nextcloudの設定を変更する
      ```
      sudo chmod 766 conf/config/config.php
      ```
      ```
      sudo sed -i -e '/<?php/a $PROXY_ADDR = gethostbyname("proxy");' conf/config/config.php
      sudo sed -i -e '/^);/i \ \'\''trusted_proxies'\'' => $PROXY_ADDR,' conf/config/config.php
      ```
   9. Nextcloudを起動する
      ```bash
      sudo docker-compose up -d
      ```
      ```
      sudo docker ps
      ```
   10. ブラウザで外部ストレージを設定する
       - "アプリ"  →  "External storage support"を「有効にする」
       - "設定"  →  "管理"  →  "外部ストレージ"
         - 下記のように入力／選択する  
            |  項目名          |  値  |
            | ----            | ---- |
            |  フォルダー名     |  _<お好みのフォルダー名>_  |
            |  外部ストレージ   |  "Amazon S3" を選択する |
            |  認証            |  アクセスキー  |
            |  バケット名       |  _<作成したS3バケット名>_  |
            |  アクセスキー     |  _<作成したIAMのシークレットキー>_  |
            |  シークレットキー  |  _<作成したIAMのシークレットキー>_  |
            |  SSLを有効       |  チェックを入れる  |
       - 必要に応じて下記を実施
         - グループ作成
         - ユーザー作成

## その他コマンド
  - 起動したコンテナを停止して削除する
      ```bash
      sudo docker-compose down --rmi all --volumes --remove-orphans
      ```
  - 各コンテナへログインする
      ```bash
      sudo docker-compose exec nextcloud /bin/bash
      ```
      ```bash
      sudo docker-compose exec proxy /bin/bash
      ```
  - コンテナ起動後のログを確認する
      ```
      sudo docker-compose logs --follow
      ```



# Installation
# Reference - Non Official
   - [EC2 と RDS を利用したNextcloud 環境の構築 - Nextcloud 環境の構築を通じて AWS での環境構築を体験する①](https://qiita.com/S_Katz/items/edadf33755c0d834eb46#%E3%83%AD%E3%83%BC%E3%83%89%E3%83%90%E3%83%A9%E3%83%B3%E3%82%B5%E3%83%BC%E3%81%AE%E8%BF%BD%E5%8A%A0)
   - [Docker/docker-compose/Proxy configuration](https://blog.seigo2016.com/blog/h-blxsnew_s)
   - [ALB を利用したサーバー負荷分散、可用性向上に向けた取り組み - Nextcloud 環境の構築を通じて AWS での環境構築を体験する④](https://qiita.com/S_Katz/items/edadf33755c0d834eb46#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)
   - [S3をNextCloudのプライマリストレージ(データディレクトリ)に設定する方法](https://qiita.com/myoshimi/items/953119efc90d8c117fd7)
   - [nextcloudによるプライベートオンラインストレージの構築](https://qiita.com/h_amakasu/items/9947d81bc0d3bc18f06d)
   - [Let's Encrypt](https://github.com/nginx-proxy/acme-companion)
   - [NextCloudで外部からのアクセスを可能にする](https://satsumahomeserver.com/blog/293491)
   - [Domain & Proxy](https://linuxfun.org/2022/07/08/nextcloud-dompose-nginx-phpfpm-on-lightsail/)

## Others
- [サーバレス版KubernetesでNextCloudによるAWS、AlibabaCloudのマルチクラウド運用を実現してみる※構築手順付き](https://www.softbank.jp/biz/blog/cloud-technology/articles/202206/nextcloud-on-severless-k8s-for-multicloud/)
- [NextCloudと無料サーバー + AWS S3で、無限に拡張できる専用クラウドストレージを格安で構築する](https://www.serversus.work/topics/ilsh3npp5padpmc8lpsv/)
- [Docker+Nginx+ownCloud+SSLで構築した](https://kikei.github.io/server/2017/02/19/owncloud.html)
- [NextCloudとS3を連携させた話](https://qiita.com/yamagami2211/items/d4eef790187fc9d49a09)
- [Nextcloud Official](https://docs.nextcloud.com/server/16/admin_manual/installation/source_installation.html#installing-via-snap-packages)
- [Nextcloud Official Docker image](https://hub.docker.com/_/nextcloud/)
- [Sample Dockerfile](https://github.com/aws-samples/aws-serverless-nextcloud)
- [Sample docker-compose①](https://github.com/nextcloud/docker/tree/master/.examples)
- [Sample docker-compose②](https://qiita.com/lukapla/items/a36a6526d74ff5926231)
- [Sample docker-compose③](https://akitoshiblogsite.com/docker-compose-nextcloud-nginx/)
- [Sample docker-compose④/Proxy configuration①](https://www.code-lab.net/?p=22084)
- [Proxy configuration②](https://kikei.github.io/server/2019/08/11/nextcloud.html)
- [Proxy configuration③](https://kikei.github.io/server/2017/02/19/owncloud.html)

# Install to Ubuntu

# Environment
- AWS EC2
  - Ubuntu 22.04 LTS (GNU/Linux 5.15.0-1011-aws x86_64)
- Finish DNS, ALB, ACM Setting

# Install Manual
- [★Install Nextcloud](https://qiita.com/S_Katz/items/756ca04ecece844ce503)
- [★docker-compose.yml](https://blog.seigo2016.com/blog/h-blxsnew_s)

## Official
- [NextCloud](https://docs.nextcloud.com/server/16/admin_manual/installation/source_installation.html)
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [docker-compose](https://docs.docker.com/compose/install/compose-plugin/#installing-compose-on-linux-systems)
- [Nginx](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

## Non Official
- [usage certbot ①](https://symfoware.blog.fc2.com/blog-entry-2511.html)
- [usage certbot ②](https://qiita.com/shin4488/items/8738e13b92143c88aab6)
- [Others](https://www.one-tab.com/page/ARtgwRyGTlueCM-75Dk9Zg)

