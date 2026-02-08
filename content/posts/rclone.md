+++
date = '2026-02-08T15:30:37+09:00'
draft = false
title = 'GoogleDriveをRcloneでクラウド接続'
math = false
tags = ["tools", "cloud", "zotero"]
+++

<!--more-->

Google Drive に zotero で管理している文献の pdf ファイルを置いておくことにしています．zotero には文献のメタデータのみを置いておくことで，zotero 無料版のデータ容量を実質的に超えた量の文献情報を保存することができます．

ここでは arch linux でクラウドストレージを同期する流れを説明します．

## pdf保存先のフォルダを作成

はじめに，Google Drive内にpdfを置くディレクトリを作成しておきます．ここでは，`papers` というディレクトリを作成したとしてすすめます．

## プラグイン ZotMoov

Zotero ver.7 以降でpdfを移動させるためには ZotMoovをいうプラグインを使用する必要があります．Zotfileが以前は使用されていましたが，現在は動きません．
`.xpi` ファイルをダウンロードして zotero にインストールしてください．
インストールしたらEdit から Settingに入り，zotmoov の "Directory to Move/Copy Files To" の先のパスを `~/GoogleDrive/Papers` にしておきます．

これでpdfデータの移動先が GoogleDrive 内の`papers` になります．

## GoogleDrive と同期

最後に`~/Papers` ディレクトリを Google Drive と同期させます．
 
Arch linux には GoogleDrive の公式クライアントがないため，`rclone` という cli ツールを使います．

### インストールと設定

`yay -S rclone` でインストールできます．

次に同期先として GoogleDrive を登録します．`rclone` ではターミナルにて対話形式で同期先を登録可能です．
`rclone config` と打つと，設定が始まるので，以下のようにすすめていきます．

```zsh
n/s/q> n    # 新しいリモート作成
name> gdrive    # リモート名を設定(gdrive，ggldrvなど)
Storage> drive  # GoogleDrive を選択
client_id> Enter    # 空欄
client_secret> Enter    # 空欄
scope> 1    #   フルアクセスを選択
service_account_file> Enter # 空欄
Edit advanced config? n    # 高度な設定なし
Use auto config? y  # ブラウザ認証 → ブラウザが自動で開くので Google アカウントでログインし，アクセスを許可 → Success! と出たらターミナルに戻る
Configure this as a Shared Drive? n    # 共有ドライブでない
Keep this "gdrive" remote? y    # 設定を保存
e/n/d/r/c/s/q> q    # 終了
```

`rclone ls gdrive: --max-depth 1` で GogleDrive の中身が表示されれば登録できています．

設定がおわったので，`rclone` でマウントします．
`rclone mount [上で設定した リモート名]: ~/GoogleDrive --daemon`

すると `ls ~/GoogleDrive` でメイン PC の方でも GoogleDrive の中身が見えるようになります．

これでzoteroに登録した文献のpdfデータた GoogleDrive に移行，保存されます．

## メインPC の `~/GoogleDrive/`

メインPCで `~/GoogleDrive/` を Dolphin などのファイルマネージャで開くと，pdfファイルのプレビューが見えなくなっています(普段から見えない場合は見た目は変わらないかも？ここはツールによりけり)．

これは `rclone mount` によって仮想ドライブで GoogleDrive の中身を見ているためです．そのためメインPCのディスク容量を圧迫しないようになっています．ファイルをクリックしたときにクラウドからダウンロードされ，閲覧できるようになります．もちろん，`papers` 以外のファイルも同じ状態です．

## GoogleDriveと常に接続する

今の状態では，PC の電源を落としたり再起動すると GoogleDrive とのマウントは解除され，再度 `~/GoogleDrive`を開くためには `rclone mount` する必要があります．しかし，`rclone mount [リモート名]: ~/GoogleDrive --daemon` にキャッシュ設定を同時にする必要があります(以下)．
```
rclone mount gdrive: ~/GoogleDrive --daemon \
  --vfs-cache-mode full \
  --vfs-cache-max-size 5G \
  --vfs-cache-max-age 24h
```

毎回このコマンドを打つのが面倒なので，今回はPCを起動したら自動で接続するようにします．

始めに．`mkdir -p ~/.config/systemd/user/` で Systemd のユーザ設定ディレクトリを作成します．次に，`user` ディレクトリに `rclone-gdrive.service` ファイルを作成し，以下を記述します．

```
[Unit]
Description=Google Drive (rclone)
AssertPathIsDirectory=%h/GoogleDrive
After=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/rclone mount gdrive: %h/GoogleDrive \
 --vfs-cache-mode full \
 --vfs-cache-max-size 5G \
 --vfs-cache-max-age 24h
ExecStop=/usr/bin/fusermount -u %h/GoogleDrive
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

保存したら，ターミナルで `systemctl --user enable --now rclone-gdrive` と打ち，自動起動を有効化します．これでPCの起動後に自動的に GoogleDrive が接続されます．
