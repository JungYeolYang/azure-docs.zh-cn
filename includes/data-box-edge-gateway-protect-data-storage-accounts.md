---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/16/2019
ms.author: alkohli
ms.openlocfilehash: 653c175a559f5c0b7dc551b396e91276332df20a
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "60754232"
---
你的设备都与存储帐户用作 Azure 中的数据目标相关联。 对存储帐户的访问控制按订阅和两个 512 位存储访问密钥与该存储帐户相关联。

当数据框边缘设备访问存储帐户时，其中一个密钥用于身份验证。 另一个密钥保留在保留，以便可以定期轮换密钥。

出于安全原因，很多数据中心要求密钥轮换。 我们建议遵循密钥轮换的以下最佳实践：

- 存储帐户密钥类似于存储帐户的根密码。 请小心保护你的帐户密钥。 不将密码分发给其他用户，硬编码，或将其保存任意位置以纯文本格式的其他人可以访问。
- [重新生成帐户密钥](../articles/storage/common/storage-account-manage.md#regenerate-access-keys)通过 Azure 门户，如果您认为可能会损害。
- Azure 管理员应该定期更改，或使用 Azure 门户的存储部分来直接访问存储帐户重新生成主密钥或辅助密钥。