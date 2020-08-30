---
layout: post
title: iOSのシステムフォント
---

iOSのシステムフォントはSF Proですが、SF Proは英語のフォントで、自然に英語に必要な文字しかサポートしていません。では普段に見ている日本語はどのフォントになっているのでしょうか？

[こちら](https://developer.apple.com/fonts/system-fonts/)を参考し、見比べてみたら、Hiragino Sansのはずです。とはいえ、これはあくまで言語設定が日本語になっているからです。漢字を表示できるいくつの言語が設定された場合、一番上の言語のデフォルトフォントが採用されます。

日本語のデフォルトフォントHiragino Sansになっていますが、フォントを直接にHiragino Sansに設定して比べてみたら、こうになります。

| ![VStack 左揃え](/assets/images/system-font-vs-hiragino-sans-vstack.png) | ![HStack ベースライン揃え](/assets/images/system-font-vs-hiragino-sans-hstack.png) |
|:-:|:-:|
| VStack 左揃え | HStack ベースライン揃え |

わずかですが、Hiragino Sansの方が大きです。その違いはおよそ1pt[^1]。Hiragino Sansでデザインしたら、およそ1ptのサイズ差が出ます。

では開発の方もHiragino Sansで実装すればいいじゃないと思いますよね。しかしHiragino Sans自身の問題かiOSだけの問題はわかりませんが、Hiragino Sansがアルファベットをサポートしているのに、十分なディセントがなくて、英文字が途切れる[^2]し、上部のスペースもシステムフォントより狭いです。

もしかしてこれらの問題を補うために調整しているかと思い、上下の差を補って、フォントを制限された高さにより小さくなれるようにしてみました。

| ![VStack 左揃え（サイズ調整後）](/assets/images/system-font-vs-hiragino-sans-vstack-with-size-adjust.png) | ![HStack ベースライン揃え（サイズ調整後）](/assets/images/system-font-vs-hiragino-sans-hstack-with-size-adjust.png) |
|:-:|:-:|
| VStack 左揃え | HStack ベースライン揃え |

完璧じゃんと思いきや、なぜか「本」と「語」の距離が1pxの差があります。

設定を中国語にして、中国語のデフォルトフォントを使い、同じ調整してみた結果、全然サイズが揃えません。

また、調整をやめて中国語のデフォルトフォントを設定してみたらこうなります。

| ![VStack 左揃え（中国語フォント）](/assets/images/system-font-vs-pingfang-hk-vstack.png) | ![HStack ベースライン揃え（中国語フォント）](/assets/images/system-font-vs-pingfang-hk-hstack.png) |
|:-:|:-:|
| VStack 左揃え | HStack ベースライン揃え |

スペースの違いがあるものの、サイズは1ptほどの差がありません。そして上下のスペースが結構大きいし、英文字が途切れる問題もありません。

もともと当てるだけのサイズ調整方法がまた「上下のスペースがより小さくなる場合」という条件をつけないと成立しなくなるので、そもそもフォントごとに調整が違うことも考えられます。

では1pt上げればどうなります？

| ![VStack 左揃え（システムフォントが1pt大きい）](/assets/images/system-font-one-size-up-vs-hiragino-sans-vstack.png) | ![HStack ベースライン揃え（システムフォントが1pt大きい）](/assets/images/system-font-one-size-up-vs-hiragino-sans-hstack.png) |
|:-:|:-:|
| VStack 左揃え | HStack ベースライン揃え |

最初の調整してみた結果より違いが大きですが、1ptぐらいの差は確実にありません。

## まとめ

システムフォントを設定した場合

- SF Proで表示できない文字は言語設定により実際のフォントが変わる
- フォントメトリクスはSF Proになり、実際のフォントを直接に設定した場合の表示と違う
- その違いは規則がないようで、言語により違う
- 日本語しか考慮しない場合、フォントサイズを1ptあげることで、デザインとのフォントサイズの差をコスパ良く改善できる

***

[^1]: [【iOS】System Fontの和文（日本語）について](https://qiita.com/moccow/items/4d0870e81db909a7aabd)

[^2]: [CoreText ヒラギノフォント(日本語)で正確に描画サイズを取得する](https://qiita.com/yusuga/items/2be8c55ca561bba44702)
