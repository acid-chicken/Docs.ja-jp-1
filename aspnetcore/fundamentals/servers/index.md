---
title: ASP.NET Core での Web サーバーの実装
author: rick-anderson
description: ASP.NET Core の Web サーバー Kestrel と HTTP.sys を検出します。 サーバーを選択する方法と、リバース プロキシ サーバーを使用するタイミングについて説明します。
ms.author: tdykstra
ms.custom: mvc
ms.date: 03/13/2018
uid: fundamentals/servers/index
ms.openlocfilehash: bb0331d7201d4e979e6c6524cbf630280c4eaeb6
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/20/2018
ms.locfileid: "36274444"
---
# <a name="web-server-implementations-in-aspnet-core"></a>ASP.NET Core での Web サーバーの実装

作成者: [Tom Dykstra](https://github.com/tdykstra)、[Steve Smith](https://ardalis.com/)、[Stephen Halter](https://twitter.com/halter73)、[Chris Ross](https://github.com/Tratcher)

ASP.NET Core アプリは、インプロセス HTTP サーバー実装を使用して実行されます。 サーバー実装は HTTP 要求をリッスンし、[HttpContext](/dotnet/api/system.web.httpcontext) に構成された[要求機能](xref:fundamentals/request-features)のセットとしてアプリに公開します。

ASP.NET Core には次の 2 つのサーバー実装が用意されています。

* [Kestrel](xref:fundamentals/servers/kestrel) は、ASP.NET Core 向けの既定のクロスプラットフォーム HTTP サーバーです。
* [HTTP.sys](xref:fundamentals/servers/httpsys) は、[HTTP.sys カーネル ドライバーおよび HTTP サーバー API](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx) に基づく Windows 専用の HTTP サーバーです。 (ASP.NET Core 1.x では、HTTP.sys は [WebListener](xref:fundamentals/servers/weblistener) と呼ばれます。)

## <a name="kestrel"></a>Kestrel

Kestrel は、ASP.NET Core のプロジェクト テンプレートに含まれる既定の Web サーバーです。

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Kestrel は単独で使用することも、IIS、NGINX、または Apache などの*リバース プロキシ サーバー*と併用することもできます。 リバース プロキシ サーバーはインターネットから HTTP 要求を受け取り、事前にいくつかの処理を行ってから Kestrel に転送します。

![リバース プロキシ サーバーなしでインターネットと直接通信する Kestrel](kestrel/_static/kestrel-to-internet2.png)

![IIS、NGINX、または Apache などのリバース プロキシ サーバーを介してインターネットと間接的に通信する Kestrel](kestrel/_static/kestrel-to-internet.png)

リバース プロキシ サーバーの有無に関わらず、いずれの構成も有効で、ASP.NET Core 2.0 またはそれ以降のアプリ用にホスティング構成がサポートされている必要があります。 詳細については、「[When to use Kestrel with a reverse proxy](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)」 (Kestrel とリバース プロキシを使用するタイミング) を参照してください。

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

アプリが内部ネットワークからの要求のみを受け入れる場合は、Kestrel を単独で使用することができます。

![内部ネットワークと直接通信する Kestrel](kestrel/_static/kestrel-to-internal.png)

アプリをインターネットに公開する場合は、Kestrel が*リバース プロキシ サーバー*として IIS、NGINX、または Apache を使用する必要があります。 以下の図のように、リバース プロキシ サーバーはインターネットから HTTP 要求を受け取り、事前にいくつかの処理を行ってから Kestrel に転送します。

![IIS、NGINX、または Apache などのリバース プロキシ サーバーを介してインターネットと間接的に通信する Kestrel](kestrel/_static/kestrel-to-internet.png)

エッジ展開 (インターネットからのトラフィックに公開される) でリバース プロキシを使用する最も重要な理由は、セキュリティです。 1.x バージョンの Kestrel はインターネットからの攻撃を防御する重要なセキュリティ機能を備えていません。 これには、適切なタイムアウト、要求サイズの制限、および同時接続の制限などが含まれます (ただし、これらに限定されない)。

詳細については、「[When to use Kestrel with a reverse proxy](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)」 (Kestrel とリバース プロキシを使用するタイミング) を参照してください。

---

IIS、NGINX、および Apache を Kestrel や[カスタム サーバー実装](#custom-servers)なしで使用することはできません。 ASP.NET Core は、プラットフォーム間で一貫して動作できるように、独自のプロセスで実行するように設計されています。 IIS、NGINX、および Apache では独自のスタートアップ プロシージャと環境が指定されます。 これらのサーバー テクノロジを直接使用するには、ASP.NET Core を各サーバーの要件に適合させる必要があります。 Kestrel などの Web サーバー実装を使用することで、ASP.NET Core は異なるサーバー テクノロジでホストされている場合でもスタートアップ プロセスと環境を制御できます。

### <a name="iis-with-kestrel"></a>IIS と Kestrel

ASP.NET Core のリバース プロキシとして [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) または [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) を使用する場合、ASP.NET Core アプリは IIS ワーカー プロセスとは別のプロセスで実行されます。 IIS プロセスで、[ASP.NET Core モジュール](xref:fundamentals/servers/aspnet-core-module)がリバース プロキシの関係を調整します。 ASP.NET Core モジュールの主な機能は、ASP.NET Core アプリを開始し、クラッシュ時にアプリを再始動し、HTTP トラフィックをアプリに転送することです。 詳細については、[ASP.NET Core モジュール](xref:fundamentals/servers/aspnet-core-module)に関するページを参照してください。 

### <a name="nginx-with-kestrel"></a>NGINX と Kestrel

Kestrel のリバース プロキシ サーバーとして Linux で NGINX を使用する方法については、「[Host on Linux with NGINX](xref:host-and-deploy/linux-nginx)」(NGINX による Linux でのホスト) を参照してください。

### <a name="apache-with-kestrel"></a>Apache と Kestrel

Kestrel のリバース プロキシ サーバーとして Linux で Apache を使用する方法については、「[Apache による Linux でのホスト](xref:host-and-deploy/linux-apache)」を参照してください。

## <a name="httpsys"></a>HTTP.sys

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Windows で ASP.NET Core アプリを実行する場合は、HTTP.sys を Kestrel の代わりに使用できます。 最適なパフォーマンスを得るには、通常は Kestrel をお勧めします。 HTTP.sys は、アプリがインターネットに公開されていて、必要な機能が HTTP.sys でサポートされているものの、Kestrel ではサポートされていないシナリオで使用できます。 HTTP.sys の機能については、[HTTP.sys](xref:fundamentals/servers/httpsys) に関するページを参照してください。

![インターネットと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internet.png)

HTTP.sys は、内部ネットワークにのみ公開されるアプリにも使用できます。 

![内部ネットワークと直接通信する HTTP.sys](httpsys/_static/httpsys-to-internal.png)

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

ASP.NET Core 1.x では、HTTP.sys は [WebListener](xref:fundamentals/servers/weblistener) と呼ばれます。 IIS がホストアプリで使用できないシナリオにおいて、Windows で ASP.NET Core アプリを実行する場合は、WebListener を代わりに使用できます。

![インターネットと直接通信する Weblistener](weblistener/_static/weblistener-to-internet.png)

内部ネットワークにのみ公開されるアプリで、必要な機能が WebListener でサポートされていて、Kestrel でサポートされていない場合は、Kestrel の代わりに WebListener も使用できます。 WebListener の機能については、[WebListener](xref:fundamentals/servers/weblistener) に関するページを参照してください。

![内部ネットワークと直接通信する WebListener](weblistener/_static/weblistener-to-internal.png)

---

## <a name="aspnet-core-server-infrastructure"></a>ASP.NET Core サーバー インフラストラクチャ

`Startup.Configure` メソッドで使用できる [IApplicationBuilder](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) は、[IFeatureCollection](/dotnet/api/microsoft.aspnetcore.http.features.ifeaturecollection) 型の [ServerFeatures](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder.serverfeatures) プロパティを公開します。 Kestrel および HTTP.sys (ASP.NET Core 1.x の WebListener) は、それぞれ単独の機能 [IServerAddressesFeature](/dotnet/api/microsoft.aspnetcore.hosting.server.features.iserveraddressesfeature) のみを公開しますが、サーバー実装が異なると追加機能が公開される場合があります。

`IServerAddressesFeature` を使用すれば、実行時にサーバー実装がバインドされたポートを見つけることができます。

## <a name="custom-servers"></a>カスタム サーバー

組み込みサーバーがアプリの要件に合わない場合は、カスタム サーバー実装を作成できます。 [Nowin](https://github.com/Bobris/Nowin) ベースの [IServer](/dotnet/api/microsoft.aspnetcore.hosting.server.iserver) 実装の作成方法については、[Open Web Interface for .NET (OWIN) のガイド](xref:fundamentals/owin)を参照してください。 実装を必要とするのは、アプリで使用される機能のインターフェイスのみです。ただし、少なくとも [IHttpRequestFeature](/dotnet/api/microsoft.aspnetcore.http.features.ihttprequestfeature) と [IHttpResponseFeature](/dotnet/api/microsoft.aspnetcore.http.features.ihttpresponsefeature) はサポートされている必要があります。

## <a name="server-startup"></a>サーバーの起動

[Visual Studio](https://www.visualstudio.com/vs/)、[Visual Studio for Mac](https://www.visualstudio.com/vs/mac/)、または [Visual Studio Code](https://code.visualstudio.com/) を使用する場合、サーバーはアプリが統合開発環境 (IDE) によって開始されたときに起動します。 Windows の Visual Studio では、[IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)/[ASP.NET Core モジュール](xref:fundamentals/servers/aspnet-core-module)またはコンソールで、起動プロファイルを使用してアプリとサーバーを開始することができます。 Visual Studio Code では、CoreCLR デバッガーをアクティブ化する [Omnisharp](https://github.com/OmniSharp/omnisharp-vscode) によって、アプリとサーバーが開始します。 Visual Studio for Mac を使用する場合は、アプリとサーバーが [Mono の Soft-Mode Debugger](http://www.mono-project.com/docs/advanced/runtime/docs/soft-debugger/) によって開始します。

コマンド プロンプトからプロジェクトのフォルダーでアプリを起動する場合、[dotnet run](/dotnet/core/tools/dotnet-run) でアプリとサーバーが起動します (Kestrel および HTTP.sys のみ)。 この構成は、`Debug` (既定) または `Release` のどちらかに設定された `-c|--configuration` オプションによって指定されます。 起動プロファイルが *launchSettings.json* ファイルに存在する場合は、`--launch-profile <NAME>` オプションを使用して起動プロファイルを設定します (`Development`、`Production` など)。 詳細については、[dotnet run](/dotnet/core/tools/dotnet-run) と [.NET Core の配布パッケージ](/dotnet/core/build/distribution-packaging)のトピックを参照してください。

## <a name="additional-resources"></a>その他の技術情報

* [Kestrel](xref:fundamentals/servers/kestrel)
* [Kestrel と IIS](xref:fundamentals/servers/aspnet-core-module)
* [NGINX による Linux でのホスト](xref:host-and-deploy/linux-nginx)
* [Apache による Linux でのホスト](xref:host-and-deploy/linux-apache)
* [HTTP.sys](xref:fundamentals/servers/httpsys) (ASP.NET Core 1.x の場合は [WebListener](xref:fundamentals/servers/weblistener) を参照)
