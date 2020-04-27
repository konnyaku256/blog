---
title: "MagicOnion.HostingをUnityで動かす"
date: 2020-04-28T01:05:59+09:00
description: "MagicOnion.HostingをUnity(Monoランタイム)でも動かすことに成功しました。"
meta_image: "posts/magiconion-hosting-on-unity/ogp.png"
tags:
- MagicOnion
- gRPC
- Unity
draft: false
---

## 前置き
`MagicOnion.Server`をホスティングするにあたって、推奨の`MagicOnion.Hosting`は.NETアプリにおける汎用的なホスティング機能の`Microsoft.Extensions.Hosting`に基づいているため、そのノウハウの恩恵を受けつつより柔軟な設定を行うことができるようになっています。(`Microsoft.Extensions.Hosting`については[.NET Generic Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-2.2)が詳しい)
そこで今回は、[前回](https://blog.konnyaku256.dev/posts/magiconion-server-on-unity/)の投稿で未検証となっていた`MagicOnion.Hosting`を用いた場合の検証結果について共有します。

## MagicOnion.HostingをUnityで動かす
以下、動いたところまで手順を共有します。

### 確認環境
- Unity 2019.3.5f1
- MagicOnion 3.0.11
- MessagePack for C# 2.1.90
- gRPC for C# 2.27.0

前回までの手順が完了していることが前提です。

### MagicOnion.Hostingを追加する
https://github.com/Cysharp/MagicOnion/tree/master/src/MagicOnion.Hosting から.NET Core用の`MagicOnion.Hosting`を追加で取り込みました。

### Microsoft.Extensions.Hostingとその依存関係を全て取り込む
`MagicOnion.Hosting`は`Microsoft.Extensions.Hosting`を必要とするのでそれらを取り込みました。
依存関係が多いため、Unity用NuGet Package Managerの[NuGetForUnity](https://github.com/GlitchEnzo/NuGetForUnity)を使いました。

### dllの重複を解消する
前回追加していた
- System.Buffers.dll
- System.Memory.dll
- System.Runtime.CompilerServices.Unsafe.dll
- System.Threading.Tasks.Extensions.dll

のdllが重複してしまったのでNuGetForUnityで取り込んだほうを削除しました。

以上で`MagicOnion.Hosting`がUnityで動くようになりました。

## 課題点
ここまでで動きはした...のですが、実際のところはUnity Editor上で実行すると実行時に`Unloading broken assembly xxx.dll, this assembly can cause crashes in the runtime`のようなエラーが発生してしまいます。(動作上の問題はありません)
ビルド時にはこれがWarningに変わるので、取り込み方に問題があってプロジェクトのアセンブリ参照がバグっているような気がしています。
この問題は解決次第、更新したいと思います。
