---
title: "OSCで心拍数を取得しアバターにいい感じに表示する"
emoji: "❤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["VRChat", "OSC", "MiBand"]
published: false
---

# まえがき

タイトルのまんまです。
VRChat コミュニティを見ていると「OSC を使ってアバターに心拍数を出してみた！」という方がちらほら居るのですが、それのやり方等に関する記事が少なく、上手い事行かなかったりしたので自分で記事を書いてみました。
拙い文章ではありますが、誰かの助けになれば幸いです。

# やり方

前提として以下の環境が揃っている必要があります。何か抜けているだけでも動きません。

| 環境           | バージョン等                                                          |
| -------------- | --------------------------------------------------------------------- |
| Mi Band        | Mi Band 4 以降                                                        |
| Mi Fit         | Mi アカウント も必須。Mi Band の AuthKey を取得するために必要         |
| Bluetooth      | 4.0/4.2 BLE 以降対応のドングル/マザーボードが必要                     |
| Python         | 今回は 3.10.2 を使用                                                  |
| Unity          | 2019.4.31f1                                                           |
| VRCSDK3        | VRCSDK3-AVATAR-2022.02.16.19.13                                       |
| VRChat         | Build 1169 以降                                                       |
| 適当なブラウザ | モダンなブラウザであればなんでもいいかも？今回は Google Chrome を使用 |

なお、Mi Fit をインストールする端末が Android の場合は改造された Mi Fit をインストールする事でも AuthKey が得られますが、今回はその方法は書きません。
この記事で紹介する Python を用いた手順より楽なため、Python の実行環境がなかったり、手早く済ませたい方は自分で検索してやってみてください。

## セットアップ

### AuthKey 取得

まずは Mi Band の AuthKey を取得する必要があります。
argento 氏が開発した[huami-token](https://github.com/argrento/huami-token)というプログラムで取得出来ます。
「Download ZIP」や `git clone` 等で適当にコードを引っ張ってきてください。
![](/images/content-bdbba89ff8df5f/1.png)
cmd や Powershell を開き `pip3 install -r requirements.txt` で必要モジュールをインストールします。
特にエラーが出なければ大丈夫です。
次に `python huami_token.py --method xiaomi --bt_keys` を実行します。
以下のようなメッセージが帰ってくるため、リンクを開き自身の Mi アカウントでログインします。
![](/images/content-bdbba89ff8df5f/2.png)
:::message alert
この際、ログインする Mi アカウントが Mi Fit と紐付けられている必要があります。
:::
ログインすると、「安全でないページ」だの何だの言われますが、それらを無視して URL をコピーし、先程のターミナルに戻りペーストします。
![](/images/content-bdbba89ff8df5f/3.png)
なんかいっぱい出てきますが、「ACT」の値が「1」になっている auth_key が Mi Fit 上でアクティブ化されているデバイスです。
使用したい Mi Band がアクティブ化されていない場合、Mi Fit 上からアクティブ化してください。
これで AuthKey の取得は完了です。この後使うためテキストファイル等に控えてください。

### OSC 側アプリケーション

次に取得した AuthKey を用いて Mi Band から心拍数を引っ張り、それを OSC で VRChat に送るアプリケーションを用意します。
vard88508 氏が開発した[vrc-osc-miband-hrm](https://github.com/vard88508/vrc-osc-miband-hrm)を使用します。
[リリースページ](https://github.com/vard88508/vrc-osc-miband-hrm/releases)で exe が配布されていますが、必要であればリポジトリをローカルにクローンし、自身で Node.js の環境を建てて使用してください。
この記事では exe を使用します。

リリースページから exe をダウンロードし、適当な場所に配置します。VRChat を起動する際ついでに起動しやすいよう、デスクトップ等に配置したり、スタートメニューやタスクバーにピン留めしても良いでしょう。
![](/images/content-bdbba89ff8df5f/4.png)
起動時、多分 SmartScreen に怒られますが慌てず「詳細情報」→「実行」から起動してください。
![](/images/content-bdbba89ff8df5f/5.png)
起動するとブラウザが立ち上がり、AuthKey の入力が求められます。先程取得した AuthKey の**先頭 0x を削除し**入力してください。空白があると死にます。
この際 Mi Band 4/5 等の古いデバイスを使用している際は「Older device support」のチェックを入れてください。
![](/images/content-bdbba89ff8df5f/6.png)
入力し Connect をクリックすると、ブラウザから「ペア設定の要求」が出てきます。自身の Mi Band とペアリングしてください。
![](/images/content-bdbba89ff8df5f/7.png)
ペアリング完了するとブラウザ上で心拍数が表示されます。表示されない場合は 30 秒程度待つと表示されます。
![](/images/content-bdbba89ff8df5f/8.png) _超でかい ブラウザをウィンドウキャプチャすれば OBS とかでも使えそうかも？_
