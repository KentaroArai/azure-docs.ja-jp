---
title: チュートリアル - Azure PowerShell を使用してカスタム VM イメージを作成する
description: このチュートリアルでは、Azure PowerShell を使って Azure にカスタム仮想マシン イメージを作成する方法について説明します
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 7360798f2f95184145a856babf501e3080cbaaf4
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/24/2020
ms.locfileid: "76274201"
---
# <a name="tutorial-create-a-custom-image-of-an-azure-vm-with-azure-powershell"></a>チュートリアル: Azure PowerShell を使用して Azure VM のカスタム イメージを作成する

カスタム イメージは Marketplace のイメージに似ていますが、カスタム イメージは自分で作成します。 カスタム イメージを使用してデプロイのブートストラップを実行し、複数の VM で一貫性を確保することができます。 このチュートリアルでは、PowerShell を使用して Azure 仮想マシンの独自のカスタム イメージを作成します。 学習内容は次のとおりです。

> [!div class="checklist"]
> * VM を sysprep して汎用化する
> * カスタム イメージを作成する
> * カスタム イメージから VM を作成する
> * サブスクリプション内のすべてのイメージを一覧表示する
> * イメージを削除する

パブリック プレビューでは、[Azure VM Image Builder](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder-overview) サービスが含まれています。 テンプレートにカスタマイズを記述するだけで、この記事のイメージ作成手順が処理されます。 [Azure Image Builder (プレビュー) をお試しください](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder)。

## <a name="before-you-begin"></a>開始する前に

以下の手順では、既存の VM を取得し、新しい VM インスタンスの作成に使用できる再利用可能なカスタム イメージに変換する方法について詳しく説明します。

このチュートリアルの例を完了するには、既存の仮想マシンが必要です。 必要に応じて、この[サンプル スクリプト](../scripts/virtual-machines-windows-powershell-sample-create-vm.md)で仮想マシンを作成できます。 このチュートリアルを実行するときは、リソース グループと VM の名前を適宜置き換えてください。

## <a name="launch-azure-cloud-shell"></a>Azure Cloud Shell を起動する

Azure Cloud Shell は無料のインタラクティブ シェルです。この記事の手順は、Azure Cloud Shell を使って実行することができます。 一般的な Azure ツールが事前にインストールされており、アカウントで使用できるように構成されています。 

Cloud Shell を開くには、コード ブロックの右上隅にある **[使ってみる]** を選択します。 [https://shell.azure.com/powershell](https://shell.azure.com/powershell) に移動して、別のブラウザー タブで Cloud Shell を起動することもできます。 **[コピー]** を選択してコードのブロックをコピーし、Cloud Shell に貼り付けてから、Enter キーを押して実行します。

## <a name="prepare-vm"></a>VM の準備

仮想マシンのイメージを作成するには、ソース VM を一般化して割り当てを解除し、Azure で一般化済みとマークすることにより、ソース VM を準備する必要があります。

### <a name="generalize-the-windows-vm-using-sysprep"></a>Sysprep を使用して Windows VM を一般化する

特に重要な点は、Sysprep がすべての個人アカウント情報を削除して、マシンをイメージとして使用できるように準備することです。 Sysprep の詳細については、「 [Sysprep の使用方法: 紹介](https://technet.microsoft.com/library/bb457073.aspx)」を参照してください。


1. 仮想マシンへの接続
2. 管理者としてコマンド プロンプト ウィンドウを開きます。 ディレクトリを *%windir%\system32\sysprep* に変更し、`sysprep.exe` を実行します。
3. **[システム準備ツール]** ダイアログ ボックスで **[システムの OOBE (Out-of-Box Experience) に入る]** を選択し、 **[一般化する]** チェック ボックスがオンになっていることを確認します。
4. **[シャットダウン オプション]** の **[シャットダウン]** を選択し、 **[OK]** をクリックします。
5. Sysprep は完了時に仮想マシンをシャットダウンします。 **VM は再起動しないでください**。

### <a name="deallocate-and-mark-the-vm-as-generalized"></a>VM の割り当てを解除して VM を汎用としてマークする

イメージを作成するには、VM の割り当てを解除し、Azure で VM を一般化済みとしてマークする必要があります。

[Stop-AzVM](https://docs.microsoft.com/powershell/module/az.compute/stop-azvm) を使用して VM の割り当てを解除します。

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName myResourceGroup `
   -Name myVM -Force
```

`-Generalized`Set-AzVm[ を使用して、仮想マシンの状態を ](https://docs.microsoft.com/powershell/module/az.compute/set-azvm) に設定します。 
   
```azurepowershell-interactive
Set-AzVM `
   -ResourceGroupName myResourceGroup `
   -Name myVM -Generalized
```


## <a name="create-the-image"></a>イメージの作成

[New-AzImageConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azimageconfig) と [New-AzImage](https://docs.microsoft.com/powershell/module/az.compute/new-azimage) を使用して、VM のイメージを作成できるようになりました。 次の例では、*myVM* という名前の VM から *myImage* という名前のイメージを作成します。

仮想マシンを取得します。 

```azurepowershell-interactive
$vm = Get-AzVM `
   -Name myVM `
   -ResourceGroupName myResourceGroup
```

イメージの設定を作成します。

```azurepowershell-interactive
$image = New-AzImageConfig `
   -Location EastUS `
   -SourceVirtualMachineId $vm.ID 
```

イメージを作成します。

```azurepowershell-interactive
New-AzImage `
   -Image $image `
   -ImageName myImage `
   -ResourceGroupName myResourceGroup
``` 

 
## <a name="create-vms-from-the-image"></a>イメージからの VM の作成

イメージが用意できたので、イメージから新しい VM を 1 つ以上作成できます。 カスタム イメージからの VM の作成は、Marketplace イメージを使用した VM の作成に似ています。 Marketplace イメージを使うときは、イメージ、イメージ プロバイダー、プラン、SKU、バージョンに関する情報を提供する必要があります。 リソース グループが同じであれば、[New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) コマンドレットに設定されている簡易パラメーターを利用し、カスタム イメージの名前を指定する必要があります。 

この例では、*myResourceGroup* で *myImage* イメージから *myVMfromImage* という名前の VM を作成します。


```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVMfromImage" `
    -ImageName "myImage" `
    -Location "East US" `
    -VirtualNetworkName "myImageVnet" `
    -SubnetName "myImageSubnet" `
    -SecurityGroupName "myImageNSG" `
    -PublicIpAddressName "myImagePIP" `
    -OpenPorts 3389
```

1 つのイメージからの同時デプロイ数は 20 VM を上限とするようお勧めします。 同じカスタム イメージからの 20 VM を超える大規模な同時デプロイを予定している場合は、複数のイメージ レプリカを含んだ [Shared Image Gallery](shared-image-galleries.md) を使用してください。 


## <a name="image-management"></a>イメージの管理 

ここでは、一般的なマネージド イメージ タスクと PowerShell を使用してそれらを完了する方法の例をいくつか示します。

すべてのイメージの名前を一覧表示します。

```azurepowershell-interactive
$images = Get-AzResource -ResourceType Microsoft.Compute/images 
$images.name
```

イメージを削除します。 この例では、*myImage* という名前のイメージを *myResourceGroup* から削除します。

```azurepowershell-interactive
Remove-AzImage `
    -ImageName myImage `
    -ResourceGroupName myResourceGroup
```

## <a name="next-steps"></a>次のステップ

このチュートリアルでは、カスタム VM イメージを作成しました。 以下の方法を学習しました。

> [!div class="checklist"]
> * VM を sysprep して汎用化する
> * カスタム イメージを作成する
> * カスタム イメージから VM を作成する
> * サブスクリプション内のすべてのイメージを一覧表示する
> * イメージを削除する

次のチュートリアルに進み、高可用性仮想マシンを作成する方法について学習してください。

> [!div class="nextstepaction"]
> [高可用性 VM の作成](tutorial-availability-sets.md)



