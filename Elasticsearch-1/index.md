# Elasticsearch学习(一)


<!--more-->

## Elasticsearch介绍

Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

## Docker安装Elasticsearch

```bash
mkdir -p es/data es/plugins
vim es/docker-compose.yml
```

`docker-compose.yml`

```yaml
version: '3'

services:
  elasticsearch:
    image: elasticsearch:8.4.1
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms64m -Xmx512m"
      - "HTTP_HOST=0.0.0.0"
    volumes:
      - ./data:/usr/share/elasticsearch/data:rw
      - ./plugins:/usr/share/elasticsearch/plugins

  kibana:
    image: kibana:8.4.1
    ports:
      - "5601:5601"
# 本处用户名密码之后获取重新填写，注意不能使用elasticsearch账号
#    environment:
#      - "ELASTICSEARCH_USERNAME=kibana"
#      - "ELASTICSEARCH_PASSWORD=A4xbHdr20mgbcAkENhlA"
    depends_on:
      - elasticsearch
```

```bash
cd es
docker-compose up -d
docker exec -it es-elasticsearch-1 bin/elasticsearch-setup-passwords auto
```

该步骤是为每个账户随机生成密码，效果如下（elasticsearch-setup-passwords 在 8.0 版本被弃用，以后将被删除）

```bash
******************************************************************************
Note: The 'elasticsearch-setup-passwords' tool has been deprecated. This       command will be removed in a future release.
******************************************************************************

Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y


Changed password for user apm_system
PASSWORD apm_system = FloDE5YBuURSV0kc3fIk

Changed password for user kibana_system
PASSWORD kibana_system = A4xbHdr20mgbcAkENhlA

Changed password for user kibana  # 用于kibana组件获取相关信息用于web展示
PASSWORD kibana = A4xbHdr20mgbcAkENhlA

Changed password for user logstash_system
PASSWORD logstash_system = ry05C9RmhGvWALhtIYEE

Changed password for user beats_system
PASSWORD beats_system = 8XfsSkR3pz8H0IHgDGoJ

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = B79ufRafdqS4gisw32IF

Changed password for user elastic  # 超级管理员，拥有所有权限
PASSWORD elastic = RUikbaL7AvegIbk4xC4j
```

接着浏览器访问`http://127.0.0.1:9200`以及`http://127.0.0.1:5601`，输入上一步获取到的 elastic 账号密码即可访问

## 使用Elasticsearch API

**本文使用 Elasticsearch 8.4.1 版本，而 ES8 已去除 type，默认身份验证**

### _cat

1. `GET /_cat/nodes` 查看所有节点 
2. `GET /_cat/health` 查看 ES 健康状况
3. `GET /_cat/master` 查看主节点
4. `GET /_cat/indices` 查看所有索引

### 文档

#### 索引文档

##### 接口

`PUT /<target>/_doc/<_id>`

`POST /<target>/_doc/`

`PUT /<target>/_create/<_id>`

`POST /<target>/_create/<_id>`

路径参数：

1. **`<target>`**（必填，字符串）目标数据流或索引的名称。

   如果目标不存在且与数据流模板不匹配，则此请求将创建索引。

2. **`<_id>`**（可选，字符串）文档的唯一标识符。

   使用`POST /<target>/_doc/`将自动生成文档 ID

请求正文：**`<field>`**（必填，字符串）包含文档数据的 JSON 源。

`PUT`用于新建文档

`POST`用于更新文档

##### 实例

`POST /test/_doc`

请求正文：

```json
{
    "name": "sukun"
}
```

响应：201

```json
{
  "_index": "test", //文档添加到的索引的名称
  "_id": "sTOACIMByqhncvtAVIy1", //添加的文档的唯一标识符
  "_version": 1, //文档版本。每次更新文档时递增
  "result": "created", //索引操作的结果，created或updated
  "_shards": { //提供有关索引操作的复制过程的信息
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0, //序列号，用于开放式并发控制
  "_primary_term": 1 //主项，用于开放式并发控制
}
```

`PUT /test/_doc/sTOACIMByqhncvtAVIy1`

请求正文：

```json
{
    "name": "test"
}
```

响应：200

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 1,
    "_primary_term": 1
}
```

#### 查询文档

##### 接口

`GET <index>/_doc/<_id>`

`HEAD <index>/_doc/<_id>`

`GET <index>/_source/<_id>`

`HEAD <index>/_source/<_id>`

路径参数：

1. **`<index>`**（必填，字符串）包含文档的索引的名称。
2. **`<_id>`**（必填，字符串）文档的唯一标识符。

使用`GET`从特定索引中检索文档及其源或存储的字段。

使用`HEAD`验证文档是否存在。

使用`_source`仅检索文档源或验证它是否存在。

##### 实例

`GET <index>/_doc/<_id>`响应：200

```json
{
  "_index": "test",
  "_id": "sTOACIMByqhncvtAVIy1",
  "_version": 2,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true, //指示文档是否存在，true或false
  "_source": { //found为true，包含文档数据；_source为false或查询参数stored_fields为true，则排除。
    "name": "test"
  }
}
```

`HEAD <index>/_doc/<_id>`响应：200

`GET <index>/_source/<_id>`响应：200

```json
{
  "name": "test"
}
```

`HEAD <index>/_source/<_id>`响应：200

#### 开放式并发操作

##### 实现

**使用乐观锁**

为确保较旧版本的文档不会覆盖较新版本，对文档执行的每个操作都由协调该更改的主分片分配一个序列号。

序列号随着每个操作的增加而增加，因此较新的操作保证具有比旧操作更高的序列号。

然后，Elasticsearch 可以使用操作的序列号来确保较新的文档版本永远不会被分配了较小序列号的更改所覆盖。

Elasticsearch 会跟踪上次操作的序列号和主项，以更改其存储的每个文档。

序列号和主项：`_seq_no` `_primary_term`

在索引、更新、删除文档时，使用查询参数`if_seq_no` `if_primary_term`可以确保仅在检索文档后未对其进行任何其他更改时才更改文档。

##### 实例

`PUT /test/_doc/sTOACIMByqhncvtAVIy1?if_seq_no=1&if_primary_term=1`

请求正文：

```json
{
    "name": "sukun"
}
```

响应：200

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 3,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

紧接着再次执行相同操作（模拟并发）

`PUT /test/_doc/sTOACIMByqhncvtAVIy1?if_seq_no=1&if_primary_term=1`

请求正文：

```json
{
    "name": "sukun"
}
```

响应：409

```json
{
    "error": {
        "root_cause": [
            {
                "type": "version_conflict_engine_exception",
                "reason": "[sTOACIMByqhncvtAVIy1]: version conflict, required seqNo [1], primary term [1]. current document has seqNo [2] and primary term [1]",
                "index_uuid": "4yncACgnS0CVAWh0hz-LKA",
                "shard": "0",
                "index": "test"
            }
        ],
        "type": "version_conflict_engine_exception",
        "reason": "[sTOACIMByqhncvtAVIy1]: version conflict, required seqNo [1], primary term [1]. current document has seqNo [2] and primary term [1]",
        "index_uuid": "4yncACgnS0CVAWh0hz-LKA",
        "shard": "0",
        "index": "test"
    },
    "status": 409
}
```

#### 更新文档

##### 接口

`POST /<index>/_update/<_id>`

路径参数：

1. **`<index>`**（必填，字符串）目标索引的名称。默认情况下，如果索引不存在，则会自动创建索引。
2. **`<_id>`**（必填，字符串）要更新的文档的唯一标识符。

作用：用于编写文档更新脚本。该脚本可以**更新、删除或跳过修改**文档，还支持传递部分文档，该部分文档**合并**到现有文档中。（若要完全替换现有文档，请使用索引 API）

过程：

1. 从索引中获取文档（与分片并置）。
2. 运行指定的脚本。
3. 为结果编制索引。

##### 实例

###### 更新部分文档

`POST /test/_update/sTOACIMByqhncvtAVIy1`

请求正文：

```json
{
    "doc": {
        "new_name": "new_name"
    }
}
```

响应：200

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 4,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

再次查询该文档：

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 4,
    "_seq_no": 3,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "sukun",
        "new_name": "new_name"
    }
}
```

###### 检测noop更新

再次执行上述更新操作，响应：200

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 4,
    "result": "noop",  // 如果没设置在请求正文中设置"detect_noop": false，不更改任何内容的更新会返回no operation
    "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```

#### 删除文档&索引

##### 接口

`DELETE /<index>`

路径参数：

**`<index>`**（必填，字符串）要删除的以逗号分隔的索引列表。不能用索引别名，默认不支持通配符。

`DELETE /<index>/_doc/<_id>`

路径参数：

1. **`<index>`**（必填，字符串）目标索引的名称。
2. **`<_id>`**（必填，字符串）文档的唯一标识符。

##### 实例

`DELETE /test/_doc/sTOACIMByqhncvtAVIy1`

响应：200

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "_version": 5,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 4,
    "_primary_term": 1
}
```

此时再查询该数据，响应：404

```json
{
    "_index": "test",
    "_id": "sTOACIMByqhncvtAVIy1",
    "found": false
}
```

`DELETE /test`

响应：200

```json
{
    "acknowledged": true
}
```

#### bulk批量操作

在单个 API 调用中执行多个索引或删除操作。这样可以减少开销，并且可以大大提高索引速度。

##### 接口

`POST /_bulk`

`POST /<target>/_bulk`

路径参数：

**`<target>`**（可选，字符串）要对其执行批量操作的数据流、索引或索引别名的名称。

##### 实例

`POST /_bulk`

请求正文：

```json
{"index": {"_index": "test","_id": "1"}}
{"field1": "value1"}
{"delete": {"_index": "test","_id": "2"}}
{"create": {"_index": "test","_id": "3"}}
{"field1": "value3"}
{"update": {"_id": "1","_index": "test"}}
{"doc": {"field2": "value2"}}

```

`POST /test/_bulk`

```json
{"index": {"_id": "1"}}
{"field1": "value1"}
{"delete": {"_id": "2"}}
{"create": {"_id": "3"}}
{"field1": "value3"}
{"update": {"_id": "1"}}
{"doc": {"field2": "value2"}}

```

格式：

```
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

响应：200

```json
{
    "took": 82, //（整数）处理批量请求所花费的时间（以毫秒为单位）
    "errors": false, //（布尔值）如果为true，则批量请求中的一个或多个操作未成功完成
    "items": [ //（对象数组）包含批量请求中每个操作的结果（按提交顺序排列）
        {
            "index": {
                "_index": "test",
                "_id": "1",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 1,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "delete": {
                "_index": "test",
                "_id": "2",
                "_version": 2,
                "result": "deleted",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 2,
                "_primary_term": 1,
                "status": 200
            }
        },
        {
            "create": {
                "_index": "test",
                "_id": "3",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 3,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "update": {
                "_index": "test",
                "_id": "1",
                "_version": 2,
                "result": "updated",
                "_shards": {
                    "total": 2,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 4,
                "_primary_term": 1,
                "status": 200
            }
        }
    ]
}
```

### Search API

首先导入官网测试数据：https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip

##### 接口

`GET /<target>/_search`

`GET /test/_search`

`POST /<target>/_search`

`POST /_search`

路径参数：

**`<target>`**（可选，字符串）要搜索的数据流、索引和别名的逗号分隔列表。支持通配符`*`。要搜索所有数据流和索引，请省略此参数或使用`*` 或`_all` 

可以使用查询参数或请求正文参数指定此 API 的多个选项。如果同时指定了这两个参数，则仅使用查询参数。

##### 实例

`GET /test/_search?from=40&size=20&sort=account_number:asc`

响应：200

```json
{
    "took": 1, //（整数）Elasticsearch执行请求需要几毫秒
    "timed_out": false, //布尔值）如果为true，则请求在完成之前超时;返回的结果可以是部分结果，也可以是空的
    "_shards": { //（对象）包含用于请求的分片计数
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": { //（对象）包含返回的文档和元数据
        "total": { //（对象）有关匹配文档数的元数据
            "value": 1000, //（整数）匹配文档的总数
            "relation": "eq" //（字符串）指示参数中匹配文档的数量是准确eq还是下限get
        },
        "max_score": null, //（浮点数）返回次数最高的文档_score。此值适用于不按null _score排序的请求
        "hits": [ //（对象数组）返回的文档对象的数组。
            {
                "_index": "test",
                "_id": "40",
                "_score": null, //（浮点数）正32位浮点数，用于确定返回文档的相关性
                "_source": { //（对象）文档正文。使用_source参数从响应中排除此属性，或指定要返回的源字段。
                    "account_number": 40,
                    "balance": 33882,
                    "firstname": "Pace",
                    "lastname": "Molina",
                    "age": 40,
                    "gender": "M",
                    "address": "263 Ovington Court",
                    "employer": "Cytrak",
                    "email": "pacemolina@cytrak.com",
                    "city": "Silkworth",
                    "state": "OR"
                },
                "sort": [
                    40
                ]
            },
            {
                "_index": "test",
                "_id": "41",
                "_score": null,
                "_source": {
                    "account_number": 41,
                    "balance": 36060,
                    "firstname": "Hancock",
                    "lastname": "Holden",
                    "age": 20,
                    "gender": "M",
                    "address": "625 Gaylord Drive",
                    "employer": "Poochies",
                    "email": "hancockholden@poochies.com",
                    "city": "Alamo",
                    "state": "KS"
                },
                "sort": [
                    41
                ]
            },
            {
                "_index": "test",
                "_id": "42",
                "_score": null,
                "_source": {
                    "account_number": 42,
                    "balance": 21137,
                    "firstname": "Harding",
                    "lastname": "Hobbs",
                    "age": 26,
                    "gender": "F",
                    "address": "474 Ridgewood Place",
                    "employer": "Xth",
                    "email": "hardinghobbs@xth.com",
                    "city": "Heil",
                    "state": "ND"
                },
                "sort": [
                    42
                ]
            },
            {
                "_index": "test",
                "_id": "43",
                "_score": null,
                "_source": {
                    "account_number": 43,
                    "balance": 33474,
                    "firstname": "Ryan",
                    "lastname": "Howe",
                    "age": 25,
                    "gender": "M",
                    "address": "660 Huntington Street",
                    "employer": "Microluxe",
                    "email": "ryanhowe@microluxe.com",
                    "city": "Clara",
                    "state": "CT"
                },
                "sort": [
                    43
                ]
            },
            {
                "_index": "test",
                "_id": "44",
                "_score": null,
                "_source": {
                    "account_number": 44,
                    "balance": 34487,
                    "firstname": "Aurelia",
                    "lastname": "Harding",
                    "age": 37,
                    "gender": "M",
                    "address": "502 Baycliff Terrace",
                    "employer": "Orbalix",
                    "email": "aureliaharding@orbalix.com",
                    "city": "Yardville",
                    "state": "DE"
                },
                "sort": [
                    44
                ]
            }
        ]
    }
}
```

`POST /test/_search`

请求正文

```json
{
    "query": {
        "range": {
            "balance": {
                "gte": 30000,
                "lte": 30500
            }
        }
    },
    "sort": {
        "account_number": "asc"
    }
}
```

响应：200

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 6,
            "relation": "eq"
        },
        "max_score": null,
        "hits": [
            {
                "_index": "test",
                "_id": "31",
                "_score": null,
                "_source": {
                    "account_number": 31,
                    "balance": 30443,
                    "firstname": "Kristen",
                    "lastname": "Santana",
                    "age": 22,
                    "gender": "F",
                    "address": "130 Middagh Street",
                    "employer": "Dogspa",
                    "email": "kristensantana@dogspa.com",
                    "city": "Vale",
                    "state": "MA"
                },
                "sort": [
                    31
                ]
            },
            {
                "_index": "test",
                "_id": "262",
                "_score": null,
                "_source": {
                    "account_number": 262,
                    "balance": 30289,
                    "firstname": "Tameka",
                    "lastname": "Levine",
                    "age": 36,
                    "gender": "F",
                    "address": "815 Atlantic Avenue",
                    "employer": "Acium",
                    "email": "tamekalevine@acium.com",
                    "city": "Winchester",
                    "state": "SD"
                },
                "sort": [
                    262
                ]
            },
            {
                "_index": "test",
                "_id": "513",
                "_score": null,
                "_source": {
                    "account_number": 513,
                    "balance": 30040,
                    "firstname": "Maryellen",
                    "lastname": "Rose",
                    "age": 37,
                    "gender": "F",
                    "address": "428 Durland Place",
                    "employer": "Waterbaby",
                    "email": "maryellenrose@waterbaby.com",
                    "city": "Kiskimere",
                    "state": "RI"
                },
                "sort": [
                    513
                ]
            },
            {
                "_index": "test",
                "_id": "514",
                "_score": null,
                "_source": {
                    "account_number": 514,
                    "balance": 30125,
                    "firstname": "Solomon",
                    "lastname": "Bush",
                    "age": 34,
                    "gender": "M",
                    "address": "409 Harkness Avenue",
                    "employer": "Snacktion",
                    "email": "solomonbush@snacktion.com",
                    "city": "Grayhawk",
                    "state": "TX"
                },
                "sort": [
                    514
                ]
            },
            {
                "_index": "test",
                "_id": "707",
                "_score": null,
                "_source": {
                    "account_number": 707,
                    "balance": 30325,
                    "firstname": "Sonya",
                    "lastname": "Trevino",
                    "age": 30,
                    "gender": "F",
                    "address": "181 Irving Place",
                    "employer": "Atgen",
                    "email": "sonyatrevino@atgen.com",
                    "city": "Enetai",
                    "state": "TN"
                },
                "sort": [
                    707
                ]
            },
            {
                "_index": "test",
                "_id": "963",
                "_score": null,
                "_source": {
                    "account_number": 963,
                    "balance": 30461,
                    "firstname": "Griffin",
                    "lastname": "Sheppard",
                    "age": 20,
                    "gender": "M",
                    "address": "682 Linden Street",
                    "employer": "Zanymax",
                    "email": "griffinsheppard@zanymax.com",
                    "city": "Fannett",
                    "state": "NM"
                },
                "sort": [
                    963
                ]
            }
        ]
    }
}
```

### Query DSL

在上述 Search API 中 query 参数使用查询 DSL 定义。

Elasticsearch 提供了基于 JSON 的完整查询 DSL（域特定语言）来定义查询。将查询 DSL 视为查询的 AST（抽象语法树），由两种类型的子句组成：

1. #### **叶查询子句**

   叶查询子句在特定字段中查找特定值，例如`匹配`项、`术语`或`范围`查询。这些查询可以单独使用。

2. **复合查询子句**

   复合查询子句包装其他叶查询**或**复合查询，用于以逻辑方式组合多个查询（如 `bool` 或 `dis_max` 查询），或更改其行为（如`constant_score`查询）。

查询子句的行为有所不同，具体取决于它们是在查询上下文还是在筛选器上下文中使用。

#### 查询和筛选器上下文

`GET /test/_search`

请求正文

```json
{
    "_source": [
        "balance",
        "age",
        "state"
    ],
    "query": { //指示查询上下文
        "bool": { //bool和match意味着这将用于对每个文档的匹配程度进行评分
            "must": { //必须出现在匹配的文档中，并有助于分数。
                "match": { 
                    "state": "NM" //state字段包含NM
                }
            },
            "must_not": { //不得出现在匹配的文档中
                "range":{
                    "age": {
                        "gte": 30 //age字段大于30
                    }
                }
            },
            "filter": { //筛指示选器上下文。筛选出不匹配的文档，但不会影响匹配文档的分数。
                "range": {
                    "balance": {
                        "lte": "30000" //balance字段小于于30000
                    }
                }
            }
        }
    }
}
```

响应：200

```json
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 4.2346063,
        "hits": [
            {
                "_index": "test",
                "_id": "688",
                "_score": 4.2346063,
                "_source": {
                    "balance": 17931,
                    "state": "NM",
                    "age": 22
                }
            },
            {
                "_index": "test",
                "_id": "942",
                "_score": 4.2346063,
                "_source": {
                    "balance": 21299,
                    "state": "NM",
                    "age": 26
                }
            },
            {
                "_index": "test",
                "_id": "375",
                "_score": 4.2346063,
                "_source": {
                    "balance": 23860,
                    "state": "NM",
                    "age": 25
                }
            }
        ]
    }
}
```

#### match

用于执行**全文搜索**的标准查询，包括模糊匹配选项

全文检索，根据评分排序，会进行分词匹配

`GET /test/_search`

请求实体

```json
{
  "query": {
    "match": {
      "address": { //<field>（必需，对象）要搜索的字段
        "query": "mill lane", //（必填）您希望在提供的<field>中找到的文本、数字、布尔值或日期
        "fuzziness": "AUTO" //（可选，字符串）模糊查询，允许匹配的最大距离。
      }
    }
  }
}
```

响应：200

```json
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 27,
            "relation": "eq"
        },
        "max_score": 9.507477,
        "hits": [
            {
                "_index": "test",
                "_id": "136",
                "_score": 9.507477,
                "_source": {
                    "account_number": 136,
                    "balance": 45801,
                    "firstname": "Winnie",
                    "lastname": "Holland",
                    "age": 38,
                    "gender": "M",
                    "address": "198 Mill Lane",
                    "employer": "Neteria",
                    "email": "winnieholland@neteria.com",
                    "city": "Urie",
                    "state": "IL"
                }
            },
            {
                "_index": "test",
                "_id": "970",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 970,
                    "balance": 19648,
                    "firstname": "Forbes",
                    "lastname": "Wallace",
                    "age": 28,
                    "gender": "M",
                    "address": "990 Mill Road",
                    "employer": "Pheast",
                    "email": "forbeswallace@pheast.com",
                    "city": "Lopezo",
                    "state": "AK"
                }
            },
            {
                "_index": "test",
                "_id": "345",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 345,
                    "balance": 9812,
                    "firstname": "Parker",
                    "lastname": "Hines",
                    "age": 38,
                    "gender": "M",
                    "address": "715 Mill Avenue",
                    "employer": "Baluba",
                    "email": "parkerhines@baluba.com",
                    "city": "Blackgum",
                    "state": "KY"
                }
            },
            {
                "_index": "test",
                "_id": "472",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 472,
                    "balance": 25571,
                    "firstname": "Lee",
                    "lastname": "Long",
                    "age": 32,
                    "gender": "F",
                    "address": "288 Mill Street",
                    "employer": "Comverges",
                    "email": "leelong@comverges.com",
                    "city": "Movico",
                    "state": "MT"
                }
            },
            {
                "_index": "test",
                "_id": "1",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 1,
                    "balance": 39225,
                    "firstname": "Amber",
                    "lastname": "Duke",
                    "age": 32,
                    "gender": "M",
                    "address": "880 Holmes Lane",
                    "employer": "Pyrami",
                    "email": "amberduke@pyrami.com",
                    "city": "Brogan",
                    "state": "IL"
                }
            },
            {
                "_index": "test",
                "_id": "70",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 70,
                    "balance": 38172,
                    "firstname": "Deidre",
                    "lastname": "Thompson",
                    "age": 33,
                    "gender": "F",
                    "address": "685 School Lane",
                    "employer": "Netplode",
                    "email": "deidrethompson@netplode.com",
                    "city": "Chestnut",
                    "state": "GA"
                }
            },
            {
                "_index": "test",
                "_id": "556",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 556,
                    "balance": 36420,
                    "firstname": "Collier",
                    "lastname": "Odonnell",
                    "age": 35,
                    "gender": "M",
                    "address": "591 Nolans Lane",
                    "employer": "Sultraxin",
                    "email": "collierodonnell@sultraxin.com",
                    "city": "Fulford",
                    "state": "MD"
                }
            },
            {
                "_index": "test",
                "_id": "568",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 568,
                    "balance": 36628,
                    "firstname": "Lesa",
                    "lastname": "Maynard",
                    "age": 29,
                    "gender": "F",
                    "address": "295 Whitty Lane",
                    "employer": "Coash",
                    "email": "lesamaynard@coash.com",
                    "city": "Broadlands",
                    "state": "VT"
                }
            },
            {
                "_index": "test",
                "_id": "715",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 715,
                    "balance": 23734,
                    "firstname": "Tammi",
                    "lastname": "Hodge",
                    "age": 24,
                    "gender": "M",
                    "address": "865 Church Lane",
                    "employer": "Netur",
                    "email": "tammihodge@netur.com",
                    "city": "Lacomb",
                    "state": "KS"
                }
            },
            {
                "_index": "test",
                "_id": "449",
                "_score": 4.1042743,
                "_source": {
                    "account_number": 449,
                    "balance": 41950,
                    "firstname": "Barnett",
                    "lastname": "Cantrell",
                    "age": 39,
                    "gender": "F",
                    "address": "945 Bedell Lane",
                    "employer": "Zentility",
                    "email": "barnettcantrell@zentility.com",
                    "city": "Swartzville",
                    "state": "ND"
                }
            }
        ]
    }
}
```

#### match_phrase

该查询分析文本并根据分析的文本创建查询。

不分词。

`GET /test/_search`

请求正文

```json
{
  "query": {
    "match_phrase": {
      "address": "mill lane"
    }
  }
}
```

响应：200

```json
{
    "took": 54,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 9.507477,
        "hits": [
            {
                "_index": "test",
                "_id": "136",
                "_score": 9.507477,
                "_source": {
                    "account_number": 136,
                    "balance": 45801,
                    "firstname": "Winnie",
                    "lastname": "Holland",
                    "age": 38,
                    "gender": "M",
                    "address": "198 Mill Lane",
                    "employer": "Neteria",
                    "email": "winnieholland@neteria.com",
                    "city": "Urie",
                    "state": "IL"
                }
            }
        ]
    }
}
```

#### multi_match

多字段查询

`GET /test/_search`

请求正文

```json
{
    "query": {
        "multi_match": {
            "query": "mill", //查询字符串
            "fields": [ //查询字段
                "address",
                "city"
            ]
        }
    }
}
```

响应：200

```json
{
    "took": 11,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": 5.4032025,
        "hits": [
            {
                "_index": "test",
                "_id": "970",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 970,
                    "balance": 19648,
                    "firstname": "Forbes",
                    "lastname": "Wallace",
                    "age": 28,
                    "gender": "M",
                    "address": "990 Mill Road",
                    "employer": "Pheast",
                    "email": "forbeswallace@pheast.com",
                    "city": "Lopezo",
                    "state": "AK"
                }
            },
            {
                "_index": "test",
                "_id": "136",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 136,
                    "balance": 45801,
                    "firstname": "Winnie",
                    "lastname": "Holland",
                    "age": 38,
                    "gender": "M",
                    "address": "198 Mill Lane",
                    "employer": "Neteria",
                    "email": "winnieholland@neteria.com",
                    "city": "Urie",
                    "state": "IL"
                }
            },
            {
                "_index": "test",
                "_id": "345",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 345,
                    "balance": 9812,
                    "firstname": "Parker",
                    "lastname": "Hines",
                    "age": 38,
                    "gender": "M",
                    "address": "715 Mill Avenue",
                    "employer": "Baluba",
                    "email": "parkerhines@baluba.com",
                    "city": "Blackgum",
                    "state": "KY"
                }
            },
            {
                "_index": "test",
                "_id": "472",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 472,
                    "balance": 25571,
                    "firstname": "Lee",
                    "lastname": "Long",
                    "age": 32,
                    "gender": "F",
                    "address": "288 Mill Street",
                    "employer": "Comverges",
                    "email": "leelong@comverges.com",
                    "city": "Movico",
                    "state": "MT"
                }
            }
        ]
    }
}
```

#### bool复合查询

| 发生       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `must`     | 子句（查询）必须出现在匹配的文档中，并有助于分数。           |
| `filter`   | 子句（查询）必须出现在匹配的文档中。但是，与`must`查询的分数不同，该查询的分数将被忽略。筛选子句在筛选器上下文中执行，这意味着将忽略评分，并考虑对子句进行缓存。 |
| `should`   | 子句（查询）应出现在匹配的文档中。                           |
| `must_not` | 子句（查询）不得出现在匹配的文档中。子句在筛选器上下文中执行，这意味着将忽略评分，并考虑对子句进行缓存。由于忽略评分，因此将返回所有文档的分数`0`。 |

该查询采用*“匹配越多越好”*的方法，因此每个匹配或子句的分数将相加，以提供每个文档的最终结果。

`POST /_search`

请求正文

```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "gender": "M"
                    }
                },
                {
                    "match": {
                        "address": "mill"
                    }
                }
            ],
            "must_not": {
                "range": {
                    "age": {
                        "gte": 10,
                        "lte": 20
                    }
                }
            },
            "should": {
                "match": {
                    "lastname": "Wallace"
                }
            }
        }
    }
}
```

#### term

返回在提供的字段中包含**确切**术语的文档。

您可以使用该查询基于**精确值**（如价格、产品 ID 或用户名）查找文档。

`POST /test/_search`

请求正文

```json
{
  "query": {
    "term": {
      "age": 20
    }
  }
}
```

响应：200

```json
{
    "took": 25,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 44,
            "relation": "eq"
        },
        "max_score": 1,
        "hits": [
            {
                "_index": "test",
                "_id": "157",
                "_score": 1,
                "_source": {
                    "account_number": 157,
                    "balance": 39868,
                    "firstname": "Claudia",
                    "lastname": "Terry",
                    "age": 20,
                    "gender": "F",
                    "address": "132 Gunnison Court",
                    "employer": "Lumbrex",
                    "email": "claudiaterry@lumbrex.com",
                    "city": "Castleton",
                    "state": "MD"
                }
            },
            // 省略
        ]
    }
}
```

注意：

默认情况下，弹性搜索会在分析期间更改`text`字段的值。例如，默认标准分析器按如下方式更改`text`字段值：

- 删除大多数标点符号
- 将其余内容划分为单个单词，称为标记
- 小写令牌

为了更好地搜索`text`字段，`match`查询还会在执行搜索之前分析您提供的搜索词。这意味着`match`查询可以在`text`字段中搜索已分析的标记，而不是确切的术语。

`term`查询**不会**分析搜索词，`term`查询仅搜索您提供**的确切**字词。这意味着在搜索`text`字段时，`term`查询可能会返回较差的结果或没有结果。（效果同 match keyword）

#### 聚合

##### 实例1

搜索 address 中包含 mill 的所有人的年龄分布以及平均年龄。

`GET /test/_search`

请求正文

```json
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    }
  }
}
```

响应：200

```json
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 4,
            "relation": "eq"
        },
        "max_score": 5.4032025,
        "hits": [
            {
                "_index": "test",
                "_id": "970",
                "_score": 5.4032025,
                "_source": {
                    "account_number": 970,
                    "balance": 19648,
                    "firstname": "Forbes",
                    "lastname": "Wallace",
                    "age": 28,
                    "gender": "M",
                    "address": "990 Mill Road",
                    "employer": "Pheast",
                    "email": "forbeswallace@pheast.com",
                    "city": "Lopezo",
                    "state": "AK"
                }
            },
            // 省略
        ]
    },
    "aggregations": {
        "ageAgg": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 38,
                    "doc_count": 2
                },
                {
                    "key": 28,
                    "doc_count": 1
                },
                {
                    "key": 32,
                    "doc_count": 1
                }
            ]
        },
        "ageAvg": {
            "value": 34
        },
        "balanceAvg": {
            "value": 25208
        }
    }
}
```

##### 实例2

按照年龄聚合，并且请求这些年龄段的这些人的平均薪资

`GET /test/_search`

请求正文

```json
{
    "aggs": {
        "ageAgg": {
            "terms": {
                "field": "age",
                "size": 10
            },
            "aggs": {
                "ageAvg": {
                    "avg": {
                        "field": "age"
                    }
                }
            }
        }
    }
}
```

响应：200

```json
{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1000,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      // 省略
    ]
  },
  "aggregations": {
    "ageAgg": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 463,
      "buckets": [
        {
          "key": 31,
          "doc_count": 61,
          "ageAvg": {
            "value": 31
          }
        },
        {
          "key": 39,
          "doc_count": 60,
          "ageAvg": {
            "value": 39
          }
        },
        {
          "key": 26,
          "doc_count": 59,
          "ageAvg": {
            "value": 26
          }
        },
        {
          "key": 32,
          "doc_count": 52,
          "ageAvg": {
            "value": 32
          }
        },
        {
          "key": 35,
          "doc_count": 52,
          "ageAvg": {
            "value": 35
          }
        },
        {
          "key": 36,
          "doc_count": 52,
          "ageAvg": {
            "value": 36
          }
        },
        {
          "key": 22,
          "doc_count": 51,
          "ageAvg": {
            "value": 22
          }
        },
        {
          "key": 28,
          "doc_count": 51,
          "ageAvg": {
            "value": 28
          }
        },
        {
          "key": 33,
          "doc_count": 50,
          "ageAvg": {
            "value": 33
          }
        },
        {
          "key": 34,
          "doc_count": 49,
          "ageAvg": {
            "value": 34
          }
        }
      ]
    }
  }
}
```

##### 实例3

香出所有年龄分布，并且这些年龄段中 M 的平均薪资和 F 的平均薪资以及这个年龄段的总体平均薪资

`GET /test/_search`

请求正文

```json
{
    "aggs": {
        "ageAgg": {
            "terms": {
                "field": "age",
                "size": 10
            },
            "aggs": {
                "genderAgg": {
                    "terms": {
                        "field": "gender.keyword",
                        "size": 10
                    },
                    "aggs": {
                        "balanceAgg": {
                            "avg": {
                                "field": "balance"
                            }
                        }
                    }
                },
                "ageBalanceAgg": {
                    "avg": {
                        "field": "balance"
                    }
                }
            }
        }
    }
}
```

响应：200

```json
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1000,
            "relation": "eq"
        },
        "max_score": 1,
        "hits": [
            // 省略
        ]
    },
    "aggregations": {
        "ageAgg": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 463,
            "buckets": [
                {
                    "key": 31,
                    "doc_count": 61,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "M",
                                "doc_count": 35,
                                "balanceAgg": {
                                    "value": 29565.628571428573
                                }
                            },
                            {
                                "key": "F",
                                "doc_count": 26,
                                "balanceAgg": {
                                    "value": 26626.576923076922
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 28312.918032786885
                    }
                },
                {
                    "key": 39,
                    "doc_count": 60,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "F",
                                "doc_count": 38,
                                "balanceAgg": {
                                    "value": 26348.684210526317
                                }
                            },
                            {
                                "key": "M",
                                "doc_count": 22,
                                "balanceAgg": {
                                    "value": 23405.68181818182
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 25269.583333333332
                    }
                },
                {
                    "key": 26,
                    "doc_count": 59,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "M",
                                "doc_count": 32,
                                "balanceAgg": {
                                    "value": 25094.78125
                                }
                            },
                            {
                                "key": "F",
                                "doc_count": 27,
                                "balanceAgg": {
                                    "value": 20943
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 23194.813559322032
                    }
                },
                {
                    "key": 32,
                    "doc_count": 52,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "M",
                                "doc_count": 28,
                                "balanceAgg": {
                                    "value": 22941.964285714286
                                }
                            },
                            {
                                "key": "F",
                                "doc_count": 24,
                                "balanceAgg": {
                                    "value": 25128.958333333332
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 23951.346153846152
                    }
                },
                {
                    "key": 35,
                    "doc_count": 52,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "M",
                                "doc_count": 28,
                                "balanceAgg": {
                                    "value": 24226.321428571428
                                }
                            },
                            {
                                "key": "F",
                                "doc_count": 24,
                                "balanceAgg": {
                                    "value": 19698.791666666668
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 22136.69230769231
                    }
                },
                {
                    "key": 36,
                    "doc_count": 52,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "M",
                                "doc_count": 31,
                                "balanceAgg": {
                                    "value": 20884.677419354837
                                }
                            },
                            {
                                "key": "F",
                                "doc_count": 21,
                                "balanceAgg": {
                                    "value": 24079.04761904762
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 22174.71153846154
                    }
                },
                {
                    "key": 22,
                    "doc_count": 51,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "F",
                                "doc_count": 27,
                                "balanceAgg": {
                                    "value": 22152.74074074074
                                }
                            },
                            {
                                "key": "M",
                                "doc_count": 24,
                                "balanceAgg": {
                                    "value": 27631.708333333332
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 24731.07843137255
                    }
                },
                {
                    "key": 28,
                    "doc_count": 51,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "F",
                                "doc_count": 31,
                                "balanceAgg": {
                                    "value": 27076.8064516129
                                }
                            },
                            {
                                "key": "M",
                                "doc_count": 20,
                                "balanceAgg": {
                                    "value": 30129.35
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 28273.882352941175
                    }
                },
                {
                    "key": 33,
                    "doc_count": 50,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "F",
                                "doc_count": 26,
                                "balanceAgg": {
                                    "value": 26437.615384615383
                                }
                            },
                            {
                                "key": "M",
                                "doc_count": 24,
                                "balanceAgg": {
                                    "value": 23638.291666666668
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 25093.94
                    }
                },
                {
                    "key": 34,
                    "doc_count": 49,
                    "genderAgg": {
                        "doc_count_error_upper_bound": 0,
                        "sum_other_doc_count": 0,
                        "buckets": [
                            {
                                "key": "F",
                                "doc_count": 30,
                                "balanceAgg": {
                                    "value": 26039.166666666668
                                }
                            },
                            {
                                "key": "M",
                                "doc_count": 19,
                                "balanceAgg": {
                                    "value": 28027
                                }
                            }
                        ]
                    },
                    "ageBalanceAgg": {
                        "value": 26809.95918367347
                    }
                }
            ]
        }
    }
}
```

### Mapping

映射是定义文档及其包含的字段的存储和索引方式的过程。

每个文档都是字段的集合，每个字段都有自己的数据类型。映射数据时，将创建一个映射定义，其中包含与文档相关的字段列表。映射定义还包括元数据字段（如`_source`字段），这些字段自定义如何处理文档的关联元数据。

使用*动态映射*和*显式映射*来定义数据。每种方法都根据您在数据旅程中所处的位置提供不同的优势。例如，将不想使用默认值的字段显式映射，或者更好地控制创建哪些字段。然后，您可以允许弹性搜索动态添加其他字段。

**注意：ES 8 已不再支持映射类型！**

#### 字段类型

##### 常见类型

binary：编码为 Base64 字符串的二进制值。

boolean：`true`和`false`值。

关键字：关键字系列，包括`keyword`、`constant_keyword`和`wildcard` 。

数字：数值类型，如`long`和`double`，用于表示金额。

##### 日期

日期类型，包括`date`和`date_nanos`。

alias：定义现有字段的别名。

##### 对象和关系类型

object：一个 JSON 对象。

flattened：作为单个字段值的整个 JSON 对象。

nested：保留其子字段之间关系的 JSON 对象。

join：为同一索引中的文档定义父/子关系。

##### 结构化数据类型

范围：范围类型，如`long_range`、`double_range`、`date_range`和`ip_range`。

ip：互联网地址和 IPv6 地址。

version：软件版本。支持语义版本控制优先规则。

murmur3：计算和存储值的哈希。

##### 聚合数据类型

aggregate_metric_double：预先聚合的指标值。

histogram：直方图形式的预聚合数值。

##### 文本搜索类型

文本字段：文本系列，包括`text`和`match_only_text`。经过分析的非结构化文本。

annotated-text：包含特殊标记的文本。用于标识命名实体。

completion：用于自动完成建议。

search_as_you_type：`text`-like 类型，用于键入完成。

token_count：文本中的标记计数。

##### 文档排名类型

dense_vector：记录浮点值的密集矢量。

rank_feature：记录数字要素以在查询时提高命中率。

rank_features：记录数字特征以在查询时提高命中率。

##### 空间数据类型

geo_point：纬度和经度点。

geo_shape：复杂形状，如多边形。

point：任意笛卡尔点。

shape：任意笛卡尔几何。

##### 其他类型

percolator：索引在查询 DSL 中编写的查询。

#### 映射参数

以下映射参数是某些或所有字段数据类型通用的：

- `analyzer`

- `coerce`

- `copy_to`

- `doc_values`

- `dynamic`

- `eager_global_ordinals`

- `enabled`

- `fielddata`

- `fields`

- `format`

- `ignore_above`

- `ignore_malformed`

- `index_options`

- `index_phrases`

- `index_prefixes`

- `index`

  `index`选项控制是否对字段值编制索引。它接受`true`或`false`并默认为`true`。未编制索引的字段通常不可查询。

- `meta`

- `normalizer`

- `norms`

- `null_value`

- `position_increment_gap`

- `properties`

- `search_analyzer`

- `similarity`

- `subobjects`

- `store`

- `term_vector`

#### 映射

要为文档编制索引，您不必先创建索引、定义映射类型和定义字段——只需为文档编制索引，索引、类型和字段将自动显示：

自动检测和添加新字段称为*动态映射*。动态映射规则可以自定义，以适合您的目的：

- 动态字段映射

  控制动态字段检测的规则。

- 动态模板

  用于为动态添加的字段配置映射的自定义规则。

##### 动态映射

当 Elasticsearch 在文档中检测到新字段时，默认情况下会将该字段*动态*添加到类型映射中。`dynamic`参数控制此行为。

您可以通过将参数设置为`true`或`runtime`来显式指示 Elasticsearch 基于传入的文档动态创建字段。启用动态字段映射后，弹性搜索将使用下表中的规则来确定如何映射每个字段的数据类型。

| JSON 数据类型                           | `"dynamic":"true"`             | `"dynamic":"runtime"`          |
| --------------------------------------- | ------------------------------ | ------------------------------ |
| `null`                                  | 未添加字段                     | 未添加字段                     |
| `true`或`false`                         | `boolean`                      | `boolean`                      |
| `double`                                | `float`                        | `double`                       |
| `long`                                  | `long`                         | `long`                         |
| `object`                                | `object`                       | 未添加字段                     |
| `array`                                 | 取决于数组中的第一个非`null`值 | 取决于数组中的第一个非`null`值 |
| `string`通过`date`检测                  | `date`                         | `date`                         |
| `string`通过`numeric`检测               | `float`或`long`                | `double`或`long`               |
| `string`未通过`date`检测或`numeric`检测 | `text`带有子字段`.keyword`     | `keyword`                      |

##### 显式映射

######  创建具有显式映射的索引

虽然动态映射对于入门非常有用，但在某些时候，您需要指定自己的显式映射。

`PUT /my_index`

请求正文

```json
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
```

响应：200

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "my_index"
}
```

######  将字段添加到现有映射

`PUT /my_index/_mapping`

请求正文

```json
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
```

响应：200

```json
{
    "acknowledged": true
}
```

###### 更新字段的映射

除了支持的映射参数外，您无法更改现有字段的映射或字段类型。更改现有字段可能会使已编制索引的数据失效。

如果需要更改其他索引中字段的映射，请使用正确的映射创建一个新索引，然后将数据重新索引到该索引中。

###### 重新索引

将文档从源复制到目标。

源可以是任何现有索引、别名或数据流。目标必须与源不同。例如，不能将数据流重新索引到其自身中。

`POST /_reindex`

请求正文

```json
{
  "source": {
    "index": "test"
  },
  "dest": {
    "index": "my_index"
  }
}
```

响应：200

```json
{
    "took": 184,
    "timed_out": false,
    "total": 1000,
    "updated": 0,
    "created": 1000,
    "deleted": 0,
    "batches": 1,
    "version_conflicts": 0,
    "noops": 0,
    "retries": {
        "bulk": 0,
        "search": 0
    },
    "throttled_millis": 0,
    "requests_per_second": -1,
    "throttled_until_millis": 0,
    "failures": []
}
```

### 分词

一个tokenizer（分词器）接收一个字符流，将之分割为独立的 tokens（词元，通常是独立的单词），然后输出 tokens 流。

例如，whitespace tokenizer 遇到空白字符时分割文本。它会将文本 "Quick brown fox!" 分割为 [Quick,brown,fox!] 。

该 tokenizer（分词器）还负责记录各个 term（词条）的顺序或 position 位置（用于 phrase 短语和 word proximity 词近邻查询），以及 term（词条）所代表的原始 word（单词）的 start（起始）和 end（结束）的 character offsets（字符偏移量）（用于高亮显示搜索的内容）。Elasticsearch 提供了很多内置的分词器，可以用来构建 customanalyzers（自定义分词器）。

#### 标准分析器

`standard`分析器是默认分析器，如果未指定任何分析器，则使用该分析器。它提供基于语法的标记化（基于 Unicode 文本分割算法，如 [Unicode 标准附录 #29](https://unicode.org/reports/tr29/) 中所述），并且适用于大多数语言。

`POST /_analyze`

请求正文

```json
{
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}
```

响应：200

```json
{
    "tokens": [
        {
            "token": "the",
            "start_offset": 0,
            "end_offset": 3,
            "type": "<ALPHANUM>",
            "position": 0
        },
        {
            "token": "2",
            "start_offset": 4,
            "end_offset": 5,
            "type": "<NUM>",
            "position": 1
        },
        {
            "token": "quick",
            "start_offset": 6,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 2
        },
        {
            "token": "brown",
            "start_offset": 12,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 3
        },
        {
            "token": "foxes",
            "start_offset": 18,
            "end_offset": 23,
            "type": "<ALPHANUM>",
            "position": 4
        },
        {
            "token": "jumped",
            "start_offset": 24,
            "end_offset": 30,
            "type": "<ALPHANUM>",
            "position": 5
        },
        {
            "token": "over",
            "start_offset": 31,
            "end_offset": 35,
            "type": "<ALPHANUM>",
            "position": 6
        },
        {
            "token": "the",
            "start_offset": 36,
            "end_offset": 39,
            "type": "<ALPHANUM>",
            "position": 7
        },
        {
            "token": "lazy",
            "start_offset": 40,
            "end_offset": 44,
            "type": "<ALPHANUM>",
            "position": 8
        },
        {
            "token": "dog's",
            "start_offset": 45,
            "end_offset": 50,
            "type": "<ALPHANUM>",
            "position": 9
        },
        {
            "token": "bone",
            "start_offset": 51,
            "end_offset": 55,
            "type": "<ALPHANUM>",
            "position": 10
        }
    ]
}
```

`POST /_analyze`

请求正文

```json
{
  "text": "中华人民共和国国歌"
}
```

响应：200

```json
{
    "tokens": [
        {
            "token": "中",
            "start_offset": 0,
            "end_offset": 1,
            "type": "<IDEOGRAPHIC>",
            "position": 0
        },
        {
            "token": "华",
            "start_offset": 1,
            "end_offset": 2,
            "type": "<IDEOGRAPHIC>",
            "position": 1
        },
        {
            "token": "人",
            "start_offset": 2,
            "end_offset": 3,
            "type": "<IDEOGRAPHIC>",
            "position": 2
        },
        {
            "token": "民",
            "start_offset": 3,
            "end_offset": 4,
            "type": "<IDEOGRAPHIC>",
            "position": 3
        },
        {
            "token": "共",
            "start_offset": 4,
            "end_offset": 5,
            "type": "<IDEOGRAPHIC>",
            "position": 4
        },
        {
            "token": "和",
            "start_offset": 5,
            "end_offset": 6,
            "type": "<IDEOGRAPHIC>",
            "position": 5
        },
        {
            "token": "国",
            "start_offset": 6,
            "end_offset": 7,
            "type": "<IDEOGRAPHIC>",
            "position": 6
        },
        {
            "token": "国",
            "start_offset": 7,
            "end_offset": 8,
            "type": "<IDEOGRAPHIC>",
            "position": 7
        },
        {
            "token": "歌",
            "start_offset": 8,
            "end_offset": 9,
            "type": "<IDEOGRAPHIC>",
            "position": 8
        }
    ]
}
```

可见标准分析器并不支持中文分词，因此我们需要额外安装中文分词器。

#### 安装ik分词器

[medcl/elasticsearch-analysis-ik(github.com)](https://github.com/medcl/elasticsearch-analysis-ik)

下载对应版本的插件，将插件解压缩到文件夹`your-es-root/plugins/ik`

```bash
mkdir es/plugins/ik
cd es/plugins/ik
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.4.1/elasticsearch-analysis-ik-8.4.1.zip
unzip elasticsearch-analysis-ik-8.4.1.zip
rm -rf *.zip
```

然后重启容器即可，使用`elasticsearch-plugin list`指令列出所有已安装的插件

两种模式：

1. `ik_max_word`

   会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query；

2. `ik_smart`

   会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询。

`POST /_analyze`

请求正文

```json
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国国歌"
}
```

响应：200

```json
{
    "tokens": [
        {
            "token": "中华人民共和国",
            "start_offset": 0,
            "end_offset": 7,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "国歌",
            "start_offset": 7,
            "end_offset": 9,
            "type": "CN_WORD",
            "position": 1
        }
    ]
}
```

#### ik分词器自定义扩展词库

`plugins/ik/config/IKAnalyzer.cfg.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords">custom/ext_stopword.dic</entry>
 	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">location</entry>
 	<!--用户可以在这里配置远程扩展停止词字典-->
	<entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry>
</properties>
```

我们可以在 nginx 里放置一个远程扩展字典，这就需要我们使用 docker 部署 nginx，这里不做演示。

## Elasticsearch-Client

版本：Spring 2.7.3，Elasticsearch 8.4.1

### 整合

新建类`config.ElasticSearchConfig`

```java
@Configuration
public class ElasticSearchConfig {

    @Bean
    public ElasticsearchClient client() {
        RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200)).build();
        ElasticsearchTransport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}
```

由于 elasticsearch 现已默认开启身份验证，因此还需修改本处以加上身份验证的代码。

```java
@Configuration
public class ElasticSearchConfig {

    @Bean
    public ElasticsearchClient elasticsearchClient() {
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(
                AuthScope.ANY,
                new UsernamePasswordCredentials("elastic", "RUikbaL7AvegIbk4xC4j")
        );
        RestClient restClient = RestClient
                .builder(new HttpHost("localhost", 9200))
                .setHttpClientConfigCallback(
                        httpClientBuilder -> httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider)
                ).build();
        ElasticsearchTransport transport = new RestClientTransport(restClient, new JacksonJsonpMapper());
        return new ElasticsearchClient(transport);
    }
}
```

### 测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class GulimallSearchApplicationTests {

    @Autowired
    private ElasticsearchClient client;

    @Test
    void testIndex() throws IOException {
        User user = new User();
        user.setId(String.valueOf(1L));
        user.setName("张三");
        user.setAge(18);
        IndexResponse response = client.index(i -> i
                .index("user")
                .id(user.getId())
                .document(user));
        System.out.println(response);
    }

    @Test
    void testGet() throws IOException {
        GetResponse<ObjectNode> response = client.get(g -> g
                        .index("user")
                        .id("1"),
                ObjectNode.class //目标类是一个原始的JSON对象
        );
        if (response.found()) {
            ObjectNode jsonNodes = response.source();
            String name = jsonNodes.get("name").asText();
            System.out.println(name);
        } else {
            System.out.println("User not found");
        }
    }

    @Test
    void testSearch() throws IOException {
        String searchText = "张三";
        SearchResponse<ObjectNode> response = client.search(s -> s
                        .index("user")
                        .query(q -> q
                                .match(t -> t
                                        .field("name")
                                        .query(searchText)
                                )
                        ),
                ObjectNode.class
        );
        TotalHits total = response.hits().total();
        boolean isExactResult = total.relation() == TotalHitsRelation.Eq;
        if (isExactResult) {
            System.out.println("There are " + total.value() + " results");
        } else {
            System.out.println("There are more than " + total.value() + " results");
        }
        List<Hit<ObjectNode>> hits = response.hits().hits();
        for (Hit<ObjectNode> hit : hits) {
            ObjectNode jsonNodes = hit.source();
            System.out.println("Found user id " + jsonNodes.get("id") + ", score " + hit.score());
        }
    }

    @Test
    void testAggregations() throws IOException {
        Query query = MatchQuery.of(m -> m
                .field("address")
                .query("mill")
        )._toQuery();
        SearchResponse<ObjectNode> response = client.search(search -> search
                        .index("test")
                        .query(query)
                        .aggregations("ageAgg", ageAgg -> ageAgg
                                .terms(terms -> terms
                                        .field("age")
                                        .size(10)
                                )
                                .aggregations("balanceAgg", balanceAgg -> balanceAgg
                                        .avg(avg -> avg
                                                .field("balance")
                                        )
                                )
                        ),
                ObjectNode.class
        );
        HitsMetadata<ObjectNode> OutHits = response.hits();
        List<Hit<ObjectNode>> innerHits = OutHits.hits();
        for (Hit<ObjectNode> hit : innerHits) {
            ObjectNode source = hit.source();
            Account account = JSON.parseObject(source.toString(), Account.class);
            System.out.println(account);
        }
        Map<String, Aggregate> aggregations = response.aggregations();
        LongTermsAggregate ageAgg = aggregations.get("ageAgg").lterms();
        for (LongTermsBucket bucket : ageAgg.buckets().array()) {
            System.out.println(
                    "年龄：" + bucket.key() + "，频数：" + bucket.docCount() + "，平均工资：" + bucket.aggregations().get("balanceAgg").avg().value()
            );
        }
    }

    @Data
    class User {
        private String id;
        private String name;
        private Integer age;
    }

    @Data
    class Account {
        private int accountNumber;
        private int balance;
        private String firstname;
        private String lastname;
        private int age;
        private String gender;
        private String address;
        private String employer;
        private String email;
        private String city;
        private String state;
    }
}
```


