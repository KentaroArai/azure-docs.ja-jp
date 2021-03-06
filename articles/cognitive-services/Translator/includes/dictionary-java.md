---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.author: erhopf
ms.openlocfilehash: 63c524880a47c6e519649bd871a6216d6faeefce
ms.sourcegitcommit: 9ee0cbaf3a67f9c7442b79f5ae2e97a4dfc8227b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "69907084"
---
[!INCLUDE [Prerequisites](prerequisites-java.md)]

[!INCLUDE [Setup and use environment variables](setup-env-variables.md)]

## <a name="initialize-a-project-with-gradle"></a>Gradle を使用してプロジェクトを初期化する

まずは、このプロジェクトの作業ディレクトリを作成しましょう。 コマンド ライン (またはターミナル) から、次のコマンドを実行します。

```console
mkdir alt-translation-sample
cd alt-translation-sample
```

次に、Gradle プロジェクトを初期化します。 次のコマンドを実行すると、Gradle の重要なビルド ファイルが作成されます。特に重要なのは `build.gradle.kts` です。これは、アプリケーションを作成して構成するために、実行時に使用されます。 作業ディレクトリから次のコマンドを実行します。

```console
gradle init --type basic
```

**DSL** を選択するよう求められたら、**Kotlin** を選択します。

## <a name="configure-the-build-file"></a>ビルド ファイルを構成する

`build.gradle.kts` の場所を特定し、好みの IDE またはテキスト エディターで開きます。 その後、次のビルド構成をコピーします。

```
plugins {
    java
    application
}
application {
    mainClassName = "AltTranslation"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("com.squareup.okhttp:okhttp:2.5.0")
    compile("com.google.code.gson:gson:2.8.5")
}
```

このサンプルでは、HTTP 要求の処理に OkHttp を使用し、JSON の処理と解析に Gson を使用していることに注意してください。 ビルド構成について詳しくは、「[Creating New Gradle Builds](https://guides.gradle.org/creating-new-gradle-builds/)」(新しい Gradle ビルドの作成) をご覧ください。

## <a name="create-a-java-file"></a>Java ファイルを作成する

サンプル アプリ用のフォルダーを作成しましょう。 作業ディレクトリから、次のコマンドを実行します。

```console
mkdir -p src\main\java
```

次に、このフォルダー内に `AltTranslation.java` というファイルを作成します。

## <a name="import-required-libraries"></a>必要なライブラリをインポートする

`AltTranslation.java` を開き、次のインポート ステートメントを追加します。

```java
import java.io.*;
import java.net.*;
import java.util.*;
import com.google.gson.*;
import com.squareup.okhttp.*;
```


## <a name="define-variables"></a>変数の定義

まず、プロジェクトのパブリック クラスを作成する必要があります。

```java
public class AltTranslation {
  // All project code goes here...
}
```

次の行を `AltTranslation` クラスに追加します。 まず、サブスクリプション キーとエンドポイントが環境変数から読み取られます。 `api-version` には、`url` と共に別途 2 つのパラメーターが追加されていることがわかります。 翻訳の入力と出力は、これらのパラメーターを使用して設定します。 このサンプルでは、英語 (`en`) とスペイン語 (`es`) です。

```java
private static String subscriptionKey = System.getenv("TRANSLATOR_TEXT_SUBSCRIPTION_KEY");
private static String endpoint = System.getenv("TRANSLATOR_TEXT_ENDPOINT");
String url = endpoint + "/dictionary/lookup?api-version=3.0&from=en&to=es";
```

Cognitive Services のマルチサービス サブスクリプションを使用している場合は、要求のパラメーターに `Ocp-Apim-Subscription-Region` も含める必要があります。 [マルチサービス サブスクリプションを使用した認証の詳細を参照してください](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication)。

## <a name="create-a-client-and-build-a-request"></a>クライアントを作成して要求をビルドする

次の行を `AltTranslation` クラスに追加して、`OkHttpClient` をインスタンス化します。

```java
// Instantiates the OkHttpClient.
OkHttpClient client = new OkHttpClient();
```

次に、POST 要求をビルドしましょう。 翻訳するテキストは変更してかまいません。

```java
// This function performs a POST request.
public String Post() throws IOException {
    MediaType mediaType = MediaType.parse("application/json");
    RequestBody body = RequestBody.create(mediaType,
            "[{\n\t\"Text\": \"Pineapples\"\n}]");
    Request request = new Request.Builder()
            .url(url).post(body)
            .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey)
            .addHeader("Content-type", "application/json").build();
    Response response = client.newCall(request).execute();
    return response.body().string();
}
```

## <a name="create-a-function-to-parse-the-response"></a>応答を解析するための関数を作成する

このシンプルな関数は、Translator Text サービスからの JSON 応答を解析し、整形するものです。

```java
// This function prettifies the json response.
public static String prettify(String json_text) {
    JsonParser parser = new JsonParser();
    JsonElement json = parser.parse(json_text);
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    return gson.toJson(json);
}
```

## <a name="put-it-all-together"></a>すべてをまとめた配置

最後に、要求を実行して応答を取得します。 次の行をプロジェクトに追加します。

```java
public static void main(String[] args) {
    try {
        AltTranslation altTranslationRequest = new AltTranslation();
        String response = altTranslationRequest.Post();
        System.out.println(prettify(response));
    } catch (Exception e) {
        System.out.println(e);
    }
}
```

## <a name="run-the-sample-app"></a>サンプル アプリを実行する

以上で、サンプル アプリを実行する準備が整いました。 コマンド ライン (またはターミナル セッション) で作業ディレクトリのルートに移動して、次のコマンドを実行します。

```console
gradle build
```

ビルドが完了したら、次のコマンドを実行します。

```console
gradle run
```

## <a name="sample-response"></a>応答のサンプル

```json
[
  {
    "normalizedSource": "pineapples",
    "displaySource": "pineapples",
    "translations": [
      {
        "normalizedTarget": "piñas",
        "displayTarget": "piñas",
        "posTag": "NOUN",
        "confidence": 0.7016,
        "prefixWord": "",
        "backTranslations": [
          {
            "normalizedText": "pineapples",
            "displayText": "pineapples",
            "numExamples": 5,
            "frequencyCount": 158
          },
          {
            "normalizedText": "cones",
            "displayText": "cones",
            "numExamples": 5,
            "frequencyCount": 13
          },
          {
            "normalizedText": "piña",
            "displayText": "piña",
            "numExamples": 3,
            "frequencyCount": 5
          },
          {
            "normalizedText": "ganks",
            "displayText": "ganks",
            "numExamples": 2,
            "frequencyCount": 3
          }
        ]
      },
      {
        "normalizedTarget": "ananás",
        "displayTarget": "ananás",
        "posTag": "NOUN",
        "confidence": 0.2984,
        "prefixWord": "",
        "backTranslations": [
          {
            "normalizedText": "pineapples",
            "displayText": "pineapples",
            "numExamples": 2,
            "frequencyCount": 16
          }
        ]
      }
    ]
  }
]
```

## <a name="next-steps"></a>次のステップ

API のリファレンスを見て、Translator Text API でできるすべてのことを理解してください。

> [!div class="nextstepaction"]
> [API リファレンス](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
