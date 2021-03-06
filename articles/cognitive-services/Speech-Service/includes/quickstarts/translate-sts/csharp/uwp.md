---
title: クイック スタート:音声間で翻訳する、C# (UWP) - Speech サービス
titleSuffix: Azure Cognitive Services
description: TBD
services: cognitive-services
author: lisaweixu
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 12/09/2019
ms.author: erhopf
ms.topic: include
ms.openlocfilehash: a0f62db319d54c2db71a86f621985a304dbbb4d2
ms.sourcegitcommit: 9ee0cbaf3a67f9c7442b79f5ae2e97a4dfc8227b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "78925363"
---
## <a name="prerequisites"></a>前提条件

開始する前に、必ず次のことを行ってください。

> [!div class="checklist"]
> * [Azure Speech リソースを作成する](../../../../get-started.md)
> * [開発環境を設定し、空のプロジェクトを作成する](../../../../quickstarts/setup-platform.md?tabs=uwp)

## <a name="add-sample-code"></a>サンプル コードを追加する

今度は、アプリケーションのユーザー インターフェイスを定義する XAML コードを追加し、C# のコードビハインド実装を追加します。

1. **ソリューション エクスプローラー**で、`MainPage.xaml` を開きます。

1. デザイナーの XAML ビューで、次の XAML スニペットを **Grid** タグに挿入します (`<Grid>` と `</Grid>` の間)。

   [!code-xml[UI elements](~/samples-cognitive-services-speech-sdk/quickstart/csharp/uwp/translate-speech-to-text/helloworld/MainPage.xaml#StackPanel)]

1. **ソリューション エクスプローラー**で、コードビハインドのソース ファイル `MainPage.xaml.cs` を開きます (`MainPage.xaml` にグループ化されています)。

1. その中のすべてのコードを次のスニペットに置き換えます。

   [!code-csharp[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/csharp/uwp/translate-speech-to-text/helloworld/MainPage.xaml.cs#code)]

1. このファイルの `SpeechTranslationFromMicrophone_ButtonClicked` ハンドラーから `YourSubscriptionKey` という文字列を見つけて、ご利用のサブスクリプション キーに置き換えます。

1. `SpeechTranslationFromMicrophone_ButtonClicked` ハンドラーから `YourServiceRegion` という文字列を見つけて、ご利用のサブスクリプションに関連付けられている[リージョン](~/articles/cognitive-services/Speech-Service/regions.md)に置き換えます。 たとえば、無料試用版サブスクリプションの場合は `westus` を使用します。

1. メニュー バーから **[ファイル]**  >  **[すべて保存]** を選択して変更内容を保存します。

## <a name="build-and-run-the-application"></a>アプリケーションの構築と実行

アプリケーションをビルドしてテストする準備が整いました。

1. メニュー バーから **[ビルド]**  >  **[ソリューションのビルド]** を選択してアプリケーションをビルドします。 これで、コードは、エラーなしでコンパイルされます。

1. **[デバッグ]**  >  **[デバッグの開始]** の順に選択するか、**F5** キーを押して、アプリケーションを起動します。 **[helloworld]** ウィンドウが表示されます。

   ![サンプル UWP 翻訳アプリケーション (C#) - クイックスタート](~/articles/cognitive-services/Speech-Service/media/sdk/qs-translate-speech-uwp-helloworld-window.png)

1. **[Enable Microphone]** を選択し、アクセス許可要求がポップアップ表示されたら、 **[はい]** を選択します。

   ![マイクへのアクセス許可要求](~/articles/cognitive-services/Speech-Service/media/sdk/qs-csharp-uwp-10-access-prompt.png)

1. **[Translate speech from the microphone input]** を選択し、デバイスのマイクに向かって英語のフレーズや文章を話します。 その音声が Speech Services に転送されて別の言語 (このケースではドイツ語) のテキストに翻訳されます。 翻訳済みのテキストが Speech Services からアプリケーションに返されてウィンドウに表示されます。

   ![音声翻訳ユーザー インターフェイス](~/articles/cognitive-services/Speech-Service/media/sdk/qs-translate-csharp-uwp-ui-result.png)

## <a name="next-steps"></a>次のステップ

[!INCLUDE [footer](./footer.md)]
