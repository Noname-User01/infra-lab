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

## 3. パスワードログイン禁止

/etc/ssh/sshd_config 編集

PasswordAuthentication no
PermitRootLogin no

sudo sshd -t → configチェック
sudo systemctl restart ssh

既存SSHは維持、新規接続は設定適用

## 4. トラブル

sshd -t エラー
unsupported option Depending/on
→ コメント文を設定行に入れていた

Missing privilege separation directory /run/sshd
→ sshサービス起動前に手動確認しただけ

sudo 無効表示
→ root禁止ではなく sudo権限不足

## 5. セキュリティ理由

パスワードログイン → 総当たり可能
公開鍵認証 → 秘密鍵必須

rootログイン禁止 → 攻撃対象を1段減らす

# 第4章 fail2ban 導入
## 1. 目的

ログイン失敗を自動検知
攻撃IPを一時BAN

Year1 必須項目

## 2. 導入

sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

確認
sudo fail2ban-client status sshd

jail sshd → active

## 3. 動作

SSHログ監視
連続失敗 → firewallでIP遮断

ブルートフォース対策

# 第5章 UFW ファイアウォール
## 1. 目的

不要ポート遮断
攻撃面積削減

## 2. 設定

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw enable

確認
sudo ufw status verbose

22/tcp のみ許可

## 3. セキュリティ理由

開放ポート少ない → 攻撃経路減少
fail2ban と併用 → 二重防御
