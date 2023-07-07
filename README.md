# Free-Shogi-Board-TS (ただの将棋盤 (TypeScript版))

Copyright (C) 2023 Hideaki Sakai
See the end of the file for license conditions.
ライセンス条件に関しては、このファイルの最後を見てください。

## 概要
普通の(現実世界の)将棋盤と同じ操作ができることを目指して、なるべくシンプルに開発したウェブアプリです。
[Vue.js](https://ja.vuejs.org/) (TypeScript) と[SVG](https://developer.mozilla.org/ja/docs/Web/SVG)を利用して開発しました。

[GitHub Pages のウェブページ](https://hideboz.github.io/free-shogi-board-ts/) でご試用いただけます。

将棋盤を使用するウェブサービスなどにご利用ください。

![ただの将棋盤のスクリーンショット](images/free-shogi-board-screenshot.png)

## セットアップ方法
セットアップするには、以下のコマンドを実行してください。

```sh
npm install
```

## 開発用サーバの起動方法
開発用サーバを起動して動作を確認するには、以下のコマンドを実行して、表示されるURLにブラウザからアクセスしてください。

```sh
npm run dev
```

## ビルド方法
ビルドするには、以下のコマンドを実行してください。
`/dist`ディレクトリにビルドされます。

```sh
npm run build
```

## 操作方法
動かしたい駒をクリック(タップ)した後、動かしたい先の場所をクリック(タップ)すると、駒を動かすことができます。盤上の駒を、相手の駒があるマスに移動させると、自動的に相手の駒を自分の駒台へ移動した上で、駒が移動します。駒を右クリック(ロングタップ)すると、駒を裏返したり、向きを先後逆にしたりすることができます。将棋盤の右と左にある縦長のスペースが、先手の駒台と後手の駒台です。将棋盤の下にある横長のスペースが使わない駒置き場です。駒落ちの場合などにご利用ください。

## 謝辞
駒画像は [Shogi Images](https://sunfish-shogi.github.io/shogi-images/) のものを使用させていただきました。

## ライセンス (License)
本ソフトウェアはGNUアフェロ一般公衆ライセンスで公開されています。COPYINGを参照してください。
(This software is released under the GNU Affero General Public License, see COPYING.)

![GNUアフェロ一般公衆ライセンス](images/agplv3-with-text-100x42.png)

This file is part of Free-Shogi-Board-TS.

Free-Shogi-Board-TS is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

Free-Shogi-Board-TS is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with Free-Shogi-Board-TS.  If not, see <https://www.gnu.org/licenses/>.
