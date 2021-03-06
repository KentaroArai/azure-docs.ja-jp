---
title: Logic Apps REST API を使用して統合サービス環境 (ISE) を作成する
description: Azure Logic Apps から Azure 仮想ネットワーク (VNET) にアクセスできるように、Logic Apps REST API を使用して統合サービス環境 (ISE) を作成します。
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/11/2020
ms.openlocfilehash: 2c6e35b1e7d160064998004f87c5b14d0eaeac5e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "79127662"
---
# <a name="create-an-integration-service-environment-ise-by-using-the-logic-apps-rest-api"></a>Logic Apps REST API を使用して統合サービス環境 (ISE) を作成する

この記事では、ロジック アプリと統合アカウントで [Azure 仮想ネットワーク](../virtual-network/virtual-networks-overview.md)にアクセスする必要があるシナリオ向けに、Logic Apps REST API を使用して[*統合サービス環境 (ISE)* ](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) を作成する方法を説明します。 ISE は、専用のストレージなど、"グローバル" なマルチテナント Logic Apps サービスとは別に確保されているリソースを使用する分離環境です。 この分離で、他の Azure テナントがご利用のアプリのパフォーマンスに与える可能性がある影響も軽減されます。 ISE には、独自の静的 IP アドレスも用意されています。 これらの IP アドレスは、パブリックのマルチテナント サービスのロジック アプリによって共有される静的 IP アドレスとは別のものです。

代わりに Azure portal を使用して ISE を作成する方法については、「[Azure Logic Apps から Azure 仮想ネットワークに接続する](../logic-apps/connect-virtual-network-vnet-isolated-environment.md)」を参照してください。

> [!IMPORTANT]
> ISE 内で実行されるロジック アプリ、組み込みトリガー、組み込みアクション、およびコネクターでは、使用量ベースの価格プランとは異なる価格プランが使用されます。 ISE の価格と課金のしくみについては、「[固定価格モデル](../logic-apps/logic-apps-pricing.md#fixed-pricing)」を参照してください。 価格については、[Logic Apps の価格](../logic-apps/logic-apps-pricing.md)に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件

* Azure portal で ISE を作成する場合と同様の、[前提条件](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#prerequisites)および [ISE のアクセスを有効にするための要件](../logic-apps/connect-virtual-network-vnet-isolated-environment.md#enable-access)

* HTTPS PUT 要求で Logic Apps REST API を呼び出して ISE を作成するために使用できるツール。 たとえば、[Postman](https://www.getpostman.com/downloads/) を使用したり、このタスクを実行するロジック アプリを構築したりできます。

## <a name="send-the-request"></a>要求を送信する

Logic Apps REST API を呼び出して ISE を作成するには、この HTTPS PUT 要求を行います。

`PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/integrationServiceEnvironments/{integrationServiceEnvironmentName}?api-version=2019-05-01`

> [!IMPORTANT]
> Logic Apps REST API 2019-05-01 バージョンでは、ISE コネクタに対して独自の HTTP PUT 要求を行う必要があります。

通常、デプロイは 2 時間以内に完了します。 状況によっては、最大でデプロイに 4 時間かかる場合があります。 デプロイの状態を確認するには、[Azure portal](https://portal.azure.com) の Azure ツール バーで、通知アイコンを選択します。これで通知ペインが開きます。

> [!NOTE]
> デプロイが失敗するか、ISE を削除する場合、サブネットがリリースされるまでに最長 1 時間かかる可能性があります。 この遅延のため、このようなサブネットを他の ISE で再利用できるようになるまで待たなければならない場合があります。
>
> 仮想ネットワークを削除すると、通常、サブネットがリリースされるまでに最長 2 時間かかる可能性があり、この操作にさらに時間がかかる場合があります。 
> 仮想ネットワークを削除する場合は、まだ接続されているリソースがないことを確認してください。 
> 「[仮想ネットワークの削除](../virtual-network/manage-virtual-network.md#delete-a-virtual-network)」を参照してください。

## <a name="request-header"></a>要求ヘッダー

要求ヘッダーに次のプロパティを含めます。

* `Content-type`:このプロパティ値は `application/json` に設定します。

* `Authorization`:このプロパティ値は、使用する Azure サブスクリプションまたはリソース グループにアクセスする顧客のベアラー トークンに設定します。

### <a name="request-body-syntax"></a>要求本文の構文

ISE の作成時に使用するプロパティを記述する要求本文の構文を次に示します。

```json
{
   "id": "/subscriptions/{Azure-subscription-ID/resourceGroups/{Azure-resource-group}/providers/Microsoft.Logic/integrationServiceEnvironments/{ISE-name}",
   "name": "{ISE-name}",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "{Azure-region}",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-1}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-2}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-3}",
            },
            {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Network/virtualNetworks/{virtual-network-name}/subnets/{subnet-4}",
            }
         ]
      }
   }
}
```

### <a name="request-body-example"></a>要求本文の例

この例の要求本文では、サンプルの値を示しています。

```json
{
   "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Logic/integrationServiceEnvironments/Fabrikam-ISE",
   "name": "Fabrikam-ISE",
   "type": "Microsoft.Logic/integrationServiceEnvironments",
   "location": "WestUS2",
   "sku": {
      "name": "Premium",
      "capacity": 1
   },
   "properties": {
      "networkConfiguration": {
         "accessEndpoint": {
            // Your ISE can use the "External" or "Internal" endpoint. This example uses "External".
            "type": "External"
         },
         "subnets": [
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-1",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-2",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-3",
            },
            {
               "id": "/subscriptions/********************/resourceGroups/Fabrikam-RG/providers/Microsoft.Network/virtualNetworks/Fabrikam-VNET/subnets/subnet-4",
            }
         ]
      }
   }
}
```

## <a name="next-steps"></a>次のステップ

* [統合サービス環境にリソースを追加する](../logic-apps/add-artifacts-integration-service-environment-ise.md)
* [統合サービス環境を管理する](../logic-apps/ise-manage-integration-service-environment.md#check-network-health)

