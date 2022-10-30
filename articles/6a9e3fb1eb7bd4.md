---
title: '【TypeScript】tscコマンドが使えない時の対処法'
emoji: '🦈'
type: 'tech'
topics:
  - 'javascript'
  - 'nodejs'
  - 'typescript'
  - 'vscode'
published: true
published_at: '2021-02-19 17:45'
---

# 状況

1. `$ npm i typescript -g`した後に`$ tsc --version`としても`command not found`となる場合
2. iTerm2 では tsc コマンドが使えるが、VSCode で tsc コマンドが使えない場合

# Too Long, Didn’t Read

1. PATH を通す
2. 再起動、Node の再インストール

# 環境

- MacOS Big Sur v11.2.1
- node v15.9.0
- npm v7.5.3

# 対処

1. `$ npm i typescript -g`した後に`$ tsc --version`としても`command not found`となる場合

PATH を通します。
参考：[PATH を通すとは？](https://qiita.com/soarflat/items/09be6ab9cd91d366bf71)

npm がグローバルで使用しているディレクトリを確認する

```terminal
$ npm bin -g
/Users/xxx/.nodebrew/node/v15.9.0/bin
```

`.zschr`に追記する（bash の方は.bash_profile で置き換えて読んでください)

```terminal
$ sudo vi ~/.zshrc
```

`i`キーを押して INSERT モードにし、最終行に下記を追記。
私は見やすいようにコメントアウトで説明を書いています。

```terminal
export PATH=$PATH:`npm bin -g`
```

`esc`キー →`:wq`→`enterキー`で保存し、`vi`を抜けます。
`$ source ~/.zshrc`コマンドでやっと編集内容を反映できます。

何も出力されなければ反映成功なのですが、何故か私は反映できないと怒られてしまいました。

```terminal
$ source ~/.zshrc
(not in PATH env variable)
/Users/xxx/.zshrc:export:129: not valid in this context: /Users/xxx/.nodebrew/node/v15.9.0/bin
```

PATH を直で書いてみる。

```terminal
(省略）
# TypeScript
export PATH="/Users/xxx/.nodebrew/node/v15.9.0/bin$PATH"
```

再び`$ souce ~/.zshrc`で反映できました。

```terminal
$ tsc --version
Version 4.1.5
```

1. は以上です。

---

2. iTerm2 では tsc コマンドが使えるが、VSCode で tsc コマンドが使えない場合

上記１のやり方で、tsc コマンドを iterm2 で使うことが可能になったわけですが、何故か VSCode のターミナルでは`command not found`のままだったので解決法を探ることに。

1. VSCode の再起動
2. Mac の再起動
3. Node, TS を入れ直す

私は３でやっと直りました。

まず、npm をアンインストール

```terminal
$ npm uninstall -g npm
$ rm -rf .npm \
>    node_modules
```

続いて Node のアンインストール。ソース版だとアンインストールの方法が違います。今までソース版を使っていた方はこれを気にパッケージ版で楽に管理しましょう！

```terminal
$ brew uninstall --force node
...

# homebrewのアップデート
$ brew update

# nodebrewのインストール
$ curl -L git.io/nodebrew | perl - setup
$ export PATH=$HOME/.nodebrew/current/bin:$PATH
$ source ~/.zshrc
```

以下のコマンドを打ち、nodebrew のバージョンが出ればインストール成功。

```terminal
$ nodebrew -v
nodebrew 1.0.1
```

Nodejs のインストール。

バージョン指定もしくは最新のものをインストールできます。
｛｝にどちらかを入力

- v○.○.○（例 v6.11.4） → バージョン 6.11.4
- latest → 最新“

```terminal
$ nodebrew install-binary {}
...
Installed successfully

# インストールしたバージョンのものが入っているか確認
$ nodebrew ls
v15.9.0

# 使用するバージョンを指定する
$ nodebrew use v15.9.0

# 最後にNodeのバージョンを確認！
$ node -v
v15.9.0
```

やっとこれで再起動前と同じ状態のとこまで戻ってきました...

ためしに tsc コマンドを打ってみます。

```terminal
$ tsc --version
zsh: command not found: tsc
```

もちろんできません。
最後に TypeScript をインストールしてあげましょう！

```terminal
$ npm install typescript -g

$ tsc --version
Version 4.1.5
```

VSCode 上のターミナルでも問題なく使えました！
Zenn 使いやすいですね！よろしければフォローお願いいたします。

---

参考：[PATH を通すとは？](https://qiita.com/soarflat/items/09be6ab9cd91d366bf71)
参考： ['tsc command not found' in compiling typescript](https://stackoverflow.com/questions/39404922/tsc-command-not-found-in-compiling-typescript)
参考：[【Mac 版】node.js のアンインストールと再インストール手順メモ](https://qiita.com/wagi0716/items/94193a80502f9d81a9e0)
