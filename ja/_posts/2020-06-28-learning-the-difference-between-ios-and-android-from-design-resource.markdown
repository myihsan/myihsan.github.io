---
layout: post
title: デザインリソースからiOSとAndroidの標準や違いを学ぶ
---

せっかくデザインしたものがエンジニアに「実装コストが高いので、標準部品で実装する」言われることが、モバイルアプリデザインするデザイナーさん誰でも経験したことがあると思います。

コストのため仕方がありませんが、実に残念なことでしょう。そうならないように最初からOSを考慮した上でデザインしていきましょう。

と言われても、自分のデバイスはiOSとAndroidのどれかひとつで、片方の常識があるとは言え、もう片方は全然わからない場合が多いと思います。[Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)や [Material Design](https://material.io/)を読むべきかもしれませんが、量が多いし、抽象的でわかりにくいところもあるし、具体的でややこし過ぎるところもあります。

公式が提供しているデザインリソース（[iOS](https://developer.apple.com/design/resources/) \| [Android](https://material.io/resources)）があり、中の部品は大体実装コストが低い標準部品[^1]ですので、OSの常識を把握するには適切だと思います。

では早速見ていきましょう。

## トップバー

| ![iOS（Navigation Bar](/assets/images/navigation-bar.png) | ![Android（App Bar）](/assets/images/app-bar-top.png) |
|:-:|:-:|
| iOS（Navigation Bar） | Android（App Bar） |

### 背景色

iOS側は固定な色で、Android側はメインな色になるのが普通ということがわかります。とは言え色だけですので、実装の配慮する必要がありません。デザイン上の必要がなければ、普通の方がユーザーに対して違和感が少ないです。

### タイトルの位置

iOS側は中央揃えで、Android側は左寄せです。タイトル自体があまり変わりがありませんが、Android側の左のボタンはひとつまでです。

## ボトムバー

| ![iOS（Toolbar）](/assets/images/toolbar.png) | ![Android（App Bar）](/assets/images/app-bar-bottom.png) |
|:-:|:-:|
| iOS（Toolbar） | Android（App Bar） |

背景色は[トップバー](#背景色)と同じですので、省略します。

### ボタンの比重

iOS側のボタンの比重が統一で、Android側にfloating action button（FAB）というMaterial Design特有なボタンをひとつまで嵌めることが可能で、違う比重が表せます。

## ボタン

| ![iOS text button](/assets/images/ios-button-text.png) | ![Android text button](/assets/images/android-button-text.png)
| ![iOS icon button](/assets/images/ios-button-icon.png) | ![Android icon text Button](/assets/images/android-button-icon-text.png) |
| ![Add to Siri button](/assets/images/add-to-siri.png) | ![Android outlined button](/assets/images/android-button-outlined.png) |
| ![Added to Siri button](/assets/images/added-to-siri.png) | ![Android Contrained Button](/assets/images/android-button-contrained.png) |
| ![Sign in with Apple button](/assets/images/sign-in-with-apple.png) | ![Toggle](/assets/images/toggle.png) |
| ![Pay with Apple Pay button](/assets/images/pay-with-apple-pay.png) | ![Floating action button](/assets/images/floating-action-button.png) |
|:-:|:-:|
| iOS | Android |

Android側の多様なスタイルと比べ、iOS側は二種類しかなく、残ったのは専用ボタンです。ボタンなんてカスタマイズしてもコストあまりかからないと思われがちですが、非活性や押さている状態のデザインやアクセシビリティも考慮しなければなりませんので、デザイン上の必要がなければ、標準の方が無難です。

## アラート、アクションシート｜ダイアログ

| ![Alert](/assets/images/alert.png) | ![Dialog](/assets/images/dialog.png) |
| ![Three options alert](/assets/images/alert-three-options.png) | ![No title dialog](/assets/images/dialog-no-title.png) |
| ![Text field alert](/assets/images/alert-text-field.png) | |
| ![Action sheet](/assets/images/action-sheet.png) | |
|:-:|:-:|
| iOS | Android |

iOS側はアラートとボトムシート二種類があり、ボタンがデフォルト以外におすすめ、破壊的、非活性のスタイルがあります。それに対して、Android側はあまりバリエーションがないように見えますが、リソースに入っていないだけです。[こちら](https://material.io/components/dialogs#usage)を参考してみればわかると思いますが、結構バリエーションがあります[^2]。

## テキストフィールド

| ![iOS text field](/assets/images/ios-text-field.png) | ![Android text field](/assets/images/android-text-field.png) |
|  | ![Android text field with helper text](/assets/images/android-text-field-helper-text.png) |
|  | ![Android text field with error](/assets/images/android-text-field-error.png) |
|  | ![Android outlined text field](/assets/images/android-text-field-outlined.png) |
|:-:|:-:|
| iOS | Android |

iOS側はクリアボタンしかありません。そのかわりにAndroid側はテキストフィールドによくあるユースケースをほぼ網羅できています。エラー表示について実装コストを考慮してiOS側はアラートを採用しがちですが、Android側はテキストフィールドを活用しましょう。

## スライダー

| ![iOS slider](/assets/images/ios-slider.png) | ![Android slider](/assets/images/android-slider.png) |
| | ![Android discret slider](/assets/images/android-slider-discret.png) |
| | ![Android slider with value indicator](/assets/images/android-slider-value-indicator.png) |
|:-:|:-:|
| iOS | Android |

iOS側はただのスライダーで、それ以上は全部自前で実装する必要があります。それに比べ、Android側は目盛りと現在の値を表示できます。

## セグメンテッドコントロール｜タブ

| ![iOS segmented control](/assets/images/segmented-control.png) | ![Android tabs](/assets/images/android-tabs.png) |
| | ![Android scrollable tabs](/assets/images/android-tabs-scrollable.png) |
|:-:|:-:|
| iOS（Segmented Control） | Android（Tabs） |

ボトムナビゲーションと似たような機能です。元々Android側はボトムナビゲーションが非推奨でしたので、タブ（iOSにおいてボトムナビゲーションを指す）というややこしい名前になりました。

iOS側とAndroid側の標準なデザインの違いが少なくありませんので、デザインのよるとカスタマイズしにくいところが出るかもしれません。こだわりがなければデフォルトでいきましょう。

また、iOS側のセグメンテッドコントロールはスクロールできないため、ボトムナビゲーションのように五つまでにしましょう。

## プルダウンメニュー｜ドロップダウンメニュ

| ![iOS pull-down menu](/assets/images/pull-down-menu.png) | ![Android dropdown menu](/assets/images/dropdown-menu.png) |
|:-:|:-:|
| iOS 14 | Android |

Android側に早期からありまして、現在に至ってテキストフィールドと一緒に使うことも可能に比べ、iOS側はiOS 9から似たような機能（Peek and Pop）がリリースされ、iOS 13でリニューアル（[Context Menus](https://developer.apple.com/design/human-interface-guidelines/ios/controls/context-menus/)）されましたが、詳細画面のプレビュー（コンテキスト）と伴う部品でした。

iOS 14からついにプレビューなしで使えるようになりました。

## 日時ピッカー

| ![iOS date picker](/assets/images/ios-date-picker.png) | なし |
|:-:|:-:|
| iOS 14 | Android |

Android側は結構前から使いやすい日時ピッパーがあります[^3] [^4]。使うのは当たり前すぎて、デザインに記載する必要がない理由で、リソースに載っていないと個人的に思います。

iOS 13以前ではローラーのようなピッカーしかなくて、使い勝手はあまり良くありません。iOS 14からついにAndroid側と似たような日時ピッパーが出てきました。とはいえ、Android側と比べるとタップと入力の切り替えはできませんし、日時の範囲を選択することもできません。

日時ピッカーはOSの所々に使われる部品ですので、例えより優秀なデザインがあっても、ユーザーの習慣が少しでも違いがあると、逆に使いにくい恐れがあります。デザインを統一するために、Android側もローラーのようなピッカーにするような習慣の違いはともかく、使い勝手を低下することはやめましょう。

## OSバージョン

すでに気づいたかもしれませんが、公式リソースのダウンロードページもこの記事もiOSのバージョンのみ記載してあります。Android側バージョンの問題を気にしなくても大丈夫でしょうか？

大体の場合は大丈夫です。iOS側のデザイン変更は主にバージョンと共に進んでいますので、配慮しなければなりませんが、Material DesignはAndroid専属なデザインガイドラインではありませんので、Androidの進化と共にデザインが更新されるというより、公式がMaterial DesignをAndroidに実装しています。普通の開発と同じ、古いバージョンをサポートしています。

とはいえ、公式だからと言って全バージョンバグなし完全に同じ動きになる保証がありませんし、古すぎるバージョンを切り捨てる場合もありますので、何か不具合を遭遇した場合、エンジニアと回避策を考えましょう。

## Android特有

これからはAndroid特有な部品に入ります。

### 選択ボタン

| ![Radio button](/assets/images/radio-button.png) | ![Checkbox](/assets/images/checkbox.png) |
|:-:|:-:|
| Radio Button | Checkbox |

単一と複数選択の場合よくあるデザインと思いますが、iOS側に存在しません。

| ![iOS selection list](/assets/images/ios-list-selection.png) | ![Android selection list](/assets/images/android-list-selection.png) |
|:-:|:-:|
| iOS | Android |

リストの場合はチェックマークで表せますが、規約を同意する意味合いを持つチェックボックスを直接表せる部品がありません。

選択ボタン自体が二つ状態しかなく、一応二つ状態のリソースを準備してあげれば簡単[^5]に実装できます。

### スナックバー

| ![Snackbar](/assets/images/snackbar.png) |
| ![Snackbar with action](/assets/images/snackbar-action.png) |
|:-:|
| Snackbar |

ユーザーのアクションは必須ではない場合、メッセージを出す部品になります。iOS側がありませんので、[自前で作る]({% post_url 2020-04-12-make-a-heads-up-display-like-the-one-in-books-app %})かアラートを使うかになりますが、Android側にせっかくありますので、積極的に使いましょう。

## まとめ

プログラミングがわからなくても、公式のデザインリソースから見てiOSとAndroidの標準や違いいろいろわかりますね。上記に書いてあるのはあくまで一部で、今後デザインリソースの更新につれ、また新たな情報が出てくるのでしょう。

このような情報を生かし、実装コストが低い、各OSの本領が発揮でき、デザイナー、エンジニア、ユーザー全部喜ぶデザインを作りましょう。

***

[^1]: 全部ではありません。特にAndroidの方はMaterial DesignでAndroid専用ではありません。

[^2]: 実装の知識がないとわからない情報ですが、Android側はタイトルとボタンのアウトにカスタムビューを入れることが可能で、タイトルとボタンを表示しないことも可能ですので、コストが増えますが、なんでもできます。しかしiOS側はほぼできません。

[^3]: [Date pickers](https://material.io/components/date-pickers)

[^4]: [Time pickers](https://material.io/components/time-pickers)

[^5]: Android側の部品のように選択をグループして、アニメーションもつけるにはそれなりに大変です。