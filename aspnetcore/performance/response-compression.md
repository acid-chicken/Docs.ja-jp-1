---
title: ASP.NET core 圧縮ミドルウェアの応答
author: guardrex
description: 応答の圧縮と圧縮ミドルウェアの応答を ASP.NET Core アプリケーションで使用する方法について説明します。
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/20/2017
uid: performance/response-compression
ms.openlocfilehash: 585a08d4a6c6e03785e87578d10f6050be8bb73c
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/20/2018
ms.locfileid: "36278220"
---
# <a name="response-compression-middleware-for-aspnet-core"></a>ASP.NET core 圧縮ミドルウェアの応答

作成者: [Luke Latham](https://github.com/guardrex)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/response-compression/samples)します ([ダウンロード方法](xref:tutorials/index#how-to-download-a-sample))。

ネットワーク帯域幅は、限られたリソースです。 通常の応答のサイズを小さくする、アプリの応答性を大幅に多くの場合、増加します。 ペイロードのサイズを小さく 1 つの方法では、圧縮、アプリの応答です。

## <a name="when-to-use-response-compression-middleware"></a>応答の圧縮のミドルウェアを使用する場合

IIS、Apache、または NGINX サーバー ベースの応答の圧縮テクノロジを使用します。 ミドルウェアのパフォーマンス可能性がありますされませんと一致するサーバーにあるモジュール。 [HTTP.sys サーバー](xref:fundamentals/servers/httpsys)と[Kestrel](xref:fundamentals/servers/kestrel)現在組み込み圧縮サポートが提供されません。

場合は、圧縮ミドルウェアの応答を使用します。

* 次のサーバー ベースの圧縮テクノロジを使用することができません。
  * [動的な圧縮の IIS モジュール](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate モジュール](http://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [NGINX 圧縮および圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接ホスティング。
  * [HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称[WebListener](xref:fundamentals/servers/weblistener))
  * [Kestrel](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>応答の圧縮

通常は、ネイティブに圧縮された応答は、応答の圧縮を利用できます。 通常ネイティブに圧縮された応答が含まれます: CSS、JavaScript、HTML、XML、および JSON。 PNG ファイルなどのネイティブ圧縮されたアセットを圧縮することはできません。 ネイティブ圧縮された応答をさらに圧縮するしようとすると、サイズと転送時間の小さな追加低下は可能性がありますする影が薄くなって、圧縮処理にかかった時間によってです。 (ファイルの内容および圧縮の効率性) に応じて約 150 ~ 1000 バイト未満のファイルを圧縮されていません。 圧縮されたファイルのサイズが圧縮されていないファイルよりも小さいファイルの圧縮のオーバーヘッドがあります。

送信することによってその機能のサーバーにクライアントを通知する必要があります、クライアントは、圧縮されたコンテンツを処理できますが、ときに、`Accept-Encoding`要求でヘッダー。 内の情報を含めることは、サーバーは、圧縮されたコンテンツを送信するとき、`Content-Encoding`圧縮された応答をエンコードする方法のヘッダー。 ミドルウェアによってサポートされているコンテンツのエンコードの指定は、次の表に示します。

| `Accept-Encoding` ヘッダーの値 | ミドルウェアのサポート | 説明                                                 |
| ------------------------------- | :------------------: | ----------------------------------------------------------- |
| `br`                            | いいえ                   | Brotli 圧縮データの形式                               |
| `compress`                      | いいえ                   | UNIX"を圧縮"データの形式                                 |
| `deflate`                       | いいえ                   | データの形式が"zlib"内のデータを圧縮する"deflate"     |
| `exi`                           | いいえ                   | W3C の効率的な XML インターチェンジ                               |
| `gzip`                          | [はい] (既定値)        | gzip ファイル形式                                            |
| `identity`                      | [はい]                  | 「エンコードなし」の識別子: 応答をエンコードする必要があります。 |
| `pack200-gzip`                  | いいえ                   | ネットワーク転送形式は Java アーカイブ用                   |
| `*`                             | [はい]                  | エンコードは明示的に要求された使用可能なコンテンツ     |

詳細については、次を参照してください。、 [IANA 公式コンテンツ コーディング リスト](http://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)です。

ミドルウェアでは、カスタムの圧縮は追加のプロバイダーを追加することができます`Accept-Encoding`ヘッダー値。 詳細については、次を参照してください。[カスタム プロバイダー](#custom-providers)以下です。

ミドルウェアは品質の値に対応できる (qvalue、 `q`) 圧縮スキームの優先順位をクライアントによって送信されるときに重み付け。 詳細については、次を参照してください。 [RFC 7231: Accept-encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4)です。

圧縮アルゴリズムは、圧縮の速度と、圧縮の有効性のバランスが適用されます。 *有効性*このコンテキストでは、出力のサイズを圧縮した後です。 最小サイズは最もによって実現*最適な*圧縮します。

要求に関連するヘッダーを送信する、キャッシュ、および圧縮されたコンテンツを受信は、次の表に記述されています。

| Header             | ロール |
| ------------------ | ---- |
| `Accept-Encoding`  | コンテンツのエンコードにクライアントに許容されるスキームを示すために、クライアントからサーバーに送信します。 |
| `Content-Encoding` | ペイロード内のコンテンツのエンコードを示すために、サーバーからクライアントに送信します。 |
| `Content-Length`   | 圧縮が発生すると、`Content-Length`本文内容が変更された応答が圧縮される場合に以降のヘッダーは削除します。 |
| `Content-MD5`      | 圧縮が発生すると、`Content-MD5`本文の内容が変更されており、ハッシュが無効になったため、ヘッダーが削除されます。 |
| `Content-Type`     | コンテンツの MIME の種類を指定します。 すべての応答を指定する必要があります、`Content-Type`です。 ミドルウェアは、応答を圧縮する必要があるかどうかを決定するには、この値を確認します。 ミドルウェアのセットを指定する[既定の MIME の種類](#mime-types)をエンコードできますが、置き換えるか、MIME の種類を追加することができます。 |
| `Vary`             | 値を使用してサーバーによって送信されると`Accept-Encoding`クライアントと、プロキシを`Vary`ヘッダーがクライアントまたはキャッシュするプロキシを示します (異なります) の値に基づいて、応答、`Accept-Encoding`要求のヘッダー。 コンテンツを返すことの結果、`Vary: Accept-Encoding`ヘッダーは、両方は圧縮され、圧縮されていない応答を個別にキャッシュされます。 |

応答の圧縮のミドルウェアの機能を調べることができます、[サンプル アプリ](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/response-compression/samples)です。 このサンプルを示しています。

* Gzip および圧縮のカスタム プロバイダーを使用してアプリの応答の圧縮します。
* 圧縮の MIME の種類の既定のリストに、MIME の種類を追加する方法です。

## <a name="package"></a>Package

::: moniker range="< aspnetcore-2.1"

ミドルウェアをプロジェクトに含めるへの参照を追加、 [Microsoft.AspNetCore.ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)パッケージです。 この機能は、ASP.NET Core 1.1.0 以上をターゲットとするアプリで使用できます。

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

ミドルウェアをプロジェクトに含めるへの参照を追加、 [Microsoft.AspNetCore.ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)パッケージまたはを使用して、 [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 以降)。

::: moniker-end

## <a name="configuration"></a>構成

次のコードは、既定 gzip 圧縮を使用し、既定の MIME の種類の応答の圧縮のミドルウェアを有効にする方法を示しています。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

> [!NOTE]
> などのツールを使用して[Fiddler](http://www.telerik.com/fiddler)、 [Firebug](http://getfirebug.com/)、または[Postman](https://www.getpostman.com/)を設定する、`Accept-Encoding`要求ヘッダーおよび応答ヘッダー、サイズ、および本文を調査します。

サンプル アプリをせずに要求を送信、`Accept-Encoding`ヘッダー応答は圧縮されていないことを確認します。 `Content-Encoding`と`Vary`ヘッダーが応答に表示されません。

![Fiddler のウィンドウが、要求に Accept-encoding ヘッダーなしの結果を示すです。 応答が圧縮されていません。](response-compression/_static/request-uncompressed.png)

要求のサンプル アプリを送信、`Accept-Encoding: gzip`ヘッダー応答が圧縮されていることを確認します。 `Content-Encoding`と`Vary`ヘッダーが応答に存在します。

![Fiddler の Accept-encoding ヘッダーで要求の結果と gzip の値を示すウィンドウです。 Vary と Content-encoding ヘッダーは、応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed.png)

## <a name="providers"></a>プロバイダー

### <a name="gzipcompressionprovider"></a>GzipCompressionProvider

使用して、 [GzipCompressionProvider](/dotnet/api/microsoft.aspnetcore.responsecompression.gzipcompressionprovider)に gzip で応答を圧縮します。 これは、指定がない場合、圧縮の既定のプロバイダーです。 レベルを圧縮を設定することができます、 [GzipCompressionProviderOptions](/dotnet/api/microsoft.aspnetcore.responsecompression.gzipcompressionprovideroptions)です。

Gzip 圧縮プロバイダーの既定値は、最速の圧縮レベル ([CompressionLevel.Fastest](/dotnet/api/system.io.compression.compressionlevel))、これが最も効率的な圧縮を得られない。 最も効率的な圧縮を使用する場合は、最適な圧縮のミドルウェアを構成することができます。

| 圧縮レベル | 説明 |
| ----------------- | ----------- |
| [CompressionLevel.Fastest](/dotnet/api/system.io.compression.compressionlevel) | 圧縮は、結果の出力は圧縮されていない最適な場合でも、可能な限り早く完了します。 |
| [CompressionLevel.NoCompression](/dotnet/api/system.io.compression.compressionlevel) | 圧縮は行われません。 |
| [CompressionLevel.Optimal](/dotnet/api/system.io.compression.compressionlevel) | 応答を最適に圧縮する、場合でも、圧縮を完了に時間がかかります。 |

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)

[!code-csharp[](response-compression/samples/2.x/Startup.cs?name=snippet1&highlight=5,12-15)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)

[!code-csharp[](response-compression/samples/1.x/Startup.cs?name=snippet2&highlight=5,12-15)]

---

## <a name="mime-types"></a>MIME タイプ

ミドルウェアは、圧縮の MIME の種類の既定のセットを指定します。

* `text/plain`
* `text/css`
* `application/javascript`
* `text/html`
* `application/xml`
* `text/xml`
* `application/json`
* `text/json`

置き換えるか、応答の圧縮のミドルウェアのオプションで MIME の種類を追加することができます。 そのワイルドカード MIME に注意してくださいなどの型`text/*`はサポートされていません。 サンプル アプリの MIME の種類を追加する`image/svg+xml`と圧縮および ASP.NET Core バナー イメージの機能 (*banner.svg*)。

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)

[!code-csharp[](response-compression/samples/2.x/Startup.cs?name=snippet1&highlight=7-9)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)

[!code-csharp[](response-compression/samples/1.x/Startup.cs?name=snippet2&highlight=7-9)]

---

### <a name="custom-providers"></a>カスタム プロバイダー

カスタムの圧縮の実装を作成する[ICompressionProvider](/dotnet/api/microsoft.aspnetcore.responsecompression.icompressionprovider)です。 [EncodingName](/dotnet/api/microsoft.aspnetcore.responsecompression.icompressionprovider.encodingname)コンテンツのエンコードにこれを表す`ICompressionProvider`が生成されます。 ミドルウェアでは、この情報を使用して、指定されたリストに基づくプロバイダーを選択して、`Accept-Encoding`要求のヘッダー。

サンプル アプリを使用して、クライアント要求を送信すると、`Accept-Encoding: mycustomcompression`ヘッダー。 ミドルウェアがカスタム圧縮の実装を使用し、応答を返し、`Content-Encoding: mycustomcompression`ヘッダー。 クライアントは、カスタムの圧縮の実装が機能するためにはカスタム エンコーディングを圧縮解除できる必要があります。

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)

[!code-csharp[](response-compression/samples/2.x/Startup.cs?name=snippet1&highlight=5,12-15)]

[!code-csharp[](response-compression/samples/2.x/CustomCompressionProvider.cs?name=snippet1)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)

[!code-csharp[](response-compression/samples/1.x/Startup.cs?name=snippet2&highlight=5,12-15)]

[!code-csharp[](response-compression/samples/1.x/CustomCompressionProvider.cs?name=snippet1)]

---

サンプル アプリケーションに要求を送信、`Accept-Encoding: mycustomcompression`ヘッダーおよび応答ヘッダーを確認します。 `Vary`と`Content-Encoding`ヘッダーが応答に存在します。 (非表示)、応答本文は、このサンプルで圧縮されていません。 圧縮の実装ではありません、`CustomCompressionProvider`サンプルのクラスです。 ただし、このような圧縮アルゴリズムを実装するかどうかを示します。

![Fiddler の Accept-encoding ヘッダーで要求の結果と mycustomcompression の値を示すウィンドウです。 Vary と Content-encoding ヘッダーは、応答に追加されます。](response-compression/_static/request-custom-compression.png)

## <a name="compression-with-secure-protocol"></a>セキュリティで保護されたプロトコルと圧縮

セキュリティで保護された接続上で圧縮された応答を制御することができます、`EnableForHttps`オプションは、既定で無効にします。 動的に生成されたページの圧縮を使用してが問題につながるセキュリティなど、 [CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit))と[侵害](https://wikipedia.org/wiki/BREACH_(security_exploit))攻撃です。

## <a name="adding-the-vary-header"></a>Vary ヘッダーを追加します。

::: moniker range=">= aspnetcore-2.0"

基づいている場合の応答の圧縮、`Accept-Encoding`ヘッダー、可能性のある複数の圧縮バージョン応答と圧縮されていないバージョンがあります。 複数のバージョンが存在しを保存するか、クライアントとプロキシのキャッシュを指示するために、`Vary`ヘッダーを追加、`Accept-Encoding`値。 ASP.NET Core 2.0 以降では、ミドルウェアは追加、`Vary`ヘッダー応答が圧縮されるときに自動的にします。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

基づいている場合の応答の圧縮、`Accept-Encoding`ヘッダー、可能性のある複数の圧縮バージョン応答と圧縮されていないバージョンがあります。 複数のバージョンが存在しを保存するか、クライアントとプロキシのキャッシュを指示するために、`Vary`ヘッダーを追加、`Accept-Encoding`値。 ASP.NET Core で 1.x では、追加、`Vary`を応答にヘッダーを手動で行います。

[!code-csharp[](response-compression/samples/1.x/Startup.cs?name=snippet1)]

::: moniker-end

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>NGINX リバース プロキシの背後にある場合のミドルウェアの問題

要求が NGINX がプロキシとなったとき、`Accept-Encoding`ヘッダーを削除します。 これは、ミドルウェアが応答を圧縮することを防ぎます。 詳細については、次を参照してください。 [NGINX: 圧縮および圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)です。 によってこの問題を追跡[NGINX (BasicMiddleware #123) の圧縮をパススルーもらう](https://github.com/aspnet/BasicMiddleware/issues/123)です。

## <a name="working-with-iis-dynamic-compression"></a>IIS 動的圧縮を使用

モジュールがある場合、アクティブな IIS 動的圧縮に対してアプリを無効にするにはサーバー レベルで構成されている場合、監視できるように、追加すると、 *web.config*ファイル。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

などのツールを使用して[Fiddler](http://www.telerik.com/fiddler)、 [Firebug](http://getfirebug.com/)、または[Postman](https://www.getpostman.com/)を設定することを`Accept-Encoding`要求ヘッダーおよび応答ヘッダー、サイズ、および本文を調査します。 応答の圧縮のミドルウェアが圧縮を次の条件を満たす応答。

* `Accept-Encoding`ヘッダーがあり、値は存在`gzip`、 `*`、または確立したユーザー圧縮プロバイダーに一致するカスタム エンコードします。 値にはできません`identity`または品質の値が (qvalue、 `q`) を 0 (ゼロ) を設定します。
* MIME の種類 (`Content-Type`) が設定されで構成されている MIME の種類に一致する必要があります、 [ResponseCompressionOptions](/dotnet/api/microsoft.aspnetcore.responsecompression.responsecompressionoptions)です。
* 要求を含めないでください、`Content-Range`ヘッダー。
* 要求は、安全なプロトコル (https) が応答の圧縮のミドルウェアのオプションで構成されている場合を除き、安全ではないプロトコル (http) を使用する必要があります。 *危険があることに注意してください[上記](#compression-with-secure-protocol)セキュリティで保護されたコンテンツの圧縮を有効にする場合。*

## <a name="additional-resources"></a>その他の技術情報

* [アプリケーションの起動](xref:fundamentals/startup)
* [ミドルウェア](xref:fundamentals/middleware/index)
* [Mozilla Developer Network: Accept-encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 セクション 3.1.2.1: コンテンツのコーディング](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 4.2.3: Gzip のコーディング](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP ファイル形式仕様バージョン 4.3](http://www.ietf.org/rfc/rfc1952.txt)
