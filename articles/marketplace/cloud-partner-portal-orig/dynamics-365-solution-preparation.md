---
title: Dynamics 365 ソリューションの準備
description: コンポーネントのパッケージ化、インストール、アンインストールのためのフレームワーク
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: dsindona
ms.openlocfilehash: ac1e4fa541e945f20904ced114a36b58d14585ba
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "80278588"
---
# <a name="dynamics-365-solution-preparation"></a>Dynamics 365 ソリューションの準備

Dynamics 365 ソリューション システムは、特定のビジネス機能を提供するコンポーネントのパッケージ化、インストール、アンインストールのためのフレームワークです。 ソリューションは、作成した拡張機能を配布する ISV およびその他の Microsoft Dynamics 365 パートナーによって使用されます。

既存の Dynamics 365 (xRM) ISV である方は、既に管理ソリューションを作成済みで、solution.zip ファイルをお持ちの場合がほとんどです。 自社のソリューションの "Display Name (表示名)" と "Description (説明)" フィールドに、お客様向けに表示する内容が反映されていることをご確認ください。 これらは、CRM Online 管理センターで表示されます。

![CRMScreenShot1](media/CRMScreenShot1.png)

_**注:** 次のパッケージの例では、ソリューション名を "SampleSolution.zip" と仮定します_

新しい ISV は、[https://msdn.microsoft.com/library/gg334530.aspx](https://msdn.microsoft.com/library/gg334530.aspx) で、ソリューションの作成に関する詳細情報を得ることができます。

ソリューションにサポート データが必要な場合:

* テスト環境でサンプル データを作成します
* 構成移行ツールを使用して、データ用の比較規則を含めたスキーマを作成します。
* 構成のスキーマをプロジェクト ファイルに保存します。 構成データを更新する場合、後でこれが必要になります。
* 構成データをエクスポートします。 エクスポート ファイルには、エクスポート内容がわかるような名前を付けます。
