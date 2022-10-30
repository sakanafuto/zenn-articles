---
title: '【Flutter】Linter for Dart の日本語訳チートシート'
emoji: '🐬'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [flutter, dart]
published: false
---

# はじめに

前回の記事で Flutter におけるの Formatter や Linter について触れ、`analysis_options.yaml` に [pedantic_mono](https://github.com/mono0926/pedantic_mono) を include し、適宜自分にあうよう設定していくと結論付けていました。

https://zenn.dev/10_tofu_01/articles/flutter_formatter_linter_lefthook

が、よくよく考えてみるといくら著名な方の設定だとしても一個人の設定ファイルをそのまんま使うのもどうなのかということと、なにより個人的に Linter ルール等の全貌が見えていないことが少し気持ち悪く感じていました。

ということできちんと [Linter ルール](https://dart-lang.github.io/linter/lints/) 一読した上でもう一度考え直してみました。
そこでチートシートが出来上がった次第です。

# チートシート

そんなこんなで出来上がったシートがこちらです。

https://docs.google.com/spreadsheets/d/1YtpfQX2F--b8Gnp4yiBT5WbHih59fDk2zmAWMZgjgYQ/edit?usp=sharing

出来上がったと言っても、[こちら](https://rydmike.com/blog_flutter_linting.html#what-are-the-rule-differences-between-all-these-packages)の神スプレッドシートをお借りして DeepL API でシートを翻訳して整形したものになります。

内容はできるだけ更新していくつもりです。
現時点で 2022/10/7 更新（Flutter 3.3, Dart 2.18 を含む）の内容のものとなっています。

シートには [Linter for Dart](https://dart-lang.github.io/linter/lints/) の各ルールのかんたんな説明と、下記のパッケージのルール適用状況やパッケージ間の比較を記載しています。

`$ dart fix` で自動修正してくれるかどうかも記載しています。

# パッケージの説明

有名な Lint パッケージをいくつか紹介します。

## [Dart lints](https://pub.dev/packages/lints)

こちらは Dart プロジェクトのための公式推奨の パッケージです。Core lints と呼ばれる最小限の Lint ルールセットと、Recommended Lints と呼ばれるより広範なセットの 2 種類が用意されています。最小限といった感じで、Lint ルールを作るためのルールといったものでしょうか。

- [Core lints](https://github.com/dart-lang/lints/blob/main/lib/core.yaml)
- [Recommended lints](https://github.com/dart-lang/lints/blob/main/lib/recommended.yaml)（Core lints のすべてのルールを含んでいます）

## [flutter_lints](https://pub.dev/packages/flutter_lints)

ライトに Flutter を開発する方はこちらで問題ないでしょう。
Flutter 2.5.0 以降を使用して `flutter create` で作成されたプロジェクトは、自動的に Flutter lints パッケージを使用するように設定されています。

上記の Dart lints の Reccomended lints ルールに加えて、Flutter プロジェクトで重要ないくつかのルールが含まれています。
すべての Lint ルールの内、47.8%を適用しているというちょうど良さもいいですね。

## [lint](https://pub.dev/packages/lint)

海外製のより厳しいルールが定められたパッケージです。ルール適用率は約 72%！
特徴としてはソースコードに各ルールを適用する理由が記載されていることです。明確な根拠があって適用しているのがよくわかります。

Lint ルールは厳しくすればするほど開発を阻害する場面もでてきますが、曖昧なまま書けてしまいあとから痛い目を見るということを防ぐためにも厳し目がいいと個人的には感じています。こちらも、この次に紹介するパッケージのどちらも一つ前の flutter_lints のルールを完全に含んでいるので、公式推奨のものから逸脱することもありません。

## [pedantic_mono](https://pub.dev/packages/pedantic_mono)

こちらは日本人の Flutter コントリビューター [mono 氏](https://mono0926.notion.site/mono0926/mono-Masayuki-Ono-8959cd3bd40142e19ec433c5cd5b449b) の公開する Lint パッケージです。
ルール適用率は驚異の 83%。

内容に関しては、ご本人による記事を読むのが早いでしょう。

https://medium.com/flutter-jp/analysis-b8dbb19d3978

# まとめ

最後にわたしの`analysis_options.yaml`を載せておきます。flutter_lints を include して公式に準拠しつつ、lint と pedantic_mono リスペクトのマゾ仕様です。といっても開発中に迷惑と感じたことはほぼありませんし、指摘からかなり勉強になることが多いです。

:::details analysis_options.yaml

@[gist](https://gist.github.com/sakanafuto/604ecafa73f6c5dbc72a922d09d7ccd2)

https://gist.github.com/sakanafuto/604ecafa73f6c5dbc72a922d09d7ccd2
:::

あ、IDE のクイックフィックスは必須です。VS Code なら `⌘ + .` で選択できます。

![](https://storage.googleapis.com/zenn-user-upload/37b67ff8f7b0-20221030.gif)
_クイックフィックス in VS Code_

皆さんの参考になれば幸いです。
