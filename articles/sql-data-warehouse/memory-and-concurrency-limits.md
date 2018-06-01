---
title: 内存和并发限制 - Azure SQL 数据仓库 | Microsoft Docs
description: 查看分配给 Azure SQL 数据仓库中的各个性能级别和资源类的内存和并发限制。
services: sql-data-warehouse
author: ronortloff
manager: craigg-msft
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 05/07/2018
ms.author: rortloff
ms.reviewer: igorstan
ms.openlocfilehash: 46d41e3ee85deb20f189bc9c82a255178f3d7eee
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2018
ms.locfileid: "33942246"
---
# <a name="memory-and-concurrency-limits-for-azure-sql-data-warehouse"></a>Azure SQL 数据仓库的内存和并发限制
查看分配给 Azure SQL 数据仓库中的各个性能级别和资源类的内存和并发限制。 若要了解详细信息并将这些功能应用于你的工作负荷管理计划，请参阅[用于工作负荷管理的资源类](resource-classes-for-workload-management.md)。 

目前有两代 SQL 数据仓库：第 1 代和第 2 代。 建议使用 SQL 数据仓库第 2 代来实现数据仓库工作负荷的最佳性能。 第 2 代引入了一种新的 NVMe 固态磁盘缓存，可将最常访问的数据保留在 CPU 附近。 这样可以为最占用资源和要求最苛刻的工作负荷免除远程 I/O。 除性能外，第 2 代还提供最高级别的扩展能力，允许你扩展至 30,000 个数据仓库单位，并提供无限制的列式存储。 我们仍支持上一代（第 1 代）SQL 数据仓库并保留相同的功能；不过，建议你尽早[升级到第 2 代](upgrade-to-latest-generation.md)。 

## <a name="data-warehouse-capacity-settings"></a>数据仓库容量设置
以下各表显示了不同性能级别的数据仓库的最大容量。 若要更改性能级别，请参阅[缩放计算 - 门户](quickstart-scale-compute-portal.md)。

### <a name="gen2"></a>第 2 代

第 2 代为每个查询提供的内存比第 1 代多 2.5 倍。 额外的内存有助于第 2 代提供快速的性能。  第 2 代的性能级别范围为 DW1000c 到 DW30000c。 

| 性能级别 | 计算节点 | 每个计算节点的分布区数 | 每个数据仓库的内存 (GB) |
|:-----------------:|:-------------:|:------------------------------:|:------------------------------:|
| DW1000c           | 2             | 30                             |   600                          |
| DW1500c           | 3             | 20                             |   900                          |
| DW2000c           | 4             | 15                             |  1200                          |
| DW2500c           | 5             | 12                             |  1500                          |
| DW3000c           | 6             | 10                             |  1800                          |
| DW5000c           | 10            | 6                              |  3000                          |
| DW6000c           | 12            | 5                              |  3600                          |
| DW7500c           | 15            | 4                              |  4500                          |
| DW10000c          | 20            | 3                              |  6000                          |
| DW15000c          | 30            | 2                              |  9000                          |
| DW30000c          | 60            | 1                              | 18000                          |

第 2 代的最大 DWU 为 DW30000c，包括 60 个计算节点，每个计算节点有一个分布区。 例如，DW30000c 级别的 600 TB 数据仓库的每个计算节点可以处理大约 10 TB 数据。

### <a name="gen1"></a>第 1 代

第 1 代的服务级别范围为 DW100 到 DW6000。 

| 性能级别 | 计算节点 | 每个计算节点的分布区数 | 每个数据仓库的内存 (GB) |
|:-----------------:|:-------------:|:------------------------------:|:------------------------------:|
| DW100             | 1             | 60                             |  24                            |
| DW200             | 2             | 30                             |  48                            |
| DW300             | 3             | 20                             |  72                            |
| DW400             | 4             | 15                             |  96                            |
| DW500             | 5             | 12                             | 120                            |
| DW600             | 6             | 10                             | 144                            |
| DW1000            | 10            | 6                              | 240                            |
| DW1200            | 12            | 5                              | 288                            |
| DW1500            | 15            | 4                              | 360                            |
| DW2000            | 20            | 3                              | 480                            |
| DW3000            | 30            | 2                              | 720                            |
| DW6000            | 60            | 1                              | 1440                           |

## <a name="concurrency-maximums"></a>并发性最大值
为了确保每个查询都有足够的资源来有效执行，SQL 数据仓库会通过向每个查询分配并发槽位来跟踪资源利用率。 系统将查询放入某个队列，然后，查询在队列中等待有足够的[并发槽位](resource-classes-for-workload-management.md#concurrency-slots)可用。 并发槽位还确定 CPU 优先级。 有关详细信息，请参阅[分析工作负荷](analyze-your-workload.md)。

### <a name="gen2"></a>第 2 代
 
**静态资源类**

下表显示了每个[静态资源类](resource-classes-for-workload-management.md)的最大并发查询数和并发槽位数。  

| 服务级别 | 并发查询数上限 | 可用的并发槽位数 |staticrc10 | staticrc20 | staticrc30 | staticrc40 | staticrc50 | staticrc60 | staticrc70 | staticrc80 |
|:-------------:|:--------------------------:|:---------------------------:|:---------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|
| DW1000c       | 32                         |   40                        | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW1500c       | 32                         |   60                        | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW2000c       | 48                         |   80                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW2500c       | 48                         |  100                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW3000c       | 64                         |  120                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW5000c       | 64                         |  200                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW6000c       | 128                        |  240                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW7500c       | 128                        |  300                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW10000c      | 128                        |  400                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW15000c      | 128                        |  600                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |
| DW30000c      | 128                        | 1200                        | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |

**动态资源类**

> [!NOTE]
> 第 2 代的 smallrc 资源类随着服务级别的增加而动态增加内存，并且最多只支持 32 个并发查询。  随着服务级别的增加，smallrc 使用的并发槽位和内存也会增加。 
>
>

下表显示了每个[动态资源类](resource-classes-for-workload-management.md)的最大并发查询数和并发槽位数。 与第 1 代不同，第 2 代的动态资源类真正地实现了动态。  第 2 代对所有服务级别中的 small-medium-large-xlarge 资源类使用 3-10-22-70 内存百分比分配。

| 服务级别 | 并发查询数上限 | 可用的并发槽位数 | smallrc 使用的槽数 | mediumrc 使用的槽数 | largerc 使用的槽数 | xlargerc 使用的槽数 |
|:-------------:|:--------------------------:|:---------------------------:|:---------------------:|:----------------------:|:---------------------:|:----------------------:|
| DW1000c       | 32                         |   40                        | 1                     |  4                     |  8                    |  28                    |
| DW1500c       | 32                         |   60                        | 1                     |  6                     |  13                   |  42                    |
| DW2000c       | 32                         |   80                        | 2                     |  8                     |  17                   |  56                    |
| DW2500c       | 32                         |  100                        | 3                     | 10                     |  22                   |  70                    |
| DW3000c       | 32                         |  120                        | 3                     | 12                     |  26                   |  84                    |
| DW5000c       | 32                         |  200                        | 6                     | 20                     |  44                   | 140                    |
| DW6000c       | 32                         |  240                        | 7                     | 24                     |  52                   | 168                    |
| DW7500c       | 32                         |  300                        | 9                     | 30                     |  66                   | 210                    |
| DW10000c      | 32                         |  400                        | 12                    | 40                     |  88                   | 280                    |
| DW15000c      | 32                         |  600                        | 18                    | 60                     | 132                   | 420                    |
| DW30000c      | 32                         | 1200                        | 36                    | 120                    | 264                   | 840                    |



#### <a name="gen1"></a>第 1 代

静态资源类

下表显示了**第 1 代**每个[静态资源类](resource-classes-for-workload-management.md)的最大并发查询数和并发槽位数。

| 服务级别 | 并发查询数上限 | 最大并发槽位数 |staticrc10 | staticrc20 | staticrc30 | staticrc40 | staticrc50 | staticrc60 | staticrc70 | staticrc80 |
|:-------------:|:--------------------------:|:-------------------------:|:---------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|:----------:|
| DW100         | 4                          |   4                       | 1         | 2          | 4          | 4          |  4         |  4         |  4         |   4        |
| DW200         | 8                          |   8                       | 1         | 2          | 4          | 8          |  8         |  8         |  8         |   8        |
| DW300         | 12                         |  12                       | 1         | 2          | 4          | 8          |  8         |  8         |  8         |   8        |
| DW400         | 16                         |  16                       | 1         | 2          | 4          | 8          | 16         | 16         | 16         |  16        |
| DW500         | 20                         |  20                       | 1         | 2          | 4          | 8          | 16         | 16         | 16         |  16        |
| DW600         | 24                         |  24                       | 1         | 2          | 4          | 8          | 16         | 16         | 16         |  16        |
| DW1000        | 32                         |  40                       | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW1200        | 32                         |  48                       | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW1500        | 32                         |  60                       | 1         | 2          | 4          | 8          | 16         | 32         | 32         |  32        |
| DW2000        | 48                         |  80                       | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW3000        | 64                         | 120                       | 1         | 2          | 4          | 8          | 16         | 32         | 64         |  64        |
| DW6000        | 128                        | 240                       | 1         | 2          | 4          | 8          | 16         | 32         | 64         | 128        |

动态资源类
> [!NOTE]
> 第 1 代的 smallrc 资源类为每个查询分配固定数量的内存，方式类似于静态资源类 staticrc10。  由于 smallrc 是静态的，因此它可以扩展到 128 个并发查询。 
>
>

下表显示了**第 1 代**每个[动态资源类](resource-classes-for-workload-management.md)的最大并发查询数和并发槽位数。

| 服务级别 | 并发查询数上限 | 可用的并发槽位数 | smallrc | mediumrc | largerc | xlargerc |
|:-------------:|:--------------------------:|:---------------------------:|:-------:|:--------:|:-------:|:--------:|
| DW100         |  4                         |   4                         | 1       |  1       |  2      |   4      |
| DW200         |  8                         |   8                         | 1       |  2       |  4      |   8      |
| DW300         | 12                         |  12                         | 1       |  2       |  4      |   8      |
| DW400         | 16                         |  16                         | 1       |  4       |  8      |  16      |
| DW500         | 20                         |  20                         | 1       |  4       |  8      |  16      |
| DW600         | 24                         |  24                         | 1       |  4       |  8      |  16      |
| DW1000        | 32                         |  40                         | 1       |  8       | 16      |  32      |
| DW1200        | 32                         |  48                         | 1       |  8       | 16      |  32      |
| DW1500        | 32                         |  60                         | 1       |  8       | 16      |  32      |
| DW2000        | 48                         |  80                         | 1       | 16       | 32      |  64      |
| DW3000        | 64                         | 120                         | 1       | 16       | 32      |  64      |
| DW6000        | 128                        | 240                         | 1       | 32       | 64      | 128      |


满足其中一个阈值时，就会按“先进先出”原则排队执行新查询。  如果查询已完成并且查询数和槽位数低于限制，则 Azure SQL 数据仓库会释放排队的查询。 

## <a name="next-steps"></a>后续步骤

若要详细了解如何利用资源类来进一步优化工作负荷，请查看以下文章：
* [用于工作负荷管理的资源类](resource-classes-for-workload-management.md)
* [分析工作负荷](analyze-your-workload.md)
