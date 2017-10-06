---
title: "ローカライズの移植性のオブジェクトを構成します。"
author: sebastienros
description: "この記事では、ポータブル オブジェクト ファイルを紹介し、Orchard のコア フレームワークと ASP.NET Core アプリケーションで使用するための手順の概要を示します。"
keywords: "ASP.NET Core、ローカリゼーション、カルチャ、言語、ポータブル オブジェクト"
ms.author: scaddie
manager: wpickett
ms.date: 09/26/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/portable-object-localization
ms.openlocfilehash: 4fa73ae08b10217de657681a27f6097fc3443737
ms.sourcegitcommit: 732cd2684246e49e796836596643a8d37e20c46d
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/01/2017
---
# <a name="configure-portable-object-localization-with-orchard-core"></a><span data-ttu-id="90984-104">Orchard コアでポータブル オブジェクト ローカリゼーションを構成します。</span><span class="sxs-lookup"><span data-stu-id="90984-104">Configure portable object localization with Orchard Core</span></span>

<span data-ttu-id="90984-105">によって[Sébastien Ros](https://github.com/sebastienros)と[Scott Addie](https://twitter.com/Scott_Addie)</span><span class="sxs-lookup"><span data-stu-id="90984-105">By [Sébastien Ros](https://github.com/sebastienros) and [Scott Addie](https://twitter.com/Scott_Addie)</span></span>

<span data-ttu-id="90984-106">この記事で ASP.NET Core アプリケーションのポータブル オブジェクト (PO) ファイルを使用するための手順を追って、 [Orchard コア](https://github.com/OrchardCMS/OrchardCore)フレームワークです。</span><span class="sxs-lookup"><span data-stu-id="90984-106">This article walks through the steps for using Portable Object (PO) files in an ASP.NET Core application with the [Orchard Core](https://github.com/OrchardCMS/OrchardCore) framework.</span></span>

<span data-ttu-id="90984-107">**注:** Orchard コアは、Microsoft 製品ではありません。</span><span class="sxs-lookup"><span data-stu-id="90984-107">**Note:** Orchard Core is not a Microsoft product.</span></span> <span data-ttu-id="90984-108">したがって、マイクロソフトはサポートされませんこの機能します。</span><span class="sxs-lookup"><span data-stu-id="90984-108">Consequently, Microsoft provides no support for this feature.</span></span>

<span data-ttu-id="90984-109">[表示またはダウンロードするサンプル コード](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/localization/sample/POLocalization)([をダウンロードする方法](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="90984-109">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/localization/sample/POLocalization) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="what-is-a-po-file"></a><span data-ttu-id="90984-110">PO ファイルとは何ですか。</span><span class="sxs-lookup"><span data-stu-id="90984-110">What is a PO file?</span></span>

<span data-ttu-id="90984-111">PO ファイルは、特定の言語の翻訳された文字列を含むテキスト ファイルとして配布します。</span><span class="sxs-lookup"><span data-stu-id="90984-111">PO files are distributed as text files containing the translated strings for a given language.</span></span> <span data-ttu-id="90984-112">PO ファイルを代わりに使用するいくつか利点*.resx*ファイルが含まれます。</span><span class="sxs-lookup"><span data-stu-id="90984-112">Some advantages of using PO files instead *.resx* files include:</span></span>
- <span data-ttu-id="90984-113">PO ファイルは、複数形化; をサポートします。*.resx*複数形化をサポートしていないファイルです。</span><span class="sxs-lookup"><span data-stu-id="90984-113">PO files support pluralization; *.resx* files don't support pluralization.</span></span>
- <span data-ttu-id="90984-114">PO ファイルはのようにコンパイルされない*.resx*ファイル。</span><span class="sxs-lookup"><span data-stu-id="90984-114">PO files aren't compiled like *.resx* files.</span></span> <span data-ttu-id="90984-115">そのため、特別なツールとビルド手順は必要ありません。</span><span class="sxs-lookup"><span data-stu-id="90984-115">As such, specialized tooling and build steps aren't required.</span></span>
- <span data-ttu-id="90984-116">PO ファイルは、オンライン編集ツールを共同でうまく機能します。</span><span class="sxs-lookup"><span data-stu-id="90984-116">PO files work well with collaborative online editing tools.</span></span>

### <a name="example"></a><span data-ttu-id="90984-117">例</span><span class="sxs-lookup"><span data-stu-id="90984-117">Example</span></span>

<span data-ttu-id="90984-118">フランス語、その複数形の 1 つを含む 2 つの文字列の翻訳を含むサンプル PO ファイルを次に示します。</span><span class="sxs-lookup"><span data-stu-id="90984-118">Here is a sample PO file containing the translation for two strings in French, including one with its plural form:</span></span>

<span data-ttu-id="90984-119">*fr.po*</span><span class="sxs-lookup"><span data-stu-id="90984-119">*fr.po*</span></span>

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

<span data-ttu-id="90984-120">この例では、次の構文を使用します。</span><span class="sxs-lookup"><span data-stu-id="90984-120">This example uses the following syntax:</span></span>

- <span data-ttu-id="90984-121">`#:`: 変換対象の文字列のコンテキストを示すコメントです。</span><span class="sxs-lookup"><span data-stu-id="90984-121">`#:`: A comment indicating the context of the string to be translated.</span></span> <span data-ttu-id="90984-122">同じ文字列が使用されているに応じて異なる方法で変換可能性があります。</span><span class="sxs-lookup"><span data-stu-id="90984-122">The same string might be translated differently depending on where it is being used.</span></span>
- <span data-ttu-id="90984-123">`msgid`: 無変換文字列。</span><span class="sxs-lookup"><span data-stu-id="90984-123">`msgid`: The untranslated string.</span></span>
- <span data-ttu-id="90984-124">`msgstr`: 翻訳された文字列。</span><span class="sxs-lookup"><span data-stu-id="90984-124">`msgstr`: The translated string.</span></span>

<span data-ttu-id="90984-125">複数形化のサポートの場合は、複数のエントリを定義できます。</span><span class="sxs-lookup"><span data-stu-id="90984-125">In the case of pluralization support, more entries can be defined.</span></span>

- <span data-ttu-id="90984-126">`msgid_plural`: 無変換複数形文字列。</span><span class="sxs-lookup"><span data-stu-id="90984-126">`msgid_plural`: The untranslated plural string.</span></span>
- <span data-ttu-id="90984-127">`msgstr[0]`: 0 の場合、翻訳された文字列。</span><span class="sxs-lookup"><span data-stu-id="90984-127">`msgstr[0]`: The translated string for the case 0.</span></span>
- <span data-ttu-id="90984-128">`msgstr[N]`: 大文字の n。 の翻訳された文字列</span><span class="sxs-lookup"><span data-stu-id="90984-128">`msgstr[N]`: The translated string for the case N.</span></span>

<span data-ttu-id="90984-129">PO ファイルの仕様を参照して[ここ](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html)です。</span><span class="sxs-lookup"><span data-stu-id="90984-129">The PO file specification can be found [here](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html).</span></span>

## <a name="configuring-po-file-support-in-aspnet-core"></a><span data-ttu-id="90984-130">ASP.NET Core の PO ファイル サポートの構成</span><span class="sxs-lookup"><span data-stu-id="90984-130">Configuring PO file support in ASP.NET Core</span></span>

<span data-ttu-id="90984-131">この例は、Visual Studio 2017 プロジェクト テンプレートから生成された ASP.NET Core MVC アプリケーションに基づいています。</span><span class="sxs-lookup"><span data-stu-id="90984-131">This example is based on an ASP.NET Core MVC application generated from a Visual Studio 2017 project template.</span></span>

### <a name="referencing-the-package"></a><span data-ttu-id="90984-132">パッケージを参照します。</span><span class="sxs-lookup"><span data-stu-id="90984-132">Referencing the package</span></span>

<span data-ttu-id="90984-133">参照を追加、 `OrchardCore.Localization.Core` NuGet パッケージです。</span><span class="sxs-lookup"><span data-stu-id="90984-133">Add a reference to the `OrchardCore.Localization.Core` NuGet package.</span></span> <span data-ttu-id="90984-134">使用可能になる[MyGet](https://www.myget.org/)次のパッケージ ソースで: https://www.myget.org/F/orchardcore-preview/api/v3/index.json</span><span class="sxs-lookup"><span data-stu-id="90984-134">It is available on [MyGet](https://www.myget.org/) at the following package source: https://www.myget.org/F/orchardcore-preview/api/v3/index.json</span></span>

<span data-ttu-id="90984-135">*.Csproj*ファイルにはここで、次のような行が含まれています (バージョン番号が異なる場合があります)。</span><span class="sxs-lookup"><span data-stu-id="90984-135">The *.csproj* file now contains a line similar to the following (version number may vary):</span></span>

[!code-xml[Main](localization/sample/POLocalization/POLocalization.csproj?range=9)]

### <a name="registering-the-service"></a><span data-ttu-id="90984-136">サービスを登録します。</span><span class="sxs-lookup"><span data-stu-id="90984-136">Registering the service</span></span>

<span data-ttu-id="90984-137">必要なサービスを追加、`ConfigureServices`メソッドの*Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="90984-137">Add the required services to the `ConfigureServices` method of *Startup.cs*:</span></span>

[!code-csharp[Main](localization/sample/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

<span data-ttu-id="90984-138">必要なミドルウェアを追加、`Configure`メソッドの*Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="90984-138">Add the required middleware to the `Configure` method of *Startup.cs*:</span></span>

[!code-csharp[Main](localization/sample/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

<span data-ttu-id="90984-139">選択肢の Razor ビューには、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="90984-139">Add the following code to your Razor view of choice.</span></span> <span data-ttu-id="90984-140">*About.cshtml*は、この例で使用します。</span><span class="sxs-lookup"><span data-stu-id="90984-140">*About.cshtml* is used in this example.</span></span>

[!code-cshtml[Main](localization/sample/POLocalization/Views/Home/About.cshtml)]

<span data-ttu-id="90984-141">`IViewLocalizer`インスタンスが挿入され、テキスト"Hello world!"を変換するために使用します。</span><span class="sxs-lookup"><span data-stu-id="90984-141">An `IViewLocalizer` instance is injected and used to translate the text "Hello world!".</span></span>

### <a name="creating-a-po-file"></a><span data-ttu-id="90984-142">PO ファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="90984-142">Creating a PO file</span></span>

<span data-ttu-id="90984-143">という名前のファイルを作成する *<culture code>.po*アプリケーション ルート フォルダー。</span><span class="sxs-lookup"><span data-stu-id="90984-143">Create a file named *<culture code>.po* in your application root folder.</span></span> <span data-ttu-id="90984-144">この例では、ファイル名は*fr.po*フランス語の言語が使用されるため。</span><span class="sxs-lookup"><span data-stu-id="90984-144">In this example, the file name is *fr.po* because the French language is used:</span></span>

[!code-text[Main](localization/sample/POLocalization/fr.po)]

<span data-ttu-id="90984-145">このファイルは、変換する文字列とフランス語に翻訳された文字列の両方を格納します。</span><span class="sxs-lookup"><span data-stu-id="90984-145">This file stores both the string to translate and the French-translated string.</span></span> <span data-ttu-id="90984-146">翻訳は、必要な場合、その親カルチャに戻します。</span><span class="sxs-lookup"><span data-stu-id="90984-146">Translations revert to their parent culture, if necessary.</span></span> <span data-ttu-id="90984-147">この例では、 *fr.po*カルチャが要求された場合、ファイルが使用される`fr-FR`または`fr-CA`です。</span><span class="sxs-lookup"><span data-stu-id="90984-147">In this example, the *fr.po* file is used if the requested culture is `fr-FR` or `fr-CA`.</span></span>

### <a name="testing-the-application"></a><span data-ttu-id="90984-148">アプリケーションのテスト</span><span class="sxs-lookup"><span data-stu-id="90984-148">Testing the application</span></span>

<span data-ttu-id="90984-149">アプリケーションを実行し、URL に移動して`/Home/About`です。</span><span class="sxs-lookup"><span data-stu-id="90984-149">Run your application, and navigate to the URL `/Home/About`.</span></span> <span data-ttu-id="90984-150">テキスト**Hello world!**</span><span class="sxs-lookup"><span data-stu-id="90984-150">The text **Hello world!**</span></span> <span data-ttu-id="90984-151">表示されます。</span><span class="sxs-lookup"><span data-stu-id="90984-151">is displayed.</span></span>

<span data-ttu-id="90984-152">URL に移動`/Home/About?culture=fr-FR`です。</span><span class="sxs-lookup"><span data-stu-id="90984-152">Navigate to the URL `/Home/About?culture=fr-FR`.</span></span> <span data-ttu-id="90984-153">テキスト**Bonjour le monde!**</span><span class="sxs-lookup"><span data-stu-id="90984-153">The text **Bonjour le monde!**</span></span> <span data-ttu-id="90984-154">表示されます。</span><span class="sxs-lookup"><span data-stu-id="90984-154">is displayed.</span></span>

## <a name="pluralization"></a><span data-ttu-id="90984-155">複数形化</span><span class="sxs-lookup"><span data-stu-id="90984-155">Pluralization</span></span>

<span data-ttu-id="90984-156">PO ファイルは、複数形化フォームをサポートし、これは、文字列と同じする必要がある変換基数に応じて異なる場合に役立ちます。</span><span class="sxs-lookup"><span data-stu-id="90984-156">PO files support pluralization forms, which is useful when the same string needs to be translated differently based on a cardinality.</span></span> <span data-ttu-id="90984-157">このタスクは、各言語が基数に基づいて、使用する文字列を選択するカスタム ルールを定義するため、複雑なが作成されます。</span><span class="sxs-lookup"><span data-stu-id="90984-157">This task is made complicated by the fact that each language defines custom rules to select which string to use based on the cardinality.</span></span>

<span data-ttu-id="90984-158">Orchard ローカリゼーション パッケージでは、異なるこれら複数形のフォームを自動的に起動する API を提供します。</span><span class="sxs-lookup"><span data-stu-id="90984-158">The Orchard Localization package provides an API to invoke these different plural forms automatically.</span></span>

### <a name="creating-pluralization-po-files"></a><span data-ttu-id="90984-159">複数形化 PO ファイルの作成</span><span class="sxs-lookup"><span data-stu-id="90984-159">Creating pluralization PO files</span></span>

<span data-ttu-id="90984-160">次の内容を追加する前に説明した*fr.po*ファイル。</span><span class="sxs-lookup"><span data-stu-id="90984-160">Add the following content to the previously mentioned *fr.po* file:</span></span>

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

<span data-ttu-id="90984-161">参照してください[PO ファイルは?](#what-is-a-po-file)この例では、各エントリが表す内容の説明。</span><span class="sxs-lookup"><span data-stu-id="90984-161">See [What is a PO file?](#what-is-a-po-file) for an explanation of what each entry in this example represents.</span></span>

### <a name="adding-a-language-using-different-pluralization-forms"></a><span data-ttu-id="90984-162">異なる複数形化フォームを使用する言語を追加します。</span><span class="sxs-lookup"><span data-stu-id="90984-162">Adding a language using different pluralization forms</span></span>

<span data-ttu-id="90984-163">英語とフランス語の文字列は、前の例で使用されました。</span><span class="sxs-lookup"><span data-stu-id="90984-163">English and French strings were used in the previous example.</span></span> <span data-ttu-id="90984-164">英語とフランス語複数形化の 2 つの形式があり、同じ形式の規則を共有するいずれかの基数が最初の複数形にマップされています。</span><span class="sxs-lookup"><span data-stu-id="90984-164">English and French have only two pluralization forms and share the same form rules, which is that a cardinality of one is mapped to the first plural form.</span></span> <span data-ttu-id="90984-165">その他の任意の基数は、2 番目の複数形にマップされます。</span><span class="sxs-lookup"><span data-stu-id="90984-165">Any other cardinality is mapped to the second plural form.</span></span>

<span data-ttu-id="90984-166">すべての言語では、同じ規則を共有します。</span><span class="sxs-lookup"><span data-stu-id="90984-166">Not all languages share the same rules.</span></span> <span data-ttu-id="90984-167">これを複数形の 3 つの形式を持つ、チェコ語の言語を示します。</span><span class="sxs-lookup"><span data-stu-id="90984-167">This is illustrated with the Czech language, which has three plural forms.</span></span>

<span data-ttu-id="90984-168">作成、`cs.po`次のように、ファイルし、複数形化が次の 3 つの異なる翻訳必要がありますに注意してください。</span><span class="sxs-lookup"><span data-stu-id="90984-168">Create the `cs.po` file as follows, and note how the pluralization needs three different translations:</span></span>

[!code-text[Main](localization/sample/POLocalization/cs.po)]

<span data-ttu-id="90984-169">チェコ語のローカライズ版は、そのまま追加`"cs"`でサポートされているカルチャの一覧に、`ConfigureServices`メソッド。</span><span class="sxs-lookup"><span data-stu-id="90984-169">To accept Czech localizations, add `"cs"` to the list of supported cultures in the `ConfigureServices` method:</span></span>

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

<span data-ttu-id="90984-170">編集、 *Views/Home/About.cshtml*いくつかの基数のローカライズされた、複数形の文字列をレンダリングするファイル。</span><span class="sxs-lookup"><span data-stu-id="90984-170">Edit the *Views/Home/About.cshtml* file to render localized, plural strings for several cardinalities:</span></span>

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

<span data-ttu-id="90984-171">**注:**カウントを表すに実際のシナリオで変数が使用します。</span><span class="sxs-lookup"><span data-stu-id="90984-171">**Note:** In a real world scenario, a variable would be used to represent the count.</span></span> <span data-ttu-id="90984-172">ここでは、非常に特殊なケースを公開する 3 つの異なる値を使用して同じコードを繰り返します。</span><span class="sxs-lookup"><span data-stu-id="90984-172">Here, we repeat the same code with three different values to expose a very specific case.</span></span>

<span data-ttu-id="90984-173">カルチャを切り替えると、次をご覧ください。</span><span class="sxs-lookup"><span data-stu-id="90984-173">Upon switching cultures, you see the following:</span></span>

<span data-ttu-id="90984-174">`/Home/About`の場合:</span><span class="sxs-lookup"><span data-stu-id="90984-174">For `/Home/About`:</span></span>

```html
There is one item.
There are 2 items.
There are 5 items.
```

<span data-ttu-id="90984-175">`/Home/About?culture=fr`の場合:</span><span class="sxs-lookup"><span data-stu-id="90984-175">For `/Home/About?culture=fr`:</span></span>

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

<span data-ttu-id="90984-176">`/Home/About?culture=cs`の場合:</span><span class="sxs-lookup"><span data-stu-id="90984-176">For `/Home/About?culture=cs`:</span></span>

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

<span data-ttu-id="90984-177">チェコ語のカルチャの 3 つの翻訳が異なることに注意してください。</span><span class="sxs-lookup"><span data-stu-id="90984-177">Note that for the Czech culture, the three translations are different.</span></span> <span data-ttu-id="90984-178">フランス語と英語のカルチャでは、2 つの最後の翻訳された文字列を同じ構造を共有します。</span><span class="sxs-lookup"><span data-stu-id="90984-178">The French and English cultures share the same construction for the two last translated strings.</span></span>

## <a name="advanced-tasks"></a><span data-ttu-id="90984-179">高度なタスク</span><span class="sxs-lookup"><span data-stu-id="90984-179">Advanced tasks</span></span>

### <a name="contextualizing-strings"></a><span data-ttu-id="90984-180">文字列を付けた</span><span class="sxs-lookup"><span data-stu-id="90984-180">Contextualizing strings</span></span>

<span data-ttu-id="90984-181">アプリケーションは、多くの場合、複数の場所で変換するために、文字列を含んでいます。</span><span class="sxs-lookup"><span data-stu-id="90984-181">Applications often contain the strings to be translated in several places.</span></span> <span data-ttu-id="90984-182">同じ文字列では、アプリ (Razor ビューまたはクラス ファイル) 内の特定の場所に別の翻訳があります。</span><span class="sxs-lookup"><span data-stu-id="90984-182">The same string may have a different translation in certain locations within an app (Razor views or class files).</span></span> <span data-ttu-id="90984-183">PO ファイルは、表される文字列の分類に使用することができます、ファイルのコンテキストの概念をサポートします。</span><span class="sxs-lookup"><span data-stu-id="90984-183">A PO file supports the notion of a file context, which can be used to categorize the string being represented.</span></span> <span data-ttu-id="90984-184">ファイルのコンテキストを使用して、文字列を翻訳できますが異なり、ファイルのコンテキスト (またはファイルのコンテキストが不足している) によって異なります。</span><span class="sxs-lookup"><span data-stu-id="90984-184">Using a file context, a string can be translated differently, depending on the file context (or lack of a file context).</span></span>

<span data-ttu-id="90984-185">PO ローカリゼーション サービスは、完全クラスまたは文字列を変換するときに使用されるビューの名前を使用します。</span><span class="sxs-lookup"><span data-stu-id="90984-185">The PO localization services use the name of the full class or the view that is used when translating a string.</span></span> <span data-ttu-id="90984-186">これは、値の設定、`msgctxt`エントリです。</span><span class="sxs-lookup"><span data-stu-id="90984-186">This is accomplished by setting the value on the `msgctxt` entry.</span></span>

<span data-ttu-id="90984-187">前に、マイナーの追加を検討してください*fr.po*例です。</span><span class="sxs-lookup"><span data-stu-id="90984-187">Consider a minor addition to the previous *fr.po* example.</span></span> <span data-ttu-id="90984-188">Razor ビューにある*Views/Home/About.cshtml*予約を設定して、ファイルのコンテキストとして定義できる`msgctxt`エントリの値。</span><span class="sxs-lookup"><span data-stu-id="90984-188">A Razor view located at *Views/Home/About.cshtml* can be defined as the file context by setting the reserved `msgctxt` entry's value:</span></span>

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

<span data-ttu-id="90984-189">`msgctxt`よう設定すると、テキストが変換に移動するとき`/Home/About?culture=fr-FR`です。</span><span class="sxs-lookup"><span data-stu-id="90984-189">With the `msgctxt` set as such, text translation occurs when navigating to `/Home/About?culture=fr-FR`.</span></span> <span data-ttu-id="90984-190">移動するときに、変換が発生しない`/Home/Contact?culture=fr-FR`です。</span><span class="sxs-lookup"><span data-stu-id="90984-190">The translation won't occur when navigating to `/Home/Contact?culture=fr-FR`.</span></span>

<span data-ttu-id="90984-191">指定されたファイルのコンテキストで特定のエントリが一致しないときに、Orchard コアの代替機構は、コンテキストなしの適切なの PO ファイルを探します。</span><span class="sxs-lookup"><span data-stu-id="90984-191">When no specific entry is matched with a given file context, Orchard Core's fallback mechanism looks for an appropriate PO file without a context.</span></span> <span data-ttu-id="90984-192">に対して定義されている特定のファイル コンテキストがあると仮定した場合は*Views/Home/Contact.cshtml*に間を移動する、`/Home/Contact?culture=fr-FR`など PO ファイルを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="90984-192">Assuming there is no specific file context defined for *Views/Home/Contact.cshtml*, navigating to `/Home/Contact?culture=fr-FR` loads a PO file such as:</span></span>

[!code-text[Main](localization/sample/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a><span data-ttu-id="90984-193">PO ファイルの場所を変更します。</span><span class="sxs-lookup"><span data-stu-id="90984-193">Changing the location of PO files</span></span>

<span data-ttu-id="90984-194">PO ファイルの既定の場所で変更できます`ConfigureServices`:</span><span class="sxs-lookup"><span data-stu-id="90984-194">The default location of PO files can be changed in `ConfigureServices`:</span></span>

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

<span data-ttu-id="90984-195">この例では、PO ファイルが読み込まれた、*ローカリゼーション*フォルダーです。</span><span class="sxs-lookup"><span data-stu-id="90984-195">In this example, the PO files are loaded from the *Localization* folder.</span></span>

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a><span data-ttu-id="90984-196">ローカライズされたファイルを検索するためのカスタム ロジックを実装します。</span><span class="sxs-lookup"><span data-stu-id="90984-196">Implementing a custom logic for finding localization files</span></span>

<span data-ttu-id="90984-197">PO ファイルを検索するより複雑なロジックが必要なときに、`OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider`インターフェイスは実装されており、サービスとして登録されていることができます。</span><span class="sxs-lookup"><span data-stu-id="90984-197">When more complex logic is needed to locate PO files, the `OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` interface can be implemented and registered as a service.</span></span> <span data-ttu-id="90984-198">これは、PO ファイルは、さまざまな場所に保存することができる場合、またはファイルをフォルダー階層内にある必要があるときに便利です。</span><span class="sxs-lookup"><span data-stu-id="90984-198">This is useful when PO files can be stored in varying locations or when the files have to be found within a hierarchy of folders.</span></span>

### <a name="using-a-different-default-pluralized-language"></a><span data-ttu-id="90984-199">複数化ごとに異なる既定言語を使用してください。</span><span class="sxs-lookup"><span data-stu-id="90984-199">Using a different default pluralized language</span></span>

<span data-ttu-id="90984-200">パッケージに含まれる、`Plural`は複数形の 2 つの形式に固有の拡張メソッド。</span><span class="sxs-lookup"><span data-stu-id="90984-200">The package includes a `Plural` extension method that is specific to two plural forms.</span></span> <span data-ttu-id="90984-201">他の複数形のフォームを必要とする言語では、拡張メソッドを作成します。</span><span class="sxs-lookup"><span data-stu-id="90984-201">For languages requiring more plural forms, create an extension method.</span></span> <span data-ttu-id="90984-202">既定の言語のすべてのローカライズ ファイルを提供する必要はありません、拡張メソッドを持つ&mdash;元の文字列は既に、コード内で直接使用できます。</span><span class="sxs-lookup"><span data-stu-id="90984-202">With an extension method, you won't need to provide any localization file for the default language &mdash; the original strings are already available directly in the code.</span></span>

<span data-ttu-id="90984-203">汎用的なを使用する`Plural(int count, string[] pluralForms, params object[] arguments)`を翻訳の文字列配列を受け入れるオーバー ロードします。</span><span class="sxs-lookup"><span data-stu-id="90984-203">You can use the more generic `Plural(int count, string[] pluralForms, params object[] arguments)` overload which accepts a string array of translations.</span></span>