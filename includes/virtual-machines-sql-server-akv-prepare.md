---
title: include 文件
description: include 文件
services: virtual-machines-windows
author: rothja
manager: craigg
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: 01020a23b102c896bbeb3d8cf455afabfc164917
ms.sourcegitcommit: f24fdd1ab23927c73595c960d8a26a74e1d12f5d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/27/2019
ms.locfileid: "58505902"
---
## <a name="prepare-for-akv-integration"></a>准备 AKV 集成
若要使用 Azure Key Vault 集成来配置 SQL Server VM，有以下几个先决条件： 

1. [安装 Azure PowerShell](#install)
2. [创建 Azure Active Directory](#register)
3. [创建密钥保管库](#createkeyvault)

以下各节描述了这些先决条件，以及稍后运行 PowerShell cmdlet 需要收集的信息。

[!INCLUDE [updated-for-az](./updated-for-az.md)]

### <a id="install"></a> 安装 Azure PowerShell
请确保已安装最新的 Azure PowerShell SDK。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/powershell/azure/install-az-ps)。

### <a id="register"></a>将应用程序注册到 Azure Active Directory

首先，订阅中需要有 [Azure Active Directory](https://azure.microsoft.com/trial/get-started-active-directory/) (AAD)。 其优点之一是允许为特定用户和应用程序授予对密钥保管库的权限。

然后，将应用程序注册到 AAD。 这会提供一个服务主体帐户，使你有权访问 VM 所需的密钥保管库。 在 Azure 密钥保管库文章中，可以找到这些步骤[应用程序注册到 Azure Active Directory](../articles/key-vault/key-vault-manage-with-cli2.md#registering-an-application-with-azure-active-directory)部分中，或您可以看到这些步骤中的屏幕快照以及**获取标识的应用程序部分**的[这篇博客文章](http://blogs.technet.com/b/kv/archive/2015/01/09/azure-key-vault-step-by-step.aspx)。 在完成这些步骤之前，需要在注册期间收集以下信息，之后在 SQL VM 上启用 Azure Key Vault 集成时需要这些信息。

* 添加应用程序后，在“已注册应用”边栏选项卡上找到“应用程序 ID”。
    稍后会将该应用程序 ID 分配给 PowerShell 脚本中的 $spName（服务主体名称）参数，以启用 Azure 密钥保管库集成。

   ![应用程序 ID](./media/virtual-machines-sql-server-akv-prepare/aad-application-id.png)

* 在执行这些步骤期间，请在创建密钥时复制密钥的密码，如下面的屏幕截图中所示。 稍后会将此密钥密码分配给 PowerShell 脚本中的 $spSecret（服务主体密码）参数。

   ![AAD 密码](./media/virtual-machines-sql-server-akv-prepare/aad-sp-secret.png)

* 应用程序 ID 和机密将还可用于在 SQL Server 中创建凭据。

* 必须为此新的客户端 ID 授予以下访问权限：**获取**、**密钥换行**、**取消密钥换行**。 可通过 [Set-AzKeyVaultAccessPolicy](https://docs.microsoft.com/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) cmdlet 实现此操作。 有关详细信息，请参阅 [Azure 密钥保管库概述](../articles/key-vault/key-vault-overview.md)。

### <a id="createkeyvault"></a>创建密钥保管库
若要使用 Azure Key Vault 来存储将用于在 VM 中加密的密钥，将需要对密钥保管库的访问权限。 如果尚未设置密钥保管库，请按照[开始使用 Azure Key Vault](../articles/key-vault/key-vault-overview.md) 一文中的步骤创建一个。 在完成这些步骤之前，需要在设置期间收集一些信息，之后在 SQL VM 上启用 Azure Key Vault 集成时需要这些信息。

    New-AzKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia'

进行创建密钥保管库的步骤时，请注意返回的 vaultUri 属性，它是密钥保管库 URL。 下面显示了该步骤中提供的示例，其中的密钥保管库名称是 ContosoKeyVault，因此密钥保管库 URL 为 https://contosokeyvault.vault.azure.net/。

稍后会将该密钥保管库 URL 分配给 PowerShell 脚本中的 $akvURL 参数，以启用 Azure Key Vault 集成。

创建密钥保管库后，需要向密钥保管库添加密钥，稍后在 SQL Server 中创建非对称密钥时，会引用此密钥。
