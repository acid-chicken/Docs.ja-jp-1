---
title: ASP.NET Core のホストと展開
author: rick-anderson
description: ホスティング環境を設定し、ASP.NET Core アプリを展開する方法を学習します。
ms.author: riande
ms.custom: mvc
ms.date: 08/07/2017
uid: host-and-deploy/index
ms.openlocfilehash: eb3a4846775fa53132d3692f56bb1b628257ca8f
ms.sourcegitcommit: 57eccdea7d89a62989272f71aad655465f1c600a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2018
ms.locfileid: "44340213"
---
# <a name="host-and-deploy-aspnet-core"></a>ASP.NET Core のホストと展開

通常、ASP.NET Core アプリをホスティング環境に展開するには、以下を実行します。

* ホスティング サーバー上のフォルダーにアプリを発行します。
* 要求の着信時にアプリを開始し、クラッシュ後、またはサーバーの再起動後にアプリを再起動するプロセス マネージャーを設定します。
* リバース プロキシの構成が必要な場合は、アプリに要求を転送するリバース プロキシを設定します。

## <a name="publish-to-a-folder"></a>フォルダーに発行する

[dotnet publish](/dotnet/articles/core/tools/dotnet-publish) CLI コマンドはアプリ コードをコンパイルし、アプリを実行するために必要なファイルを *publish* フォルダーにコピーします。 Visual Studio から展開する場合は、ファイルが展開先にコピーされる前に [dotnet publish](/dotnet/core/tools/dotnet-publish) ステップが自動的に行われます。

### <a name="folder-contents"></a>フォルダーの内容

*publish* フォルダーには、アプリとその依存関係、および必要に応じて .NET ランタイムの *.exe* ファイルと *.dll* ファイルが含まれています。

.NET Core アプリは、*自己完結型*または*フレームワーク依存*アプリとして発行できます。 アプリが自己完結型の場合、.NET ランタイムを含む *.dll* ファイルが *publish* フォルダーに含まれています。 アプリがフレームワークに依存する場合、.NET ランタイムのファイルは含まれていません。これは、サーバーにインストールされている .NET のバージョンへの参照がアプリに含まれていないためです。 既定の展開モデルはフレームワークに依存します。 詳細については、「[.NET Core アプリケーションの展開](/dotnet/articles/core/deploying/index)」を参照してください。

*.exe* ファイルと *.dll* ファイルに加え、ASP.NET Core アプリの *publish* フォルダーには、通常、構成ファイル、静的資産、および MVC ビューが含まれています。 詳細については、[ディレクトリ構造](xref:host-and-deploy/directory-structure)に関するページを参照してください。

## <a name="set-up-a-process-manager"></a>プロセス マネージャーを設定する

ASP.NET Core アプリは、サーバーの起動時に起動し、クラッシュした場合には再起動する必要があるコンソール アプリです。 起動と再起動を自動化するには、プロセス マネージャーが必要です。 ASP.NET Core の最も一般的なプロセス マネージャーは次のとおりです。

* Linux
  * [NGINX](xref:host-and-deploy/linux-nginx)
  * [Apache](xref:host-and-deploy/linux-apache)
* Windows
  * [IIS](xref:host-and-deploy/iis/index)
  * [Windows サービス](xref:host-and-deploy/windows-service)

## <a name="set-up-a-reverse-proxy"></a>リバース プロキシを設定する

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

アプリで [Kestrel](xref:fundamentals/servers/kestrel) Web サーバーを使用する場合は、リバース プロキシ サーバーとして、[NGINX](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache)、または [IIS](xref:host-and-deploy/iis/index) を使用できます。 リバース プロキシ サーバーはインターネットから HTTP 要求を受け取り、事前にいくつかの処理を行ってから Kestrel に転送します。

リバース プロキシ サーバーの有無に関わらず、いずれの構成も有効で、ASP.NET Core 2.0 またはそれ以降のアプリ用にホスティング構成がサポートされている必要があります。 詳細については、「[When to use Kestrel with a reverse proxy](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)」 (Kestrel とリバース プロキシを使用するタイミング) を参照してください。

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

アプリを [Kestrel](xref:fundamentals/servers/kestrel) Web サーバーを使用して、インターネットに公開する場合は、リバース プロキシ サーバーとして、[NGINX](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache)、または [IIS](xref:host-and-deploy/iis/index) を使用します。 リバース プロキシ サーバーはインターネットから HTTP 要求を受け取り、事前にいくつかの処理を行ってから Kestrel に転送します。 リバース プロキシを使用する主な理由はセキュリティです。 詳細については、「[When to use Kestrel with a reverse proxy](xref:fundamentals/servers/kestrel?tabs=aspnetcore1x#when-to-use-kestrel-with-a-reverse-proxy)」 (Kestrel とリバース プロキシを使用するタイミング) を参照してください。

---

## <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

プロキシ サーバーとロード バランサーの背後でホストされているアプリでは、追加の構成が必要になる場合があります。 追加の構成をしないと、要求が発生したスキーム (HTTP/HTTPS) とリモート IP アドレスにアクセスできない場合があります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

## <a name="using-visual-studio-and-msbuild-to-automate-deployment"></a>Visual Studio と MSBuild を使用して展開を自動化する

多くの場合、展開には、[dotnet publish](/dotnet/core/tools/dotnet-publish) からサーバーへの出力のコピーのほか、追加の作業が必要になります。 たとえば、追加のファイルが必要になる場合や、*publish* フォルダーから除外される場合があります。 Visual Studio では Web 展開で MSBuild を使用します。この MSBuild は、展開時に他の多くの作業を行うためにカスタマイズすることができます。 詳細については、[Visual Studio の発行プロファイル](xref:host-and-deploy/visual-studio-publish-profiles)に関するページと、『[Using MSBuild and Team Foundation Build](http://msbuildbook.com/)』という書籍を参照してください。

[Web の発行機能](xref:tutorials/publish-to-azure-webapp-using-vs)または[組み込みの Git サポート](xref:host-and-deploy/azure-apps/azure-continuous-deployment)を使用して、Visual Studio から Azure App Service にアプリを直接展開することができます。 Azure DevOps Services では、[Azure App Service への継続的な展開](/azure/devops/pipelines/targets/webapp)がサポートされています。

## <a name="publishing-to-azure"></a>Azure への発行

Visual Studio を使用してアプリを Azure に発行する方法については、「[Visual Studio を使用して Azure App Service に ASP.NET Core アプリを発行する](xref:tutorials/publish-to-azure-webapp-using-vs)」をご覧ください。 Azure へのアプリの発行は、[コマンド ライン](xref:tutorials/publish-to-azure-webapp-using-cli)で行うこともできます。

## <a name="host-in-a-web-farm"></a>Web ファームでのホスト

Web ファーム環境 (たとえば、スケーラビリティのためのアプリの複数のインスタンスの展開) で ASP.NET Core アプリをホストするための構成については、<xref:host-and-deploy/web-farm> を参照してください。

## <a name="additional-resources"></a>その他の技術情報

ホスティング環境として Docker を使用する方法については、[Docker での ASP.NET Core アプリのホスティング](xref:host-and-deploy/docker/index)に関するページを参照してください。
