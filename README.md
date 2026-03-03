2026/02/27 star
# ubuntu-lab-01 構築ログ

# 第1章 VMサーバー構築
## 0. 目的
Linuxサーバ構築の基礎練習
3年ロードマップ Year1 インフラ基礎

## 1. 環境詳細
Host OS : Windows 10
CPU : 2コア
RAM : 2024MB
VirtualBox : version ?
Ubuntu ISO : ubuntu-24.04.4-live-server-amd64.iso

## 2. 作業ログ
2026-02-27
VM作成 ubuntu-lab-01
user nanu 作成
ログイン成功

## 3. トラブル対応
ログイン不可
原因：USキーボード配列
対策：Shift+2で@


# 第2章 SSHサーバ導入構築
## 0. 目的
Host → VM に SSH接続できるようにする

## 1. SSHサーバ導入
sudo apt update
sudo apt install openssh-server

確認
ls -l /usr/sbin/sshd
ls /etc/ssh/ssh_host_*

## 2. VirtualBox設定
ネットワーク : NAT
ポートフォワーディング
Host IP : 127.0.0.1
Host Port : 2222
Guest Port : 22
Protocol : TCP

ssh ホスト鍵とユーザー鍵の違い

2.1  ホスト鍵:サーバーの身分証(なりすまし対策)
     生成タイミング→はOpenSSHサーバーをインストール時に自動生成
     {保存場所（Ubuntu側）}
        /etc/ssh/ssh_host_rsa_key
        /etc/ssh/ssh_host_ecdsa_key
        /etc/ssh/ssh_host_ed25519_key

2.2 ユーザー鍵(公開鍵認証用)
    役割:パスワード代わりのログイン認証
    生成コマンド→ssh-keygen -t ed25519
     {保存場所（クライアント側）}
        ~/.ssh/id_ed25519        ← 秘密鍵（絶対に渡さない）
        ~/.ssh/id_ed25519.pub    ← 公開鍵（サーバーに登録）
        
保存場所（クライアント側）
~/.ssh/id_ed25519        ← 秘密鍵（絶対に渡さない）
~/.ssh/id_ed25519.pub    ← 公開鍵（サーバーに登録）
        
これらは不正ログインを防ぐ為に活用する
ホスト鍵 ＝ 「このサーバーは本物か？」
ユーザー鍵 ＝ 「このユーザーは本人か？」

## 3. 接続確認
Host側
ssh nanu@127.0.0.1 -p 2222

## 4. トラブル対応
REMOTE HOST IDENTIFICATION HAS CHANGED
原因:VM再作成でSSHホスト鍵が変わった
対策:ssh-keygen -R [127.0.0.1]:2222

# 第3章 SSHセキュリティ強化
## 1. 目的

公開鍵認証のみ許可
ブルートフォース攻撃対策
Year1 インフラ基礎

## 2. 公開鍵認証設定

ssh-keygen -t ed25519
ssh-copy-id nanu@127.0.0.1 -p 2222

ログイン確認 → パスワード無しで接続成功

## 3. 認証制限

PasswordAuthentication no
PermitRootLogin no
sshd -t
systemctl restart ssh

## 4. fail2ban導入

インストール
enable
status確認

ログ監視 → 失敗回数でBAN

## 5. UFW設定

default deny incoming
allow 22/tcp
status確認

## 6. セキュリティまとめ

鍵認証 → 総当たり不可
root禁止 → 権限集中回避
fail2ban → bot遮断
UFW → 攻撃面削減

## 3. セキュリティ理由

開放ポート少ない → 攻撃経路減少
fail2ban と併用 → 二重防御

# 第4章 nginx + FastAPI リバースプロキシ構築

## 0. 目的
nginx を公開入口とし、FastAPI を application 層として安全に動作させる。

## 1. 構成概要
- nginx が外部からの HTTP リクエストを受ける
- nginx が uvicorn（FastAPI）へ転送（reverse proxy）
- systemd が uvicorn を常駐管理

```
Client
  ↓ HTTP:80
nginx
  ↓ proxy_pass → 127.0.0.1:8000
uvicorn (FastAPI)
```

## 2. FastAPI（uvicorn）の常駐化
### systemd サービス作成
```bash
sudo nano /etc/systemd/system/fastapi.service
```

サービス定義:
```ini
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

サービス反映:
```bash
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
```

確認:
```bash
sudo systemctl status fastapi
```

## 3. nginx リバースプロキシ設定
### 設定ファイル作成
```bash
sudo nano /etc/nginx/sites-available/fastapi
```

内容:
```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 有効化
```bash
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/
```

### デフォルト設定解除
```bash
sudo rm /etc/nginx/sites-enabled/default
```

設定テスト:
```bash
sudo nginx -t
```

再起動:
```bash
sudo systemctl restart nginx
```

## 4. 確認
FastAPI へ nginx を介してアクセスできる:
```bash
curl http://localhost
# {"message":"Hello World"}
```

FastAPI 単体:
```bash
curl http://127.0.0.1:8000
# {"message":"Hello World"}
```

## 5. 補足ポイント
- nginx は外部からの入口としての交通整理と境界防御を担当
- uvicorn（FastAPI）は内部ポート 8000 でアプリ実行
- systemd はプロセス監視と自動再起動を担当

