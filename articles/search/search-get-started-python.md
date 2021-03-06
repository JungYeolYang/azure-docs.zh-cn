---
title: 快速入门：Python 和 REST Api-Azure 搜索
description: 创建、 加载和使用 Python、 Jupyter Notebook 和 Azure 搜索 REST API 查询索引。
ms.date: 05/15/2019
author: heidisteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 1ab6bb069f60f4d2dbb4cfaecda54c3c2ef20adc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806426"
---
# <a name="quickstart-create-an-azure-search-index-using-jupyter-python-notebooks"></a>快速入门：创建 Azure 搜索索引中使用 Jupyter 的 Python 笔记本
> [!div class="op_single_selector"]
> * [Python (REST)](search-get-started-python.md)
> * [PowerShell (REST)](search-create-index-rest-api.md)
> * [C#](search-create-index-dotnet.md)
> * [Postman (REST)](search-fiddler.md)
> * [门户](search-create-index-portal.md)
> 

生成创建、 加载和查询 Azure 搜索的 Jupyter notebook[索引](search-what-is-an-index.md)使用 Python 和[Azure 搜索服务 REST Api](https://docs.microsoft.com/rest/api/searchservice/)。 本文介绍如何构建您自己的笔记本执行步骤的。 （可选） 可以运行已完成的笔记本。 若要下载副本，请转到[Azure 搜索-python 示例存储库](https://github.com/Azure-Samples/azure-search-python-samples)。

如果还没有 Azure 订阅，请在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)，然后[注册 Azure 搜索](search-create-service-portal.md)。

## <a name="prerequisites"></a>必备组件

本快速入门使用以下服务和工具。 

+ [创建 Azure 搜索服务](search-create-service-portal.md)或在当前订阅下[查找现有服务](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。 可以使用本快速入门的免费服务。 

+ [Anaconda 3.x](https://www.anaconda.com/distribution/#download-section)，提供 Python 3.x 和 Jupyter Notebook。

## <a name="get-a-key-and-url"></a>获取密钥和 URL

REST 调用需要在每个请求中使用服务 URL 和访问密钥。 搜索服务是使用这二者创建的，因此，如果向订阅添加了 Azure 搜索，则请按以下步骤获取必需信息：

1. [登录到 Azure 门户](https://portal.azure.com/)，在搜索服务的“概述”页中获取 URL。 示例终结点可能类似于 `https://mydemo.search.windows.net`。

1. 在“设置” > “密钥”中，获取有关该服务的完全权限的管理员密钥。 有两个可交换的管理员密钥，为保证业务连续性而提供，以防需要滚动一个密钥。 可以在请求中使用主要或辅助密钥来添加、修改和删除对象。

![获取 HTTP 终结点和访问密钥](media/search-fiddler/get-url-key.png "Get an HTTP endpoint and access key")

所有请求对发送到服务的每个请求都需要 API 密钥。 具有有效的密钥可以在发送请求的应用程序与处理请求的服务之间建立信任关系，这种信任关系以每个请求为基础。

## <a name="connect-to-azure-search"></a>连接到 Azure 搜索

打开 Jupyter notebook，并通过请求服务上的索引列表来验证从本地工作站的连接。 在与 Anaconda3 Windows，可以使用 Anaconda 导航器来启动笔记本。

1. 创建新的 Python3 笔记本。

1. 在第一个单元格，加载用于处理 JSON 和构建 HTTP 请求使用的库。

   ```python
   import json
   import requests
   from pprint import pprint
   ```

1. 在第二个单元格中，输入将为每个请求中的常量的请求元素。 搜索服务名称 （您的搜索的服务的名称） 和管理员 API 密钥 （您的管理的 API 密钥） 替换为有效的值。 

   ```python
    endpoint = 'https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/'
    api_version = '?api-version=2019-05-06'
    headers = {'Content-Type': 'application/json',
           'api-key': '<YOUR-ADMIN-API-KEY>' }
   ```

1. 在第三个单元格中，表述请求。 此 GET 请求以搜索服务的索引集合为目标，并选择名称属性。

   ```python
   url = endpoint + "indexes" + api_version + "&$select=name"
   response  = requests.get(url, headers=headers)
   index_list = response.json()
   pprint(index_list)
   ```

1. 运行每个步骤。 如果存在索引，则响应将包含索引的列表。 下面的屏幕截图，该服务包括 azureblob 索引和 realestate-我们的示例索引。

   ![向 Azure 搜索请求的 HTTP 的 Jupyter 笔记本中的 Python 脚本](media/search-get-started-python/connect-azure-search.png "向 Azure 搜索请求的 HTTP 的 Jupyter 笔记本中的 Python 脚本")

   空索引集合返回此响应： `{'@odata.context': 'https://mydemo.search.windows.net/$metadata#indexes(name)', 'value': []}`

> [!Tip]
> 上一项免费服务，你被限制为三个索引、 索引器和数据源。 本快速入门中创建各项之一。 请确保有空间来进行进一步操作之前创建新对象。

## <a name="1---create-an-index"></a>1 - 创建索引

除非使用门户，可以加载数据之前，服务上必须存在索引。 此步骤中使用[创建索引 REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)推送到该服务的索引架构

字段集合定义的结构*文档*。 所需的元素的索引包含一个名称和字段集合。 每个字段有一个名称、 类型和特性，以确定如何使用它 (例如，是否是全文索引时才可搜索、 筛选或可在搜索结果中检索)。 一个索引，其中一个类型的字段中`Edm.String`必须指定为*密钥*文档标识。

此索引名为"hotels 的上一年度"，并且请参阅下面的字段定义。 它是更大的子集[Hotels 索引](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/Hotels_IndexDefinition.JSON)在其他演练中使用。 我们修整它在此快速入门中为了简单起见。


1. 在下一步的单元中，将下面的示例粘贴到单元格来提供架构。 

    ```python
    index_schema = {
       "name": "hotels-py",  
       "fields": [
         {"name": "HotelId", "type": "Edm.String", "key": "true", "filterable": "true"},
         {"name": "HotelName", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "true", "facetable": "false"},
         {"name": "Description", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "false", "facetable": "false", "analyzer": "en.lucene"},
         {"name": "Description_fr", "type": "Edm.String", "searchable": "true", "filterable": "false", "sortable": "false", "facetable": "false", "analyzer": "fr.lucene"},
         {"name": "Category", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Tags", "type": "Collection(Edm.String)", "searchable": "true", "filterable": "true", "sortable": "false", "facetable": "true"},
         {"name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Rating", "type": "Edm.Double", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Address", "type": "Edm.ComplexType", 
         "fields": [
         {"name": "StreetAddress", "type": "Edm.String", "filterable": "false", "sortable": "false", "facetable": "false", "searchable": "true"},
         {"name": "City", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "StateProvince", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "PostalCode", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"},
         {"name": "Country", "type": "Edm.String", "searchable": "true", "filterable": "true", "sortable": "true", "facetable": "true"}
        ]
       }
      ]
    }
    ```

2. 在另一个单元格中，表述请求。 该 PUT 请求以搜索服务的索引集合为目标，并创建根据你在上一步中提供的索引架构的索引。

   ```python
   url = endpoint + "indexes" + api_version
   response  = requests.post(url, headers=headers, json=index_schema)
   index = response.json()
   pprint(index)
   ```

3. 运行每个步骤。

   响应包括的架构的 JSON 表示形式。 下面的屏幕截图修剪索引架构的部分，以便您可以看到多个响应。

    ![请求创建索引](media/search-get-started-python/create-index.png "请求创建索引")

> [!Tip]
> 可能还检查索引列表在门户中，或重新运行服务连接请求以确定进行验证， *hotels py*索引集合中列出的索引。

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2 - 加载文档

若要将推送文档，请使用对索引的 URL 终结点的 HTTP POST 请求。 REST api[添加、 更新或删除文档](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)。 文档是源自[HotelsData](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotels/HotelsData_toAzureSearch.JSON) GitHub 上。

1. 在新的单元格，提供了三个符合索引架构的文档。 指定每个文档的上传操作。

    ```python
    documents = {
        "value": [
        {
        "@search.action": "upload",
        "HotelId": "1",
        "HotelName": "Secret Point Motel",
        "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
        "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
        "Category": "Boutique",
        "Tags": [ "pool", "air conditioning", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1970-01-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "677 5th Ave",
            "City": "New York",
            "StateProvince": "NY",
            "PostalCode": "10022",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "2",
        "HotelName": "Twin Dome Motel",
        "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Boutique",
        "Tags": [ "pool", "free wifi", "concierge" ],
        "ParkingIncluded": "false",
        "LastRenovationDate": "1979-02-18T00:00:00Z",
        "Rating": 3.60,
        "Address": {
            "StreetAddress": "140 University Town Center Dr",
            "City": "Sarasota",
            "StateProvince": "FL",
            "PostalCode": "34243",
            "Country": "USA"
            }
        },
        {
        "@search.action": "upload",
        "HotelId": "3",
        "HotelName": "Triple Landscape Hotel",
        "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
        "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
        "Category": "Resort and Spa",
        "Tags": [ "air conditioning", "bar", "continental breakfast" ],
        "ParkingIncluded": "true",
        "LastRenovationDate": "2015-09-20T00:00:00Z",
        "Rating": 4.80,
        "Address": {
            "StreetAddress": "3393 Peachtree Rd",
            "City": "Atlanta",
            "StateProvince": "GA",
            "PostalCode": "30326",
            "Country": "USA"
        }
      }
     ]
    }
    ```

2. 在另一个单元格中，表述请求。 此 POST 请求以酒店 py 索引的文档集合为目标，并将推送上一步中提供的文档。

   ```python
   url = endpoint + "indexes/hotels-py/docs/index" + api_version
   response  = requests.post(url, headers=headers, json=documents)
   index_content = response.json()
   pprint(index_content)
   ```

3. 运行每个步骤，若要将文档推送到你的搜索服务中的索引。 结果应类似于下面的示例。 

   ```
   {'@odata.context': "https://mydemo.search.windows.net/indexes('hotels-py')/$metadata#Collection(Microsoft.Azure.Search.V2019_05_06.IndexResult)",
    'value': [{'errorMessage': None,
            'key': '1',
            'status': True,
            'statusCode': 201},
           {'errorMessage': None,
            'key': '2',
            'status': True,
            'statusCode': 201},
           {'errorMessage': None,
            'key': '3',
            'status': True,
            'statusCode': 201}]}
     ```


## <a name="3---search-an-index"></a>3 - 搜索索引

此步骤说明如何使用索引进行查询[搜索文档 REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents)。


1. 在新的单元格，提供了一个查询表达式。 下面的示例搜索术语"酒店"和"wifi"。 它还会返回*计数*的文档的匹配，并*选择*要在搜索结果中包括哪些字段。

   ```python
   searchstring = '&search=hotels wifi&$count=true&$select=HotelId,HotelName'
   ```

2. 明确表述请求。 此 GET 请求以酒店 py 索引的文档集合为目标，并将附加上一步中指定的查询。

   ```python
   url = endpoint + "indexes/hotels-py/docs" + api_version + searchstring
   response  = requests.get(url, headers=headers, json=searchstring)
   query = response.json()
   pprint(query)
   ```

   结果应类似于以下输出。

   ```
   {'@odata.context': "https://mydemo.search.windows.net/indexes('hotels-py')/$metadata#docs(*)",
    '@odata.count': 3,
    'value': [{'@search.score': 1.0,
               'HotelId': '1',
               'HotelName': 'Secret Point Motel'},
              {'@search.score': 1.0,
               'HotelId': '2',
               'HotelName': 'Twin Dome Motel'},
              {'@search.score': 1.0,
               'HotelId': '3',
               'HotelName': 'Triple Landscape Hotel'}]}
   ```

3. 请尝试几个其他查询示例，若要了解的语法。 可以应用筛选器、 执行前两个结果，按特定字段中，订购或 

   + `searchstring = '&search=*&$filter=Rating gt 4&$select=HotelId,HotelName,Description'`

   + `searchstring = '&search=hotel&$top=2&$select=HotelId,HotelName,Description'`

   + `searchstring = '&search=pool&$orderby=Address/City&$select=HotelId, HotelName, Address/City, Address/StateProvince'`

## <a name="clean-up"></a>清理 

如果不再需要应删除该索引。 一项免费服务被限制为三个索引。 您可能想要删除任何不主动使用为其他教程留出空间的索引。

   ```python
  url = endpoint + "indexes/hotels-py" + api_version
  response  = requests.delete(url, headers=headers)
   ```

返回现有索引的列表，可以验证索引删除。 如果酒店 py 消失了，就会知道您已成功的请求。

```python
url = endpoint + "indexes" + api_version + "&$select=name"

response  = requests.get(url, headers=headers)
index_list = response.json()
pprint(index_list)
```

## <a name="next-steps"></a>后续步骤

了解有关查询语法和方案的详细信息。

> [!div class="nextstepaction"]
> [创建基本查询](search-query-overview.md)
