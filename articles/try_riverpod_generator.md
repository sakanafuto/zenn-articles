---
title: '【Flutter】Riverpod 2.0 の Notifier と riverpod_generator の解説'
emoji: '🐬'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, provider, riverpod, notifier]
published: true
---

# はじめに

[**Riverpod 2.0**](https://pub.dev/packages/riverpod) がちょっと前に来ました。それに合わせてリリースされた [**riverpod_generator**](https://pub.dev/packages/riverpod_generator) と、あまり注目されていない（?） `StateProvider` 及び `StateNotifier` の代替手段となる **`Notifier`** についてまとめてみました。

## 動作環境

| 名前                | バージョン |
| ------------------- | ---------- |
| Flutter             | 3.3.8      |
| hooks_riverpod      | ^2.0.0     |
| riverpod_annotation | ^1.0.6     |
| riverpod_generator  | ^1.0.6     |
| build_runner        | ^2.2.1     |

# riverpod_generator

`Riverpod` の自動生成ができるパッケージです。実装したい項目をふわっと書いて、`@riverpod` のアノテーションを付けたらいつもの `build_runner` くんが自動生成してくれます。

https://pub.dev/packages/riverpod_generator

適当な例を試してみます。必要なパッケージは次の通り。

- `$ flutter pub add riverpod riverpod_annotation`
- `$ flutter pub add build_runner riverpod_generator -d`

カウンターアプリでを例に解説したいので、とりあえず数字を監視して画面に表示させてみます。

#### いつもの手書きの場合

```dart
final CounterProvider = Provider<int>((ref) => 0);

...
@override
Widget build(BuildContext context, WidgetRef ref) {
    final sampleCounter = ref.watch(counterProvider);

    return Center(child: Text(sampleCounter.toString()));
...
```

結果

![](https://storage.googleapis.com/zenn-user-upload/7aa1800bf5a6-20221110.png)

---

#### riverpod_generator の場合

```dart:counter_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

@riverpod
int counter(CounterRef ref) => 0;
```

『3 ステップ + 自動生成』でよしなに `Provider` だったり `FutureProvider` を作ってくれます。

1. `riverpod_annotation` を import する。
1. `part <ファイル名>.g.dart;` する。
1. `@riverpod` アノテーションを指定し、**やりたいこと**を書く。

**やりたいこと**は目的によって書き方が異なりますので各自ご確認ください。今回は簡単なユースケースなので 1 行で済んじゃってます。返り値に監視したい State の型を宣言し、初期値をリターンするのがポイントです。

関数の定義で `counter` という文字列が出てきていますが、これはライブラリ特有の書き方です。何となく分かると思いますが、自動生成で名前を解決してくれて、 `xxxProvider` というプロバイダが定義されるよというものです。引数の `ref` の型も `XxxRef` としてあげれば OK です。

---

<!-- **自動生成** -->

`$ flutter pub run build_runner watch --delete-conflicting-outputs` で自動生成します。

`watch` オプションを指定することによって Freezed の際のような待ち時間もなく、変更があるたびに結構スムーズに生成ファイルを生成・変更してくれます。

```dart:counter_provider.g.dart
...

/// See also [counter].
final counterProvider = AutoDisposeProvider<int>(
  counter,
  name: r'counterProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : $counterHash,
);
typedef CounterRef = AutoDisposeProviderRef<int>;
```

`final counterProvider = ...` とプロバイダを定義してくれていますね。
ちなみに `riverpod_generator` で自動生成されるプロバイダはすべて autoDispose 修飾子がつくようです。

https://riverpod.dev/ja/docs/concepts/modifiers/auto_dispose/

そして最終行で `AutoDisposeProviderRef<int>` として `CounterRef` 型を定義してくれています。これによって、定義ファイルさえ読み込んでいれば `counterProvider` が使えるようになっているんですね。

```dart:counter_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

@riverpod
int counter(CounterRef ref) => 0;
```

```dart: counter_screen.dart
import 'package:flutter/material.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';

class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final sampleCounter = ref.watch(counterProvider);

    return Center(
      child: Text(sampleCounter.toString()),
    );
  }
}
```

このように定義して自動生成を行い、別ファイルから `Counter` を呼び出してあげるとちゃんと `counterProvider` として値を取得できました。

結果

![](https://storage.googleapis.com/zenn-user-upload/7aa1800bf5a6-20221110.png)

# Notifier

続いて、**Riverpod 2.0** 新要素の **Notifier** について触れておきます。

**Notifier** は **Riverpod 2.0** で新しく追加されたもので、`StateProvider` と `StateNotifierProvider` の両方の挙動をカバーできるようなプロバイダとなっています。

先程の例はカウンターアプリといいつつ `Provider` で 0 を監視しているだけでしたが、本来のカウンターアプリを `Riverpod` で再現しようとすると以下のようなります。

```dart
final counterProvider = StateProvider<int>((ref) => 0);

class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final sampleCounter = ref.watch(counterProvider);
    return Center(
      child: ElevatedButton(
        child: Text(sampleCounter.toString()),
        onPressed: () => ref.read(counterProvider.notifier).state++,
      ),
    );
  }
}
```

単純な数値の加算減算や、Enum の切り替えなどをする分には `StateProvider` を用いて、複雑な実装をしたい場合には `StateNotifier` を使うという運用が多かったと思いますが、今後は `Notifier`（と `AsyncNotifier`） 推奨になります。

`Notifier` を継承したクラスを作成することで、`StateNotifier` のようにより柔軟な State の監視が可能になります。

```dart
class Counter extends Notifier<int> {
  @override
  int build() => 0;

  // オプション
  void increment() => state++;
}
```

`Riverpod` に慣れ親しんで来た人はわかると思いますが、直感的で使いやすいです。。。
build メソッドをオーバーライドして初期値を `StateProvider` のように設定し、必要に応じて `StateNotifier` のように関数を定義します。

そしてプロバイダを作成します。どちらの書き方でも結構です。
そうすれば、先程の `CounterScreen` のコードで同じように動作します。

```dart
final counterProvider = NotifierProvider<Counter, int>(() => Counter());
// or
final counterProvider = NotifierProvider<Counter, int>(Counter.new);
```

定義した `increment` メソッドを適用してあげてもいいです。

```diff dart
+ onPressed: () => ref.read(counterProvider.notifier).increment(),
- onPressed: () => ref.read(counterProvider.notifier).state++,
```

結果

![](https://storage.googleapis.com/zenn-user-upload/329d5bab345d-20221110.gif)

# Notifier を generate

とても便利な `Notifier` ですが、`riverpod_generator` の自動生成の対象となっています。

書き方も基本的には記事の最初に書いた方法に倣うだけで、気をつける部分は `Notifier<int>` ではなく `_$Counter` を継承しているところでしょうか。 `Freezed` や `Hive` などで似たような書き方をしますね。

```dart:counter_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

:::details 自動生成ファイルの中身

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'counter_provider.dart';

// **************************************************************************
// RiverpodGenerator
// **************************************************************************

// ignore_for_file: avoid_private_typedef_functions, non_constant_identifier_names, subtype_of_sealed_class, invalid_use_of_internal_member, unused_element, constant_identifier_names, unnecessary_raw_strings, library_private_types_in_public_api

/// Copied from Dart SDK
class _SystemHash {
  _SystemHash._();

  static int combine(int hash, int value) {
    // ignore: parameter_assignments
    hash = 0x1fffffff & (hash + value);
    // ignore: parameter_assignments
    hash = 0x1fffffff & (hash + ((0x0007ffff & hash) << 10));
    return hash ^ (hash >> 6);
  }

  static int finish(int hash) {
    // ignore: parameter_assignments
    hash = 0x1fffffff & (hash + ((0x03ffffff & hash) << 3));
    // ignore: parameter_assignments
    hash = hash ^ (hash >> 11);
    return 0x1fffffff & (hash + ((0x00003fff & hash) << 15));
  }
}

String $CounterHash() => r'4243b34530f53accfd9014a9f0e316fe304ada3e';

/// See also [Counter].
final counterProvider = AutoDisposeNotifierProvider<Counter, int>(
  Counter.new,
  name: r'counterProvider',
  debugGetCreateSourceHash:
      const bool.fromEnvironment('dart.vm.product') ? null : $CounterHash,
);
typedef CounterRef = AutoDisposeNotifierProviderRef<int>;

abstract class _$Counter extends AutoDisposeNotifier<int> {
  @override
  int build();
}
```

:::

グローバルに宣言するプロバイダ、個人的にどこで作成するか迷うことが多いのですが、自動生成を使うと生成ファイル内でプロバイダを作成してくれるのでこれが意外とありがたい。

ちなみに、定義ファイルの時点ではクラスがどんな State プロパティを持つか書いていませんが、生成ファイルでは `AutoDisposeNotifierProviderRef<int>` と、`int型`であることがわかっていますね。これは `build` メソッドの返り値をみて判断しているからなのだそう。

# まとめ

見ての通り `riverpod_generator` は便利ではありますが、まだ旨味をあまり享受できてないように感じます。新しい技術ですし、今後もっと機能が追加されていくことを願います。

`Notifier` については、今回の例だと `StateProvider` のほうが簡潔に書けてしまい良さをあまりお伝えできなかったかもしれません。`StateProvider` でできることは `Notifier` でカバーでき、更に柔軟性も高いというイメージを持っていただけたら結構です。

更にもう 1 つ、Riverpod 2.0 で追加された非同期的な State を扱える `AsyncNotifier` を学べば`StateNotifier` / `StateNotifierProvider` も完全に置き換えられるようになります。

`AsyncNotifier` は次回あたりで解説できたらなと思います。

`Flutter`、本当に技術の進化が早くてびっくりします。
キャッチアップのためにも Twitter で色々発信していますので、よければフォローお願いします 🙌

# 参考

- [riverpod_generator: ^1.0.6](https://pub.dev/packages/riverpod_generator)
- [riverpod: ^2.1.1](https://pub.dev/packages/riverpod)
- [How to use Notifier and AsyncNotifier with the new Flutter Riverpod Generator](https://codewithandrea.com/articles/flutter-riverpod-async-notifier/)
