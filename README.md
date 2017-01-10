# 環境構築手順

## 前提条件
- bitbucketのアカウント情報を取得している事

- PCをVT-Xに対応している事。以下の方法で確認する
    - Windows
        - PCを再起動してBIOSセットアップ画面で、V-TXをenableに設定する
        - 参考URL
            - http://qiita.com/hiroyasu55/items/11a4c996b0c62450940f

## 構築手順

#### brew-caskをインストール
- Windows
    - 不要
- MacOS
    - インストールコマンド
        - brew install caskroom/cask/brew-cask

#### RLoginをインストール
- Windows
    - 役割
        - SSHクライアント
     - ダウンロードURL（rlogin.zipをクリック）
        - http://nanno.dip.jp/softlib/
    - 参考URL
        - http://qiita.com/pocket8137/items/e294715b5154487b9ae0
- MacOS
    - 不要

#### Chocolateyをインストール
- Windows
    - 役割
        - Windows用パッケージ管理
    - 参考URL
        - http://qiita.com/konta220/items/95b40b4647a737cb51aa
        - https://chocolatey.org/install
    - 備考
        - 管理者権限で実行し、コマンドプロンプトでインストールする必要
- MacOS
    - 不要

#### vagrantのインストール
- Windows
    - インストールコマンド
        - `cinst vagrant`
- MacOS
    - インストールコマンド
        - `brew cask install vagrant`
- 参考URL
    - http://qiita.com/hiroyasu55/items/11a4c996b0c62450940f

#### VirtualBoxのインストール
- Windows
    - インストールコマンド
        - `cinst virtualbox`
- MacOS
    - インストールコマンド
          - `brew cask install virtualbox`
- 参考URL
    - http://qiita.com/hiroyasu55/items/11a4c996b0c62450940f

#### PATHを通す
- Windows
    - コマンドプロンプトを再起動
- Mac
    - 不要

#### vagrantの設定ファイルを配置
- Windows、MacOS
    - vagrantディレクトリを作成
        - cd c:\users\ユーザ名
        - mkdir vagrant
    - 設定ファイルをvagrantディレクトリに配置
        - BOXパス
            - https://uhuru.app.box.com/files/0/f/8603385869/vagrant
        - ファイル名
            - bootstrap.sh
            - Vagrantfile
        - 配置先
            - c:\users\ユーザ名\vagrant\
    - bootstrap.shの設定値（メールアドレス、ユーザ名、パスワード、３つのブランチ名）を書き換える
        - bootstrap.sh内のコメントを参照
    - vagrantディレクトリ下で`vagrant up`を実行
        - 15分くらい待つ。。。。

#### 仮想環境へSSHでログイン
- Windows
    - RLonginに下記を設定
        - IP：192.168.33.10
        - ユーザ名：vagrant
        - パスワード：vagrant
- MacOS
    - cd /Users/{ユーザ名}/vagrant
    - vagrant ssh

#### 仮想環境上のPostgres設定を変更
1. 権限付与
    - su postgres
        - パスワード入力
    - psql
    - alter role postgres with encrypted password 'postgres';
    - \q
    - exit
2. スキーマ作成
    - cd /home/vagrant/sato/back-end/　→/home/vagrant/sato/sato-sos-back-end/　
    - bundle config build.nokogiri --use-system-libraries=true --with-xml2-include=/usr/include/libxml2
    - bundle install
    - bundle exec rake db:setup
3. 初期データを突っ込む
    - bundle exec rake db:seed



#### dev環境のPostgresデータを仮想環境にインポートする方法
    - https://dashboard.heroku.com/apps/uhuru-backend-dev にログインしてから
      OverviewタブのHeroku Postgresアイコンを押す。
    - BackupsのCapture Backupボタンを押してから,Downloadボタンを押す。
    - ダウンロードしたファイル名をlatest.dump.devにリネームして、共有フォルダーである~/vagrantにコピーする。
    - 仮想環境に接続してダウンロードしたlatest.dump.devをPostgresにインポートする
        # cd ~/vagrant
        # vagrant ssh
        # sudo su - postgres
        # cd /vagrant
        # pg_restore --verbose --clean --no-acl --no-owner -h localhost -d postgres latest.dump.dev

    - 以下のユーザでログインできる。
        - k.hirakawa+devT001@sub-uhuru.jp
        - k.hirakawa+devH001@sub-uhuru.jp
        - k.hirakawa+devE001@sub-uhuru.jp
        - aki.nakabayashi+devq001@sub-uhuru.jp
        - aki.nakabayashi+devj001@sub-uhuru.jp
        - aki.nakabayashi+devf001@sub-uhuru.jp
    - PASS：devdev
    - 「Oh noes, an error!
         The application made an invalid OAuth request.
         Unknown client ID f9fqc45jw0by8a891ko0uvu7yl6r8o7」
        と怒られたら、
        oauth2_clientsテーブルに怒られたclient IDのレコードを追加する。
        このとき、redirect_uriは以下にする。
        http://192.168.33.10/#/receive_token

#### front-endの準備っぽい事
- front-endでnpm installする
```
    cd /home/vagrant/sato/front-end →/home/vagrant/sato/sato-sos-front-end/
    sudo npm install
```

- dist.shを実行
```
    ./dist.sh
```

#### 仮想プリンタの追加
- Windows
    ローカルでパワーシェルを叩く
- MacOS
    ローカルでシェルを叩く

#### サーバ起動
- back-end
```
    cd /home/vagrant/sato/back-end
    bundle exec rackup config.ru -p 9292 -o 0.0.0.0
```
- front-end
```
    cd /home/vagrant/sato/front-end
    gulp serve-local
```

#### 起動後の確認
- back-end
    - ブラウザで`192.168.33.10:9292`にアクセス
- front-end
    - ブラウザで`192.168.33.10:1337`にアクセス
    - 備考
        - back-endも同時に起動していないとうまく動かないです

### postgresの再起動コマンド
sudo service postgresql restart
