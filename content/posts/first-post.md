+++
date = '2025-05-23T18:12:12+09:00'
draft = false
title = 'skkがvivaldiで使えなかった'
+++

<!--more-->

## skk?
SKK(Simple Kana to Kanji conversion program)は，日本語入力システムの一つ．日本語を入力する際に，形態素解析をせずに変換を行うことが特徴．かな→漢字の変換の際には，変換したい熟語の語頭でshiftを押す．

## skkのインストール
OS：endeavourOS
Desktop environment：KDE Plasma
Kernel：6.14.6-arch1-1

endeavourOSはarch系のディストリビューションなので，pacmanでインストールする．インプットメソッドは，Linuxならfcitx5かibus(windowsはCorvusSKK,macはAquaSKKが有名．それぞれ仕様が少しだけ違う).今回はfcitx5-skkをインストールした．

```
# fcitx5-skkのインストール
$ sudo pacman -S fcitx5-skk
```
fcitxがインストールされていないときは `fitex5-im` も同時にインストールする．

## vivaldiで起きたこと
skkをインストールしてmozcと置き換えると，基本的にskkによる日本語入力ができるようになった．しかし，webブラウザのvivaldiでのみ日本語入力ができなかった．具体的には 英数/かな 切り替えのキーを押しても日本語が入力されないという現象が起きた． 

## 解決法
vivaldiの設定(`vivaldi://flags`)に行き，`Use Ozone Platform` をX11からWaylandに，`Enable the use of the IME API` をEnabledにするとvivaldiでも日本語入力ができるようになった．





