---
layout: post
title: Xcodeサーバーでの継続的インテグレーション
---

継続的インテグレーション、CI（continuous integration）といえばGithub Actions、Bitrise、Jenkinsなどのサードパーティサービスやツールが浮かびますよね。実際にXcodeを使うだけで、簡単にできます。

まずはCIの実行環境として使うMacからXcode Serverを立ち上げます。

![Xcode Serverの立ち上げ](/assets/images/turning-on-xcode-server.png)

Xcode > PreferencesのServers & Botsの右上のスイッチをオンにして、CIを実装するユーザーを選んだら、Xcode Serverの立ち上げが完了しました。

そして開発用のMacからXcode Serverの接続設定をします。

![Xcode Serverへの接続設定](/assets/images/adding-xcode-server.png)

Xcode > PreferencesのAccountsから追加します。

最後は実際にCIを実行するボットを設定します。

Xcodeでプロジェクトを開いた状態で、Product > Create Bot...を選びます。

リポジトリーのアクセス権限が確認され、プライベートリポジトリーの場合なぜか下記のエラーが出てきます。

![SSH指紋検証失敗](/assets/images/ssh-fingerprint-failed-to-verify-when-creating-bot.png)

私の環境がおかしいかもしれませんが、ViewボタンをクリックしてCancelすればアクセスの確認が再開し、今度は正常な認証失敗になります。

![認証失敗](/assets/images/credentials-missing-when-creating-bot.png)

Sign In...ボタンをクリックして認証情報を提供すれば、リポジトリーアクセス権限の確認できます[^1]。

CIを実行するブランチの選択、build configurationの設定とインテグレーションの設定を経て、テストを実行するデバイスの設定になります。

![テストを実行するデバイスの設定](/assets/images/devices-for-bot-to-test-with.png)

全てのデバイスで同時にテストすることは可能ですが、XcodeはDepolyment Targetにより、実行できないデバイスを排除してくれないので、Specific iOS Devices and Simulatorsを選択し、実行可能なデバイスのみ選択する必要があります。

続きは署名とプロビジョニングの設定で、simulatorのみであれば不要になります。

xcodebuildの追加プロパティーと環境変数の設定を経て、triggersの設定になります。XcodeGenやCocoapodsなどを使ってビルドする前にスクリプトを実装する必要がある場合、ここで追加できます。

XcodeGenでは下記のPre-Intergration Scriptを追加する必要があります。

```sh
# スクリプトがbotのルートから実行されるため、プロジェクト配下に移動
cd ProjectName
# 多数のツールを使う場合exportしてからでも良い
/usr/local/bin/xcodegen　
```

これでbotの作成が完了で、初回のインテグレーションが始めます。今後設定された条件（間隔やコミット）で自動にインテグレーションが始めます。

Xcode上の操作を加え、[API](https://developer.app)もありますので、ある程度の自動化カスタマイズが可能だと思います。

## 結論

Xcodeサーバーでの継続的インテグレーションの設定は極めて簡単で、すぐできます。機能はそこまで多くないけど、インテグレーション前後スクリプトを実行することが可能ですので、大体の重要が満足でき、わりといい選択肢です。余分なMacがあればぜひ試してみてください。

## 参考

[Continuous integration using Xcode Server](https://help.apple.com/xcode/mac/11.4/index.html?localePath=en.lproj#/dev466720061)

***

[^1]: ここはあくまでの確認で、実際の認証設定は実際にCIを実行するMacで設定する必要があります。
