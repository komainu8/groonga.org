---
layout: post.ja
title: Debian 6.0(squeeze)およびUbuntu 10.04(lucid)サポート終了予定について
description: Debian 6.0(squeeze)およびUbuntu 10.04(lucid)サポート終了予定について
---

Debian 6.0(squeeze)およびUbuntu 10.04(lucid)サポート終了予定について
--------------------------------------------------------------------

Groonga
Projectで今後サポートを終了する予定のリリースについてお知らせします。

### 対象について

Groonga
3.1.1では、以下のリリースについてパッケージの提供を終了することにしました。

-   Debian 6.0(squeeze)
-   Ubuntu 10.04(lucid)

### 終了時期について

2013/12/29にリリース予定のGrooga
3.1.1からパッケージの提供をとりやめます。

### 対象ユーザについて

Debian 6.0(squeeze)もしくはUbuntu
10.04(lucid)でお使いの場合には、それぞれ、Debian
7.0(wheezy)もしくはUbuntu
12.04(precise)以降へのアップグレードをおすすめします。

Debian
6.0(squeeze)自体はすでに古い安定版扱いなので、現在の安定版とされているDebian
7.0(wheezy)がリリースされて一年後となる来年5月にサポートが切れます。

### サポートを終了する背景について

現在Groonga
Projectでは、毎月29日に新しいバージョンのパッケージをビルドし、独自に用意したリポジトリで提供しています。
パッケージを提供するのは簡単に導入できるようにするためですが、これをさらに進めて独自のリポジトリを登録したりしないでも済むようにDebianのリポジトリに入れるための作業を進めています。

この作業にあたっては、Multiarchと呼ばれるマルチアーキテクチャへの対応が必要です。ただしこの機能は古いリリースである、Debian
6.0(squeeze)やUbuntu
10.04(lucid)では提供されていません。そのため、それぞれのサポートを終了することにしました。

バッケージの提供は終了しますが、環境の移行が難しい場合にはソースからビルドする手順を案内しているので、そちらを参照してください。

-   [Debianでソースからビルド](http://groonga.org/ja/docs/install/debian.html#build-from-source)
-   [Ubuntuでソースからビルド](http://groonga.org/ja/docs/install/ubuntu.html#build-from-source)

### さいごに

これからも、毎月29日のリリースで最新のGroongaをお届けしていきます。

それでは、Groongaでガンガン検索してください！
