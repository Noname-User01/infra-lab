# 1. VM環境設定
設定... → 一般 → 詳細
共有クリップボード：双方向
ドラッグ＆ドロップ：双方向

対象プロジェクト起動

## 2. Guest Additionの導入に移る
VM起動後 Virtual Boxメニュー
→デバイス → Guest Additions CDイメージを挿入

VM内で実行するコマンド
```
sudo apt update
sudo apt install build-essential dkms linux-headers-$(uname -r)
sudo mkdir /media/cdrom
sudo mount /dev/cdrom /media/cdrom
sudo /media/cdrom/VBoxLinuxAdditions.run
sudo reboot
```

## 3. コピーできるかの確認
try it.

---------------------------------------------------------------------------
<br>


## 1入力バグが起きた時の対処方法
###ssh server 起動確認コマンド
```
 sudo systemctl status ssh
```
端末のstty設定が壊れた時
→入力遅延が起き、入力 → enter   で初めて画面に入力が表示される。

この場合コマンドラインに以下のコマンドを入力する事で解決
```
reset
```
```
stty sane
```

ssh鍵確認コマンド
```
ls -l ~/.ssh
```

---------------------------------------------------------------------------
<br>


ssh ログインパスワードを消去する手順
1. まずはログイン確認をする
```
ssh [Username]@localhost -p 2222
```
※別ターミナルでも成功するかを確認する。条件を変えて確認しないとロックアウト → ログインできなくなる
→今回の設定ではlocalhostの部分を127.0.0.1にする
```
ssh nanu@127.0.0.1 -p 2222
```

## 2. sshd_config編集
```
sudo nano /etc/ssh/sshd_config
```
次に以下の項目がコメントアウトされている場合それを外す
・PasswordAuthentication no
・ChallengeResponseAuthentication no
・UsePAM yes
・PubkeyAuthentication yes
2.1. ついでにrootログインを停止する
```
PermitRootLogin no    #not_root_login
```

///////////////////////////////////  <br>
ssh一時的に接続できなくなった  <br>
既存の接続は有効でコマンドは実行できるからトラブルを解決次第対策を考える。  <br>
//////////////////////////////////  <br>

ssh接続状態でsshサーバが停止してもsshdが停止しても切れない。
新規のssh接続はsshdが動いていないと作れない。

sudo aot update    //install comannd update
sudo apt install openssh-server -y

sudo systemctl status ssh
//sshのソケットが有効になっているかの確認

以下sshd(SSH server)の設定ファイルを起動せずに構文チェックが可能なコマンド
```
sudo sshd -t
```
意味は以下の通り
・sudo → 管理者で実行
・sshd → SSH server programの本体
・-t   → test modeの起動が可能であり/etc/sh/sshd_config の文法のみを検閲する

//ホストコマンド
ssh-keygen -t ed25519    //共通暗号キー生成
ssh nanu@localhost    //sshへログインを試みる

---------------------------------------------------------------------------
<br>


## Pythonを導入する方法
まずは以下のコマンドを入力しディレクトリを作成
```
mkdir ~.infra-lab    //ディレクトリの作成
cd ~/infra-lab     //作成したディレクトリへ移動
```

### Pythonのインストール
```
sudo apt update
sudo apt install python
sudo apt install pyrhon3.12-venv

python -m venv venv    //仮想環境作成
source venv/bin/activate     //仮想環境有効化

pip install --upgrade pip    //pip 更新
pip install fastapi uvicorn    //fastapi & uvicornのインストール
```

サーバー起動コマンド
```
uvicorn test:app --host 127.0.0.1 --port 8000
```

fastAPI用の.py新規ファイルを作る方法
```
nano [ファイル名]
```
内容
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return{"message:"Hello World"}
```

ブラウザに    http://127.0.0.1:8000    を入力

新規でポートもあけておきましょう
```
sudo ufw allow 8000/tcp
sudo ufw status verbose
```

---------------------------------------------------------------------------
<br>


## FastAPIの構築







systemdサービス化
```
sudo nano /etc/systemd/system/fastapi.service
```

記載内容
```
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=nanu
WorkingDirectory=/home/nanu/infra-lab/app
ExecStart=/home/nanu/infra-lab/venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

サーバー状態確認
```
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
sudo systemctl status fastapi
```
stauts → active☑ 確認



## Ngineの導入
```
sudo apt install nginx
```

サーバー設定ファイル
```
sudo nano /etc/nginx/sites-available/fastapi
```

内容
```
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

















