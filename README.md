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


# 第2章 VMサーバー構築
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

## 3. 接続確認
Host側
ssh nanu@127.0.0.1 -p 2222

## 4. トラブル対応
REMOTE HOST IDENTIFICATION HAS CHANGED
原因:VM再作成でSSHホスト鍵が変わった
対策:ssh-keygen -R [127.0.0.1]:2222
