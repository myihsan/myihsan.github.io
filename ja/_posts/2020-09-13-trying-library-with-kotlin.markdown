---
layout: post
title: Kotlinでのライブラリー試用
---

Swiftは`swift package init --type executable`だけで、簡単にライブラリーを使ってみることは可能ですが、Kotlinでは似たようなことができるかを調べてみました。自分はAndroidでしかKotlinを使ったことがないし、Gradleもあまり詳しくないので、最初は全然ピンと来ませんでしたが、実際に簡単です。

## IntelliJ IDEA

Android StudioはAndroidプロジェクトに特化したバージョンですので、単純にKotlinでライブラリーを試したい場合、IntelliJ IDEAを[ダウンロード](https://www.jetbrains.com/idea/download)する必要があります。Community Editionでも大丈夫です。

## 新規プロジェクト

| ![新規Kotlinプロジェクト](/assets/images/new-kotlin-project.png) |
|:-:|
| 新規Kotlinプロジェクト |

プラットフォームにより適当なテンプレートを選ぶ必要がありますが、JVMでよければ「Console Application」を選ぶのは一番便利です。

## ライブラリー試用

「Console Application」テンプレートであれば、`./src/main/kotlin/main.kt`が生成されて、ライブラリーのREADME通りにGradleで依存を設定した上で、`fun main()`でライブラリーを試せます。

とはいえ、なぜかConfigurationが設定されていなくて、Runボタンが押せません。自分でConfigurationを追加すればいいですが、`fun main()`の左にあるRunボタンが押せば、`fun main()`を実行するConfigurationが自動に生成されます。

| ![`fun main()`の左にあるRunボタン](/assets/images/run-button-beside-main-funciton.png) |
|:-:|
| `fun main()`の左にあるRunボタン |

## まとめ

IntelliJ IDEAのテンプレートだけで、簡単ですよね。

Android開発に何かのライブラリーを導入したい場合、Androidプロジェクトでemulatorで実行するより、IntelliJ IDEAで気楽に試しましょう。
