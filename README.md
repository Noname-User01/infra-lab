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

第3章 SSH公開鍵認証構築
0. 目的

Host → VM を公開鍵認証で接続
パスワード依存の排除

1. 公開鍵作成（Host側）
ssh-keygen -t ed25519
保存場所
C:\Users\m1150.DESKTOP-IBSS08K.ssh\

秘密鍵：id_ed25519
公開鍵：id_ed25519.pub

2. 公開鍵登録（VM側）
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
公開鍵貼り付け後、保存。

3. 接続確認
ssh nanu@127.0.0.1 -p 2222
パスワード入力無しでログイン成功。

4. トラブル対応

REMOTE HOST IDENTIFICATION HAS CHANGED
原因：VM再作成でホスト鍵変更
対策：ssh-keygen -R [127.0.0.1]:2222

ssh.service failed
sshd_config 編集ミスにより起動失敗。
sudo sshd -t で構文エラー検出。
修正後、systemctl restart ssh で復旧。

Bad configuration option / unsupported option
設定行に不要文字や英文が混入。
コメントは行頭に # が必要。

Missing privilege separation directory: /run/sshd
sshd起動失敗によりディレクトリ未生成。

sudo mkdir -p /run/sshd
sudo systemctl restart ssh

新規接続不可・既存接続は維持
sshdは接続時のみ受付を担当。
セッション確立後はforkされたプロセスが継続するため、既存SSHは生存。

5. メモ

SSH設定変更手順
別セッション確保

sudo sshd -t
systemctl restart ssh

新規接続確認後に旧セッション終了
