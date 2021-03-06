---
title: 使用 Azure 映像生成器 （预览版） 创建 Linux VM
description: 使用 Azure 映像生成器创建 Linux VM。
author: cynthn
ms.author: cynthn
ms.date: 05/02/2019
ms.topic: article
ms.service: virtual-machines-linux
manager: jeconnoc
ms.openlocfilehash: 854645af95d780053d94668921e41ac189bbbfb7
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/06/2019
ms.locfileid: "65159505"
---
# <a name="preview-create-a-linux-vm-with-azure-image-builder"></a>预览版：使用 Azure 映像生成器创建 Linux VM

本文演示了如何创建使用 Azure 映像生成器和 Azure CLI 的自定义的 Linux 映像。 这篇文章中的示例使用三个不同[定制员](image-builder-json.md#properties-customize)的自定义映像：

- Shell (ScriptUri)-下载和运行[shell 脚本](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/customizeScript.sh)。
- Shell （内联）-运行特定命令。 在此示例中，内联命令包括创建目录和更新操作系统。
- 文件的副本[来自 GitHub 的文件](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/exampleArtifacts/buildArtifacts/index.html)到 VM 上的目录。

我们将使用示例.json 模板要配置的映像。 下面是我们正在使用的.json 文件： [helloImageTemplateLinux.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Linux_Managed_Image/helloImageTemplateLinux.json)。 

> [!IMPORTANT]
> Azure 映像生成器目前处于公共预览状态。
> 此预览版在提供时没有附带服务级别协议，不建议将其用于生产工作负荷。 某些功能可能不受支持或者受限。 有关详细信息，请参阅 [Microsoft Azure 预览版补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="register-the-features"></a>注册功能
若要在预览期间使用 Azure 映像生成器，您需要注册新的功能。

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

检查功能注册状态。

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

检查您的注册。

```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState

az provider show -n Microsoft.Storage | grep registrationState
```

如果他们未说已注册，运行以下命令：

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages

az provider register -n Microsoft.Storage
```

## <a name="create-a-resource-group"></a>创建资源组

我们将使用一些部分信息重复，因此我们将创建一些变量来存储该信息。


```azurecli-interactive
# Resource group name - we are using myImageBuilderRG in this example
imageResourceGroup=myImageBuilerRGLinux
# Datacenter location - we are using West US 2 in this example
location=WestUS2
# Name for the image - we are using myBuilderImage in this example
imageName=myBuilderImage
# Run output name
runOutputName=aibLinux
```

为订阅 id。 创建一个变量 可以获取此使用`az account show | grep id`。

```azurecli-interactive
subscriptionID=<Your subscription ID>
```

创建资源组。

```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```


授予该资源组中创建资源的映像生成器权限。 `--assignee`值是图像生成器服务的应用程序注册 ID。 

```azurecli-interactive
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```

## <a name="download-the-json-example"></a>下载.json 示例

下载示例.json 文件，并将其配置与你创建的变量。

```azurecli-interactive
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Linux_Managed_Image/helloImageTemplateLinux.json -o helloImageTemplateLinux.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateLinux.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateLinux.json
sed -i -e "s/<region>/$location/g" helloImageTemplateLinux.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateLinux.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateLinux.json
```

## <a name="create-the-image"></a>创建映像
映像的配置提交到的 VM 映像生成器服务

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateLinux.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateLinux01
```

开始创建映像。

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Run 
```

等待生成完成。 这可能需要大约 15 分钟。


## <a name="create-the-vm"></a>创建 VM

创建使用你生成的映像的 VM。

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name myVM \
  --admin-username azureuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

创建 VM 的输出中获取的 IP 地址，并用它来通过 ssh 连接到 VM。

```azurecli-interactive
ssh azureuser@<pubIp>
```

你应看到该映像在一天，一旦建立 SSH 连接的消息使用自定义 ！

```console

*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

类型`exit`完成以关闭 SSH 连接。

## <a name="check-the-source"></a>请检查源

在映像生成器模板，在属性中，你将看到源映像、 自定义对其编写脚本运行，并分发的位置。

```azurecli-interactive
cat helloImageTemplateLinux.json
```

有关更详细的有关该.json 文件的信息，请参阅[图像生成器模板参考](image-builder-json.md)

## <a name="clean-up"></a>清理

完成后，删除的资源。

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateLinux01

az group delete -n $imageResourceGroup
```


## <a name="next-steps"></a>后续步骤

若要了解有关本文中使用的.json 文件的组件的详细信息，请参阅[映像生成器模板引用](image-builder-json.md)。