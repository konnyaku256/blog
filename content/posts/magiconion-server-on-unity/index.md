---
title: "MagicOnion ServerをUnityで動かす"
date: 2020-04-14T11:25:21+09:00
description: "MagicOnion ServerをUnity(Monoランタイム)でも動かすことができたので紹介します。"
meta_image: "posts/magiconion-server-on-unity/ogp.png"
tags:
- MagicOnion
- gRPC
- Unity
draft: false
---

## MagicOnion
MagicOnionはC#製のリアルタイム通信エンジンで、gRPCにおける4つのRPCサービスがAPIとStreamingの2つのサービスとしてラップされています。Streamingサービスではクライアント/サーバ間で双方向通信しつつ他のクライアントにBroadcastできるなど、ゲーム開発に必要な機能がgRPCの上に拡張されています。
ClientはUnityと.NET Core、Serverは.NET Coreがサポートされているので、クライアントからサーバまでC#だけで一貫した開発ができます。

> MagicOnion is an Realtime Network Engine like [SignalR](https://github.com/aspnet/AspNetCore/tree/master/src/SignalR), [Socket.io](https://socket.io/) and RPC-Web API Framework like any web-framework.
>
> MagicOnion is built on [gRPC](https://grpc.io/) so fast(HTTP/2) and compact(binary) network transport. It does not requires `.proto` and generate unlike plain gRPC. Protocol schema can share a C# interface and classes.
>
> MagicOnion is for Microservices(communicate between .NET Core Servers like Orleans, ServiceFabric, AMBROSIA), API Service(for WinForms/WPF like WCF, ASP.NET Core MVC), Native Client’s API(for Xamarin, Unity) and Realtime Server that replacement like Socket.io, SignalR, Photon, UNet, etc.

https://github.com/Cysharp/MagicOnion

開発者の@neueccさんによる資料がたいへん参考になります。
<iframe src="//www.slideshare.net/slideshow/embed_code/key/s9FspXgoqnZxvL" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/neuecc/unitymagiconionc" title="Unityによるリアルタイム通信とMagicOnionによるC#大統一理論の実現" target="_blank">Unityによるリアルタイム通信とMagicOnionによるC#大統一理論の実現</a> </strong> from <strong><a href="https://www.slideshare.net/neuecc" target="_blank">Yoshifumi Kawai</a></strong> </div>

## MagicOnion ServerをUnityで動かす
本題です。
MagicOnionはC#で一貫した開発体験を提供してくれますが、さらにUnityで一貫させることはできないのか？と思い、試してみました。
UnityにはServer Buildというビルドオプションがあるので、これを使ってUnityをHeadlessモードでビルド、コンソールアプリとして実行することができるようになっています。
つまり、Unityでもサーバサイド開発ができます。
ただし、上記で紹介したようにMagicOnionのサーバ実装は.NET CoreしかサポートされていないのでUnity(Monoランタイム)で動かすには少しばかり修正が必要でした。

以下、動いたところまで手順を共有します。

### 確認環境
- Unity 2019.3.5f1
- MagicOnion 3.0.11
- MessagePack for C# 2.1.90
- gRPC for C# 2.27.0

### とりあえずUnity用に必要なものをimportする
https://github.com/Cysharp/MagicOnion#unity-client-supports に従って
- MagicOnion
- MessagePack for C#
- gRPC for C#

をimportしました。

### MagicOnion.Serverを追加する
MagicOnionがUnity Client用なので`MagicOnion.Server`以下のほとんどが除外されていました。
https://github.com/Cysharp/MagicOnion/tree/master/src/MagicOnion/Server から.NET Core用の`MagicOnion.Server`を追加で取り込みました。

この時点で案の定Unity EditorのConsoleにいくつかのエラーが発生しました。
これらを順に解決していきました。

### MagicOnion/Server/Hubs/Group.ImmutableArray.cs: error CS0234, CS0246
- `error CS0234: The type or namespace name 'Immutable' does not exist in the namespace 'System.Collections'`
- `error CS0246: The type or namespace name 'ImmutableArray<>' could not be found`

Unity(Mono)と.NET CoreのSystem.Collections名前空間の実装内容は微妙に違っていて、Unity側には`System.Collections.Immutable`が存在しないようでした。
https://www.nuget.org/packages/System.Collections.Immutable/ から手動で.NET Standard2.0用のdllを追加しました。

### MagicOnion/Server/Hubs/StreamingHubHandlerRepository.cs: error CS0246
- `error CS0246: The type or namespace name 'UniqueHashDictionary<>' could not be found`

`MagicOnion/Utils/UniqueHashDictionary.cs`が`#if NON_UNITY` defineによってUnity側で無効になっているようでした。
Unity以外のランタイムで使用する想定はなかったのでdefineを外しました。

### MagicOnion/Server/MethodHandler.cs: error CS0122
- `error CS0122: 'xxx' is inaccessible due to its protection level`

xxxのアクセス修飾子がinternalで定義されていて参照できないようでした。
今回はimportしたcsコードをdll化する想定がなかったため、暫定的にpublic定義に変更して解決しました。

### MagicOnion/Server/Hubs/Group.ConcurrentDictionary.cs: error CS0246
- `error CS0246: The type or namespace name 'ReservedWhenAllPromise' could not be found`

`MagicOnion/Utils/ReservedWhenAllPromise.cs`が`#if NON_UNITY` defineによってUnity側で無効になっているようでした。
Unity以外のランタイムで使用する想定はなかったのでdefineを外しました。

ここまででUnity Editor Console上のエラーが全て消えました。

### InvalidProgramException: Invalid IL code in MagicOnion.Utils.UniqueHashDictionary:CreateTable()
サンプル実装を使って動作確認したら発生しました。
`MagicOnion/Utils/UniqueHashDictionary.cs`で実装されている`CreateTable()`メソッドで発生していました。
メソッド内の変数代入処理で
```cs
ref var v = ref values[i];
```
のように`ref`による参照渡しを使っていることが原因のようでした。
通常の値渡しによる代入に変更して解決しました。

以上でMagicOnion ServerがUnityで動くようになりました。

## 所感
MagicOnionのサーバ実装は.NET CoreしかサポートされてないしUnityでは無理かな〜と思っていたのですが、意外にも(動かすだけなら)すんなりと動かせてしまったので驚いています。
ただ、実際に実装する際に推奨されている`MagicOnion.Hosting`を使った場合の動作確認まではできていないので今後検証したいと思っています。
できれば.NET Generic Hostに乗っかっておきたいですからね。

MagicOnion ServerをUnityで動かした場合、Unityの物理演算やゲームロジック、AIをそのままサーバ側に持たせることができるので、クライアントからサーバまでのゲーム開発を完全にUnityで一貫、コードも共通化させることができそうです。
Unity Server Buildでのパフォーマンスが通常のサーバアプリケーションと比べると気になりますが、開発効率向上の点で今後期待できそうでした。