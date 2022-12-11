---
title: '【Flutter】Material Theme Builder で好みの色をアプリに組み込む'
emoji: '🐬'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [flutter, dart, materialdesign]
published: true
published_at: 2022-12-15 07:00
---

:::message
🎅 本記事は [**Flutter Advent Calendar 2022**](https://qiita.com/advent-calendar/2022/flutter) 15 日目、初学者向けの記事になります 🎅
:::

https://qiita.com/advent-calendar/2022/flutter

# はじめに

個人開発をするとき、いつもデザイン適用に悩みます。とくに配色。

デザインを知らないエンジニアでも、なんとかセンスのいい配色を適用してアプリをかっこよく見せたいものです。そんな中 Flutter でよく耳にする **Material Design** について色々調べていたところ、いい感じの色を選ぶだけですぐに Flutter アプリに適用できるサービスを見つけました。

https://m3.material.io/theme-builder#/dynamic

本記事ではこちらのサービスを使い、直感的に選択した色を Flutter アプリに適用するまでを記した記事になります。

:::details 余談
つい先日に『センスは知識からはじまる』という書籍を読みました。
センスは感覚ではなく知識の集約だから、才能と決めつけず勉強しようねといった内容です。

勉強になる面白い本でした。

https://www.amazon.co.jp/%E3%82%BB%E3%83%B3%E3%82%B9%E3%81%AF%E7%9F%A5%E8%AD%98%E3%81%8B%E3%82%89%E3%81%AF%E3%81%98%E3%81%BE%E3%82%8B-%E6%B0%B4%E9%87%8E%E5%AD%A6-ebook/dp/B00LIQMVLQ/ref=tmm_kin_swatch_0?_encoding=UTF8&qid=&sr=

:::

---

## Material Design

Flutter を触っているとやたら Material Design という言葉を聞きますよね。

Material Design は 2014 年に Google が提唱したデザインシステムで、その名の通りマテリアル（＝物質的）な、現実の物質の法則に則った直感的なデザインを表現しています。

そういった概念を [Material Design のガイドライン](https://m2.material.io/design/guidelines-overview) に落とし込み、ガイドラインに準拠して用意された UI ブロックやその状態や伝達の仕組みを Material Components (MDC) というフレームワークとして提供しています。

MDC は Flutter にも提供されていて、`AppBar` や `BottomNavigationBar`、`Drawer` などの Widget を実装するだけでオシャレな UI が実現できるのはこのためです。

Material Design や Material Design for Flutter はこのあたりが参考になると思います。

- [Material Design](https://m3.material.io/)（M3）
- [Material Components widgets](https://docs.flutter.dev/development/ui/widgets/material)
- [MDC-101 Flutter:Material Components (MDC) Basics](https://codelabs.developers.google.com/codelabs/mdc-101-flutter#0)（ハンズオン）

---

Material Design の Style には Icon や Elevation、Typography などさまざまな項目がありますが、今回は『Color』に着目しています。

Material Design の Color に関する説明はこちらです。

https://m3.material.io/styles/color/overview

# TL;DR

1. [Material Theme Builder](https://m3.material.io/theme-builder#/dynamic)で好きな配色を選ぶ。
2. Export から「Flutter（Dart）」を選択する。
3. ファイルを unzip し、`color_schemes.g.dart`と`main.g.dart`を lib 下にドラッグアンドドロップする。
4. `main.g.dart`に倣って`main.dart`を書き換える。

# Material Design Builder for Flutter

テーマビルダーを適用するプロジェクトです。
デフォルトの counter app にコンポーネントを追加しただけです。

:::details main.dart

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: const MyHomePage(title: 'm3'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;
  double _currentSliderValue = 20;
  bool _light = true;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Container(
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(16),
        ),
        margin: const EdgeInsets.symmetric(horizontal: 40, vertical: 120),
        padding: const EdgeInsets.symmetric(horizontal: 40, vertical: 64),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headline4,
            ),
            const TextField(
              obscureText: true,
              decoration: InputDecoration(
                border: OutlineInputBorder(),
                labelText: 'Password',
              ),
            ),
            ElevatedButton(
                onPressed: () {}, child: const Text('ElevatedButton')),
            Slider(
              value: _currentSliderValue,
              max: 100,
              divisions: 5,
              label: _currentSliderValue.round().toString(),
              onChanged: (double value) {
                setState(() {
                  _currentSliderValue = value;
                });
              },
            ),
            Switch(
              value: _light,
              activeColor: Colors.red,
              onChanged: (bool value) {
                setState(() {
                  _light = value;
                });
              },
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
      bottomNavigationBar: NavigationBar(
        destinations: const [
          Icon(Icons.list),
          Icon(Icons.home),
          Icon(Icons.settings),
        ],
      ),
    );
  }
}
```

:::

![](https://storage.googleapis.com/zenn-user-upload/41f1f4537728-20221211.png =320x)

[Material Theme Builder](https://m3.material.io/theme-builder#/dynamic) にアクセスし、『Custom』からサイドバーの Core colors の 4 色をピッカーから選択します。

![](https://storage.googleapis.com/zenn-user-upload/4155258843af-20221211.png)
_Material Theme Builder_

## Core colors

Accent color 3 色と Neutral color 1 色を選びます。

色の役割のあれこれは [こちら](https://m3.material.io/styles/color/the-color-system/key-colors-tones) に書いてありますが、正直難しいです。これ理解して柔軟に扱えるデザイナーさんがいたらめっちゃ心強いですね。

#### Primary color

アプリの画面やコンポーネントで最も頻繁に表示される色です。

#### Secondary color

UI の中であまり目立たない部品に使われ、色の表現を広げます。
こちらはオプションで、テーマビルダーにおいても Primary color を選択すると自動で調整されます。

#### Tertiary color

Primary color と Secondary color のバランスをとったり、注目度を高めるためのアクセントとして用いたり。
自由に色を設定して製品の色彩表現の幅を広げることを目的としています。

#### Neutral color

面や背景に用いられ、テキストやアイコンを強調する色としても用いられる。

<br>

![](https://storage.googleapis.com/zenn-user-upload/4a7f8626b67a-20221211.png =320x)
_好みの色を選択する_

<br>

色を選択し終えたら右上の『Export』から Flutter（Dart）をクリックし、zip をダウンロードし解凍します。

`color_schemes.g.dart`と`main.g.dart`がフォルダに含まれているので、そのままプロジェクトの`/lib`配下に移動します。

<br>

![](https://storage.googleapis.com/zenn-user-upload/71979ae9fa6c-20221211.png)
_Flutter の他にも kotlin や CSS ファイルとしても出力できる_

<br>

この時点で`main.g.dart`は動作するものとなっており、適用した色を確認することができます。今回はすでに`main.dart`を編集しているので、`main.g.dart`の内容を真似て適用してあげます。

```diff dart
import 'package:flutter/material.dart';
+ import 'color_schemes.g.dart';
```

```diff dart
return MaterialApp(
+ theme: ThemeData(useMaterial3: true, colorScheme: lightColorScheme),
+ darkTheme: ThemeData(useMaterial3: true, colorScheme: darkColorScheme),
  home: const MyHomePage(title: 'm3'),
);
```

また、`color_schemes.g.dart`の ColorScheme class でエラーが出ているので該当箇所をコメントアウトします。

<br>

![](https://storage.googleapis.com/zenn-user-upload/a2dd009d9df8-20221211.png)
_エラー箇所をコメントアウトする_

<br>

エミュレータを再読み込みしてあげると、先程選択したテーマが反映されていると思います。

![](https://storage.googleapis.com/zenn-user-upload/f067b8196e77-20221211.png =320x)

一気に counter app のデフォルト感がなくなりましたね。

また、`useMaterial3: true`というように Material Design 3（Material Design の最新版）を適用しているので、ボタンの形状や`AppBar`の色使いなどが若干変わっています。

## ダークモード

`color_schemes.g.dart`の記述でお気づきかもしれませんが、ダークモード用の配色も用意されています。

:::details darkColorScheme

```dart
const darkColorScheme = ColorScheme(
  brightness: Brightness.dark,
  primary: Color(0xFFFFB2B9),
  onPrimary: Color(0xFF67001F),
  primaryContainer: Color(0xFF91002F),
  onPrimaryContainer: Color(0xFFFFDADC),
  secondary: Color(0xFFE5BDBF),
  onSecondary: Color(0xFF43292C),
  secondaryContainer: Color(0xFF5C3F42),
  onSecondaryContainer: Color(0xFFFFDADC),
  tertiary: Color(0xFFE8C08E),
  onTertiary: Color(0xFF442B06),
  tertiaryContainer: Color(0xFF5D411B),
  onTertiaryContainer: Color(0xFFFFDDB6),
  error: Color(0xFFFFB4AB),
  errorContainer: Color(0xFF93000A),
  onError: Color(0xFF690005),
  onErrorContainer: Color(0xFFFFDAD6),
  background: Color(0xFF201A1A),
  onBackground: Color(0xFFECE0E0),
  surface: Color(0xFF201A1A),
  onSurface: Color(0xFFECE0E0),
  surfaceVariant: Color(0xFF524344),
  onSurfaceVariant: Color(0xFFD7C1C2),
  outline: Color(0xFF9F8C8D),
  onInverseSurface: Color(0xFF201A1A),
  inverseSurface: Color(0xFFECE0E0),
  inversePrimary: Color(0xFFB61E44),
  shadow: Color(0xFF000000),
  surfaceTint: Color(0xFFFFB2B9),
  // outlineVariant: Color(0xFF524344),
  // scrim: Color(0xFF000000),
);
```

:::

<br>

`ThemeData`で読み込むカラースキームをダークモードの配色にします。

```diff dart
return MaterialApp(
- theme: ThemeData(useMaterial3: true, colorScheme: lightColorScheme),
+ theme: ThemeData(useMaterial3: true, colorScheme: darkColorScheme),
  home: const MyHomePage(title: 'm3'),
);
```

![](https://storage.googleapis.com/zenn-user-upload/7b3af60ca08a-20221211.png =320x)
_ダークモードの適用_

これだけでダークモード実装の工数がかなり省けますね。

---

適用する色のロールの変更もかんたんです。

例えば FloatingActionButton の色を アクセントカラーにしたいときは、以下のようにテーマを定義している直近の先祖（この場合 MyApp の MaterialApp）のテーマデータを取得し、その中の`tertiaryContainer`の色を適用してあげます。

```diff dart
floatingActionButton: FloatingActionButton(
  onPressed: _incrementCounter,
  tooltip: 'Increment',
+ backgroundColor: Theme.of(context).colorScheme.tertiaryContainer,
  child: const Icon(Icons.add),
),
```

![](https://storage.googleapis.com/zenn-user-upload/9b5f1712b61b-20221211.png =320x)
_FAB の色を変更_

# おわりに

はじめてアドベントカレンダーに参加してみましたが、同じ技術を扱う開発者の方との一体感を感じられる良いイベントですね。

Flutter を触り始めてから半年ほど経ちますが、個人開発やコミュニティ（主に Twitter）の盛り上がり、多くのイベント開催など触れていて本当に楽しい技術です。来年はもっとコミュニティの発展や個人開発に力を入れたいなぁ。

zenn 記事もまだまだ書いていく予定ですので、今後もよろしくお願いします。
よいクリスマス、よいお年を！

## 参考

- [Material Design](https://m3.material.io/)
- [Material Components widgets](https://docs.flutter.dev/development/ui/widgets/material)
