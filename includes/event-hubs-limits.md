---
title: include 文件
description: include 文件
services: event-hubs
author: sethmanheim
ms.service: event-hubs
ms.topic: include
ms.date: 02/26/2018
ms.author: sethm
ms.custom: include file
ms.openlocfilehash: 9d6b54027adcf2b12c6ca4081a11208a31f620e8
ms.sourcegitcommit: 3102f886aa962842303c8753fe8fa5324a52834a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61464361"
---
下表列出了特定于 [Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)的配额和限制。 有关事件中心定价的信息，请参阅[事件中心定价](https://azure.microsoft.com/pricing/details/event-hubs/)。

| 限制 | 范围 | 说明 | 值 |
| --- | --- | --- | --- |
| 每个订阅的事件中心命名空间数 |订阅 |- |100 |
| 每个命名空间的事件中心数 |命名空间 |创建新事件中心的后续请求会被拒绝。 |10 |
| 每个事件中心的分区数 |实体 |- |32 |
| 每个事件中心的使用者组数 |实体 |- |20 |
| 每个命名空间的 AMQP 连接数 |命名空间 |系统会拒绝后续的附加连接请求，且调用代码会收到异常。 |5,000 |
| 事件中心事件的最大大小|实体 |- |1 MB |
| 事件中心名称的最大大小 |实体 |- |50 个字符 |
| 每个使用者组的非 epoch 接收者数 |实体 |- |5 |
| 事件数据的最长保留期限 |实体 |- |1-7 天 |
| 最大吞吐量单位 |命名空间 |超出吞吐量单位限制会导致数据受到限制，并生成[服务器忙异常](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception)。 若要请求更多的标准层的吞吐量单位，文件[支持请求](/azure/azure-supportability/how-to-create-azure-support-request)。 [额外的吞吐量单位](../articles/event-hubs/event-hubs-auto-inflate.md)将基于承诺的购买以大小为 20 个单位的块的形式提供。 |20 |
| 每个命名空间的授权规则数量 |命名空间|将拒绝后续的授权规则创建请求。|12 |
