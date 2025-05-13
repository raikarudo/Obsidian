---
title: "【Xcode】無料の実機ビルドでどこまでできるのか"
source: "https://qiita.com/koogawa/items/15b231e2728ff64e08f3"
author:
  - "[[Qiita]]"
published: 2019-08-25
created: 2025-05-14
description: "これは何ご存知の通り、Xcode 7から開発中のアプリを無料で実機にインストールできるようになりました。それまでは年間11,800円の登録費用を払って、Developer Programに登録する…"
tags:
  - "clippings"
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=)

## Qiitaにログインして、便利な機能を使ってみませんか？

[ログイン](https://qiita.com/login?callback_action=login_or_signup&redirect_to=%2Fkoogawa%2Fitems%2F15b231e2728ff64e08f3&realm=qiita) [新規登録](https://qiita.com/signup?callback_action=login_or_signup&redirect_to=%2Fkoogawa%2Fitems%2F15b231e2728ff64e08f3&realm=qiita)

この記事は最終更新日から5年以上が経過しています。

## これは何

ご存知の通り、Xcode 7から開発中のアプリを無料で実機にインストールできるようになりました。それまでは年間11,800円の登録費用を払って、Developer Programに登録する必要がありました。  
この記事では、無料で実機ビルドした場合の制限事項などをまとめます。

なお、無料で実機ビルドする方法については、下記記事が参考になります。  
[\[Xcode\]\[iOS\] 有料ライセンスなしでの実機インストール 全工程解説！ ｜ Developers.IO](http://dev.classmethod.jp/smartphone/run-on-devices-without-apple-developer-program-license/)

- 2016/8/6 時点での仕様になります
- Apple Developer Program に登録済みのアカウントを「有料アカウント」と表記します
- Apple Developer Program に登録していないアカウントを「無料アカウント」と表記します

## 管理画面の違い

まずはDeveloperサイト上で操作できる内容を比較してみます。

## Apple Developerサイトのメニュー

有料アカウントでは Certificates などが管理できるのに対し、無料アカウントではそれらの項目が見当たりません。

▼有料アカウント  
[![スクリーンショット 2016-08-06 9.39.41.png](https://qiita-image-store.s3.amazonaws.com/0/8292/9bd381c4-9916-b3a8-a6ab-23e79ecf0a95.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F8292%2F9bd381c4-9916-b3a8-a6ab-23e79ecf0a95.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2e0f611f3ec6c7773d18baadfc070c76)

▼無料アカウント  
[![スクリーンショット 2016-08-06 9.39.14.png](https://qiita-image-store.s3.amazonaws.com/0/8292/3cee6fd0-94ee-ccd5-360a-4e2a03865358.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F8292%2F3cee6fd0-94ee-ccd5-360a-4e2a03865358.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=54fe04ccea75ae8163bb400782ea1387)

## iTunes Connect

無料アカウントではアプリの管理画面にはアクセスできません。つまり、アプリをApp Storeに公開することはできません。もちろん、In App PurchaseやTestFlight の管理もできません。

▼この画面に飛ばされます  
[![image](https://qiita-image-store.s3.amazonaws.com/0/8292/22301f0c-b3e5-46e7-53cf-29c4199049ea.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F8292%2F22301f0c-b3e5-46e7-53cf-29c4199049ea.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a33232069d72e60b506297f84daf8452)

## 制限事項

次は機能単位での制限事項について調べてみます。

## 1\. Push Notificationsなどが使えない

無料アカウントでは多くのサービスが使えません。

使用できないサービスの例：

- Apple Pay
- Game Center
- iCloud
- In-App Purchasing
- Push Notifications
- Wallet

無料アカウントで使えるサービスはXcodeプロジェクトのCapabilities画面を開くことで確認できます。

[![image](https://qiita-image-store.s3.amazonaws.com/0/8292/4e59b16e-5002-bd1d-6fcd-63530dd14363.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F8292%2F4e59b16e-5002-bd1d-6fcd-63530dd14363.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=44480f9fef7ad1830378359a6b3159c1)

詳細はAppleの公式ドキュメントを参照してください。

- [Supported Capabilities](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/SupportedCapabilities/SupportedCapabilities.html#//apple_ref/doc/uid/TP40012582-CH38-SW1)

## 2\. 有効期限は7日

プロビジョニングプロファイルの有効期限は 7 日のようです。（最近までは90日だったようです）

```xml
<key>TimeToLive</key>
<integer>7</integer>
```

試しにプロビジョニングプロファイルを作ってみたところ、有効期限が7日後の8/13に設定されていました。（今日は8/6）

[![image](https://qiita-image-store.s3.amazonaws.com/0/8292/eff9d7f5-d9f3-d492-a263-97c1acc3476c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F8292%2Feff9d7f5-d9f3-d492-a263-97c1acc3476c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=57d46e66172a4cd8265add744ad3ece3)

## 3\. 登録できるデバイス数は不明

登録済みのデバイスは `ProvisionedDevices` のリストに追加されます。何台まで登録できるかはわかっていません。（ご存知の方おりましたら教えて下さい）

**追記：** 2018年11月現在で 4台 まではインストールできた、という情報をいただきました。 [@junpluse](https://qiita.com/junpluse "junpluse") さんありがとうございます！

```xml
<key>ProvisionedDevices</key>
    <array>
        <string>XXXX</string>
    </array>
```

## 4\. TestFlightのInternal/Externalテストは使えない

iTunes Connect の管理画面がそもそもないので、TestFlight は使用できません。

## 参考

## 無料アカウントで生成されるプロビジョニングプロファイルの中身

`LocalProvision` というキーが追加されるようです。

※一部はマスクしております

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>AppIDName</key>
    <string>XC Wildcard</string>
    <key>ApplicationIdentifierPrefix</key>
    <array>
    <string>XXXX</string>
    </array>
    <key>CreationDate</key>
    <date>2016-08-05T15:22:33Z</date>
    <key>Platform</key>
    <array>
        <string>iOS</string>
    </array>
    <key>DeveloperCertificates</key>
    <array>
        <data>XXXX</data>
    </array>
    <key>Entitlements</key>
    <dict>
        <key>keychain-access-groups</key>
        <array>
            <string>XXXX.*</string>        
        </array>
        <key>get-task-allow</key>
        <true/>
        <key>application-identifier</key>
        <string>XXXX.com.example.PersonalTeam</string>
        <key>com.apple.developer.team-identifier</key>
        <string>XXXX</string>
    </dict>
    <key>ExpirationDate</key>
    <date>2016-08-12T15:22:33Z</date>
    <key>Name</key>
    <string>iOS Team Provisioning Profile: com.example.PersonalTeam</string>
    <key>ProvisionedDevices</key>
    <array>
        <string>XXXX</string>
    </array>
    <key>LocalProvision</key>
    <true/>
    <key>TeamIdentifier</key>
    <array>
        <string>XXXX</string>
    </array>
    <key>TeamName</key>
    <string>Kosuke Ogawa</string>
    <key>TimeToLive</key>
    <integer>7</integer>
    <key>UUID</key>
    <string>XXXX-XXXX-XXXX-XXXX-XXXX</string>
    <key>Version</key>
    <integer>1</integer>
</dict>
```

## リンク

- [Supported Capabilities](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/SupportedCapabilities/SupportedCapabilities.html#//apple_ref/doc/uid/TP40012582-CH38-SW1)
- [Testflight based internal / External testing](http://stackoverflow.com/questions/30973799/ios-9-new-feature-free-provisioning-run-your-app-on-a-device-just-with-your-ap)
- [\[Xcode\]\[iOS\] 有料ライセンスなしでの実機インストール 全工程解説！ ｜ Developers.IO](http://dev.classmethod.jp/smartphone/run-on-devices-without-apple-developer-program-license/)

[3](https://qiita.com/koogawa/items/#comments)

コメント一覧へ移動

新規登録して、もっと便利にQiitaを使ってみよう

1. あなたにマッチした記事をお届けします
2. 便利な情報をあとで効率的に読み返せます
3. ダークテーマを利用できます
[ログインすると使える機能について](https://help.qiita.com/ja/articles/qiita-login-user)

[新規登録](https://qiita.com/signup?callback_action=login_or_signup&redirect_to=%2Fkoogawa%2Fitems%2F15b231e2728ff64e08f3&realm=qiita) [ログイン](https://qiita.com/login?callback_action=login_or_signup&redirect_to=%2Fkoogawa%2Fitems%2F15b231e2728ff64e08f3&realm=qiita)

[152](https://qiita.com/koogawa/items/15b231e2728ff64e08f3/likers)

いいねしたユーザー一覧へ移動

139