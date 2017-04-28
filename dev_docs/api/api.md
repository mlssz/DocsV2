FORMAT: 1A

# Wonderful Repository API

## Tips
- host: http://o1.mephis.me:8234
- db doc: [https://mlssz.github.io/DocsV2/dev_docs/db_design/index.html](https://mlssz.github.io/DocsV2/dev_docs/db_design/index.html)

## readme

- 响应吗 `501` 表示 `Not Implemented`, 即接口未写完
- `Date(String)` 表示 日期字符串
- _id 为 mongodb 中数据项的 pk
- url中的 page 参数是从 0 开始的

## url 中的others参数

url 中的others参数作为筛选要求

> 下列 reponse body 结构表示，返回值为一个array，array的元素是object， 以及object的键值

others结构：

- (array)
    - (object)
        - key `String` (required) -- 需要筛选的键
        - value `T | Array<T>` (optional) -- 如果value为单个值，则直接匹配，如果value为数组，则进行多值匹配
        - region `Array<T>` (optional) -- 该值为包含两个元素的数组，[from, to]
        
## 错误信息

1. url path 中的参数错误一律返回 `status 404` 与空响应体
2. 其他错误一律返回 `status 400` 与相应的错误信息，格式如下

```javascript
    // Response 400 (application/json)
    {
        "error": "Your request body is wrong!"
    }
```



# Group Material

## Material [/material/{id}]

+ Parameters
    + id (String, required) - 物资的_id

### 获取一个特定的物资 [GET]

 **Response 200 Body**
- _id `String` -- 物资_id
- id `Number` -- 物资编号
- type `String` -- 物资类型
- description `Date(String)` -- 物资描述
- import_time `Date(String)` -- 入库时间
- estimated_export_time   `Date(String)` -- 估计出库时间
- height `Number` -- 物资长度，该系统中固定为1
- width `Number` -- 物资长度，该系统中固定为1
- length `Number` -- 物资长度，该系统中固定为1
- repository_id `Number` -- 储存仓库的id
- location_id `Number` -- 仓库中位置的id
- layer `Number` -- 位置中的层（0， 1， 2）
- status `Number` -- 状态码，详见 db doc
- last_migrations `String` -- 最近一次搬运记录_id
- location_update_time `Date(String)` -- 位置更新时间

+ Response 200 (application/json)
    {
        "_id": "1491451593158",
        "id": 1491451593158,
        "type": "tester",
        "description": "wonderful repository",
        "import_time": "2017-04-06T04:57:36.801Z",
        "estimated_export_time": "2017-04-06T04:57:36.801Z",
        "height": 1,
        "width": 1,
        "length": 2,
        "repository_id": 3,
        "layer": 0,
        "location_id": 2,
        "status": 300,
        "last_migrations": "1234",
        "location_update_time": "2017-04-06T04:57:36.801Z"
    }

### 修改物资的部分信息 [PATCH]

>该接口用于修改物资的部分信息。请求体为一个对象，它的keys都是要修改的键，values是新的值

**Request Body**

- key: value  -- key都是要修改的键，value是新的值，可多对

+ Request (application/json)
    {
        "repository_id": 4,
        "location_id": 5,
    }

+ Response 200 (application/json)
    {}

### 删除特定物资 [DELETE]

+ Response 200 (application/json)
    {}

## Migration of Material [/material/{id}/migration/{mid}]

+ Parameters
    + id (String, required) - 物资的_id
    + mid (String, required) - 移动信息的_id

### 取消移动物资 [DELETE]

> 该接口只能修改对应任务处于未开始状态（status==0）的任务。通过 离当前最近的且已完成的移动信息 恢复 物资 的数据，主要是repository_id、location_id和last_migration，并修改 status 为 100，删除移动信息与任务信息。

> 表示出库时，还需要删除exportinfo数据。


+ Response 200 (application/json)
    {}


## Migrations Of Material [/material/{id}/migrations]

+ Parameters
    + id (String, required) - 物资的_id

### 查看物品移动记录数量 [GET /material/{id}/migrations]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 移动记录数量

+ Response 200 (application/json)
    {
        "num": 2
    }
    
### 查看一个物品的移动记录 [GET /material/{id}/migrations{?page, limit}]

**Response 200 Body**

- _id `String` -- 移动信息的_id
- material `String`-- 物资_d
- date `date` -- 搬运完成时间, 未完成为空值
- from_repository `number` -- 原仓库, 仓库id, 0表示入库
- from_location `number` -- 原位置, 原位置的id
- from_layer `number` -- 原层
- to_repository `number` -- 目标仓库, 仓库id, -1表示出库
- to_location `number` -- 目标位置, 目标位置的id
- to_layer `number` -- 目标层

+ Parameters
    + id (String, required) - 物资的id
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    
+ Response 200 (application/json)

    [{
        "_id": "dsafdsaf32141314",
        "material": "dsafdsaf32141314",
        "from_repository": 2,
        "from_location": 12,
        "from_layer": 0,
        "to_repository": -1,
        "to_location": 0,
        "to_layer": 0,
    }, ...]
    
### 移动物资到新位置 [POST]

> 该接口会创建一个新的migration，并修改materials的repository_id, location_id和status, 再创建一个相应的task。

> 表示出库时，还需要创建一个exportinfo数据。

**Resquest Body**

- repository `Number` (required) -- 目标仓库, 仓库id, -1表示出库
- location `Number` (required) -- 目标位置, 目标位置的id
- exportinfo (object, required while repository == -1)
    - destination `String` (required) -- 去向

**Response 201 Body**

- _id `String` -- 移动信息的_id
- material `String`-- 物资_id
- date `Date` -- 搬运完成时间, 未完成为空值
- from_repository `Number` -- 原仓库, 仓库id
- from_location `Number` -- 原位置, 原位置的id
- from_layer `number` -- 原层
- to_repository `Number` -- 目标仓库, 仓库id, -1表示出库
- to_location `Number` -- 目标位置, 目标位置的id
- to_layer `number` -- 目标层
- exportinfo (object) -- 当to_repository == -1时有效
    - destination `String` -- 去向

+ Request (application/json)
    {
        "repository": -1,
        "location": 0,
        "exportinfo": {
            "distination": "幻想乡"
        }
    }

+ Response 201 (application/json)
    {
        "_id": "dsafdsaf32141314",
        "material": "dsafdsaf32141314",
        "from_repository": 2,
        "from_location": 12,
        "from_layer": 1,
        "to_repository": -1,
        "to_location": 0,
        "to_layer": 1,
        "exportinfo": {
            "distination": "幻想乡"
        }
    }


## Materials [/materials]

### 录入多个物资 [POST]

> 该接口根据管理员数据的物资相关信息与数量，生成多个物资，id自动生成

**Request Body**

- type `String` (required) -- 物资类型
- description `String` (optional, default: "") -- 物资描述
- import_time `Date`  (optional, default: now) -- 物资入库时间
- estimated_export_time `Date` (optional) -- 该值为时间数值, date.getTime(), 估计出库时间
- height `Number` (optional, default: 1) --  物资高度，该系统中固定为1
- width `Number` (optional, default: 1) --  物资高度，该系统中固定为1
- length `Number` (optional, default: 2) --  物资高度，该系统中固定为1
- repository_id `Number` (required) --  储存仓库的id
- location_id `Number` (required) -- 仓库中位置的id
- layer `Number` (required) -- 位置中的层
- num `Number` (optional, default: 1) -- 物资数量

**Response 201 Body**

- (array)
    - (object)
      - _id `String` -- 物资_id
      - id `Number` -- 物资编号
      - type `String` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - layer `Number` -- 位置中的层
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Request (application/json)
    {
        "type": "tester",
        "description": "wonderful repository",
        "import_time": 1491451593158,
        "estimated_export_time": 1491451593158,
        "height": 1,
        "width": 1,
        "length": 2,
        "repository_id": 2,
        "location_id":  3,
        "layer": 1,
        "num": 2
    }

+ Response 201 (application/json)
    [{
        "_id": "dsafdsaf32141314",
         "id": 1491451593158,
         "type": "tester",
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 2,
         "repository_id": 3,
         "location_id": 2,
         "layer": 1,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }...]
    


### 获取所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20


### 获取所有物资信息 [GET /materials{?page, limit}]


**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 物资_id
      - id `Number` -- 物资编号
      - type `String` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - layer `Number` -- 位置中的层
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "_id": "dsafdsaf32141314",
         "id": 1491451593158,
         "type": "tester",
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 2,
         "repository_id": 3,
         "location_id": 2,
         "layer": 1,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]

### 删除所有物资 [DELETE]

> 该接口只提供给root用户，或用于开发测试，慎用！

+ Response 200 (application/json)
    {}

## Materials In Repository [/repository/{id}/materials]

+ Parameters
    + id (required) - 仓库_id

### 获取特定仓库所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20

### 获取特定仓库中的物资 [GET /repository/{id}/materials{?page, limit}]

**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 物资_id
      - id `Number` -- 物资编号
      - type `String` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - layer `Number` -- 位置中的层
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + id (required) - 仓库id
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "_id": "dsafdsaf32141314",
         "id": 1491451593158,
         "type": "tester",
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 2,
         "repository_id": 3,
         "location_id": 2,
         "layer": 0,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]

## Materials In Location [/repository/:rid/location/:lid/materials]

+ Parameters
    + rid (required) - 仓库id
    + lid (required) - 位置id

### 获取特定位置所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20

### 获取特定位置的物资 [GET /repository/{rid}/location/{lid}/materials{?page, limit}]

**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 物资_id
      - id `Number` -- 物资编号
      - type `String` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - layer `Number` -- 位置中的层
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + rid (required) - 仓库id
    + lid (required) - 位置id
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "_id": "dsafdsaf32141314",
         "id": 1491451593158,
         "type": "tester",
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 2,
         "repository_id": 3,
         "location_id": 2,
         "layer": 0,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]

# Group Repository

## Repository [/repository/{id}]

+ Parameters
    + id (String, required) - 仓库的_id
    
### 返回仓库详细信息 [GET]

> 包括 locations 的信息

**Response 200 Body**

- _id `String` -- 仓库_id
- id `Number` -- 仓库编号
- available_space `Number` -- 空闲空间数量
- stored_count `Number` -- 已存放物资数
- locations(array) -- 各位置信息
    - (object)
      - id `Number` -- 位置id, 该值用于定位,顺时针算
      - place `Number` -- 块号, 该值仅用于显示，根据文档图，分四个大块，该值表示第几块（顺时针算）
      - label `String` -- 位置标记, 该值仅用于显示，显示location时其上的标记
      - available_space `Number` -- 剩余空间, 该位置剩余的高度。(2*3*10 立方米)
      - materials_num `Array<Number>` -- 各层存放的物资的数量

+ Response 200 (application/json)
    
    {
        "_id": "dsafkhklsdfakfsda",
        "id": 1,
        "available_space": 40,
        "stored_count": 23,
        "locations": [{
            id: 2,
            place: 3,
            label: "23",
            available_space: 34,
            materials: [1, 2, 3]
        }, ...]
    }
    
### 删除仓库 [DELETE]

+ Response 200 (application/json)
    
    {}
    
## Repostories [/repositories]

### 返回所有仓库信息 [GET]

> 不包括 locations 的信息

**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 仓库_id
      - id `Number` -- 仓库编号
      - available_space `Number` -- 空闲空间数量
      - stored_count `Number` -- 已存放物资数

+ Response 200 (application/json)
    
    [{
        "_id": "dsafkhklsdfakfsda",
        "id": 1,
        "available_space": 40,
        "stored_count": 23
    }, ...]
    
### 创建一个仓库 [POST]

**Request Body**

- id `Number` -- 仓库编号, >= 0

**Response 200 Body**

- _id `String` -- 仓库_id
- id `Number` -- 仓库编号
- available_space `Number` -- 空闲空间数量
- stored_count `Number` -- 已存放物资数
- locations(array) -- 各位置信息
    - (object)
      - id `Number` -- 位置id, 该值用于定位,顺时针算
      - place `Number` -- 块号, 该值仅用于显示，根据文档图，分四个大块，该值表示第几块（顺时针算）
      - label `String` -- 位置标记, 该值仅用于显示，显示location时其上的标记
      - available_space `Number` -- 剩余空间, 该位置剩余的高度。(2*3*10 立方米)
      - materials_num `Array<Number>` -- 各层存放的物资的数量
      
+ Request (application/json)

    {
        "id": 2
    }

+ Response 200 (application/json)
    
    {
        "_id": "dsaflksdjfsalkfj",
        "id": 1,
        "available_space": 40,
        "stored_count": 23,
        "locations": [{
            id: 2,
            place: 3,
            label: "23",
            available_space: 34,
            materials: [1, 2, 3]
        }, ...]
    }
    
### 删除所有仓库 [DELETE]

+ Response 200 (application/json)
    
    {}

## Locations [/repository/{id}/locations]

+ Parameters
    + id (String, required) - 仓库的_id

### 返回仓库中所有位置的信息 [GET]

**Response 200 Body**

- (array)
    - (object)
      - id `Number` -- 位置id, 该值用于定位,顺时针算
      - place `Number` -- 块号, 该值仅用于显示，根据文档图，分四个大块，该值表示第几块（顺时针算）
      - label `String` -- 位置标记, 该值仅用于显示，显示location时其上的标记
      - available_space `Number` -- 剩余空间, 该位置剩余的高度。(2*3*10 立方米)
      - materials_num `Array<Number>` -- 各层存放的物资的数量

+ Response 200 (application/json)
    [{
        id: 2,
        place: 3,
        label: "23",
        available_space: 34,
        materials: [1, 2, 3]
    }, ...]
    
## Location [/repository/{id}/location]
    
### 自动分配仓库中的空位 [GET /repository/{id}/empty-location{?num, width, height, length}]

> 该接口查询仓库余位，根据货物大小，调度分配到合适的仓库位置(具体到某个仓 库内的某个货架的位置)。在将货物信息保存到数据库之前，需要先调用该接口获得空位。如果没有找到需要的空位，会返回404错误

**Response 200 Body**

- (array)
    - (object)
      - location `Number` - 位置id
      - layer `Number` - 层，从下到上（0-2）
      - num `Number` - 该位置的空位数

+ Parameters
    + id (String, required) - 仓库的_id
    + width (Number, optional) - 物资的宽度
        + Default: 1
    + length (Number, optional) - 物资的长度
        + Default: 1
    + height (Number, optional) - 物资的高度
        + Default: 1
    + num (Number, optional) - 物资的数量
        + Default: 1
    
+ Response 200 (application/json)
    // HTTP1.1 GET /repository/1/empty-location?num=10
    [{
        "location": 3,
        "layer": 0,
        "num": 3
    }, {
        "location": 3,
        "layer": 1,
        "num": 7
    }]
    
# Group Error

## Errors [/errors]

### 获取错误信息数量 [HEAD]

> 当你希望一页一页地获取时，你可以先通过该接口获取所有的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 错误信息数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20


### 获取所有错误信息 [GET /errors{?page, limit}]

**Response 200 Body**

- (array)
    - (object)
        - fixed `Boolean` -- 修复完成
        - error_code `Number` -- 错误码, 1位置错误2无法识别
        - repository `Number` --  错误仓库
        - location `Number` --  错误位置
        - layer `Number` -- ❎错误层
        - material `Number` -- 物资id, 如果错误码为2，则为空值
        - image `Number` --  照相图片，错误照片，圈出错误

+ Parameters
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
        "fixed": false,
        "error_code": 1,
        "repository": 1,
        "location": 3,
        "layer": 0,
        "material": 32143214,
        "image": "/errors/a.png"
    }, ...]

### 创建错误 [POST]

> 这个api留给server end调用, 在创建错误的同时需要创建任务。

**Request Body**

- repository `Number` (required) -- 错误所在仓库id
- location `Number` (required) -- 错误所在位置id
- layer `Number` (required) -- 错误所在的层
- error_code `Number` (optional, default: 1) -- 错误码, 1位置错误2无法识别
- material `Number` (optional) -- 物资id, 如果错误码为1，则为必须
- image `String` (required) -- 照相图片路径, 错误照片，圈出错误

**Response 201 Body**
- _id `String` -- 任务_id
- action `Number` --  动作，详见 db doc
- status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
- publish_time `Date`-- 任务发布时间
- start_time  `Date` --   任务开始时间
- end_time   `Date`  --   任务结束时间
- remark     `Date`  --   任务附加评语, 一般用于任务取消时
- error(object) -- 当action为6开头时才会有这个键值
  - repository `Number` --  错误仓库
  - location `Number` --  错误位置
  - layer `Number` (required) -- 错误所在的层
  - material `Number` -- 物资id, 如果错误码为2，则为空值
  - image `Number` --  照相图片，错误照片，圈出错误


+ Request (application/json)
    {
        "repository": 2,
        "location": 3,
        "error_code": 1,
        "material": 3214132,
        "layer": 0,
        "image": "/errors/a.png"
    }
    
+ Response 201 (application/json)
    {
        "_id": "dsafdsadsaf32413141kl2",
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": "",
        "error": {
          "repository": 1,
          "location": 3,
          "layer": 0,
          "material": 32143214,
          "image": "/errors/a.png"
        }
    }

    
# Group Task

## Tasks [/tasks]

### 获取所有任务数量 [HEAD /tasks{?other}]

> 当你希望一页一页地获取时，你可以先通过该接口获取所有的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 错误信息数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20


### 获取所有任务信息 [GET /tasks{?page, limit, other}]

**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 任务_id
      - action `Number` --  动作，详见 db doc
      - status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
      - publish_time `Date`-- 任务发布时间
      - start_time  `Date` --   任务开始时间
      - end_time   `Date`  --   任务结束时间
      - staff(object)
        - _id `String` -- 职员_id
        - name `String`  -- 职员名
        - account `String`  -- 职员账户
        - passwd `String`  -- 职员密码
        - sex `Number`  -- 性别，0女1男
        - age `Number`   -- 年龄
        - permission `Number`  -- 职员权限，0管理员1员工99root
        - signup_time `Date`  -- 注册时间
        - last_login_time `Date`  -- 最近登录时间
      - remark     `Date`  --   任务附加评语, 一般用于任务取消时
      - material(object) -- 当action为5开头时才会有这个键值
        - _id `String` -- 物资_id
        - id `Number` -- 物资编号
        - type `String` -- 物资类型
        - description `Date(String)` -- 物资描述
        - import_time `Date(String)` -- 入库时间
        - estimated_export_time   `Date(String)` -- 估计出库时间
        - height `Number` -- 物资长度，该系统中固定为1
        - width `Number` -- 物资长度，该系统中固定为1
        - length `Number` -- 物资长度，该系统中固定为1
        - status `Number` -- 状态码，详见 db doc
        - last_migrations `String` -- 最近一次搬运记录_id
        - location_update_time `Date(String)` -- 位置更新时间
        - from_repository `number` -- 原仓库, 仓库id, 0表示入库
        - from_location `number` -- 原位置, 原位置的id
        - from_layer `number` -- 原层
        - to_repository `number` -- 目标仓库, 仓库id, -1表示出库
        - to_location `number` -- 目标位置, 目标位置的id
        - to_layer `number` -- 目标层
      - error(object) -- 当action为6开头时才会有这个键值
        - _id `String` -- 错误_id
        - repository `Number` --  错误仓库
        - location `Number` --  错误位置
        - layer `Number` -- 错误所在的层
        - material `Number` -- 物资id, 如果错误码为2，则为空值
        - image `Number` --  照相图片，错误照片，圈出错误


+ Parameters
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    [{
        "_id": "dsafdsadsaf32413141kl2",
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": "",
        "staff": {
            "_id": "dsafdsadsaf32413141kl2",
            "name": "因幡帝",
            "account": "inaba_tewi",
            "passwd": "123456",
            "sex": 0,
            "age": 222,
            "permission": 1,
            "signup_time": 1491451593158,
            "last_login_time": 1491451593158
        },
        "material": {
            "_id": "dsafdsadsaf32413141kl2",
            "id": 1491451593158,
            "type": "tester",
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 2,
            "status": 300,
            "from_repository": 2,
            "from_location": 0,
            "from_layer": 0,
            "to_repository": 12,
            "to_location": 1,
            "to_layer": 0,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }, ...]
    
### 接受任务 [POST]

> 给任务填入staff，设置start_time为当前时间，修改status

**Request Body**

- account `String`  -- 职员账户

**Response Body**

- (array)
    - (object)
      - _id `String` -- 任务_id
      - action `Number` --  动作，详见 db doc
      - status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
      - publish_time `Date`-- 任务发布时间
      - start_time  `Date` --   任务开始时间
      - remark     `Date`  --   任务附加评语, 一般用于任务取消时
      - material(object) -- 当action为5开头时才会有这个键值
        - _id `String` -- 物资_id
        - id `Number` -- 物资编号
        - type `String` -- 物资类型
        - description `Date(String)` -- 物资描述
        - import_time `Date(String)` -- 入库时间
        - estimated_export_time   `Date(String)` -- 估计出库时间
        - height `Number` -- 物资长度，该系统中固定为1
        - width `Number` -- 物资长度，该系统中固定为1
        - length `Number` -- 物资长度，该系统中固定为1
        - status `Number` -- 状态码，详见 db doc
        - last_migrations `String` -- 最近一次搬运记录_id
        - location_update_time `Date(String)` -- 位置更新时间
        - from_repository `number` -- 原仓库, 仓库id, 0表示入库
        - from_location `number` -- 原位置, 原位置的id
        - from_layer `number` -- 原层
        - to_repository `number` -- 目标仓库, 仓库id, -1表示出库
        - to_location `number` -- 目标位置, 目标位置的id
        - to_layer `number` -- 目标层
      - error(object) -- 当action为6开头时才会有这个键值
        - _id `String` -- 错误_id
        - repository `Number` --  错误仓库
        - location `Number` --  错误位置
        - layer `Number` -- 错误所在的层
        - material `Number` -- 物资id, 如果错误码为2，则为空值
        - image `Number` --  照相图片，错误照片，圈出错误

+ Request (application/json)
    {
        "account": "2314dsf"
    }
     
+ Response 200 (application/json)

    [{
        "_id": "dsafdsadsaf32413141kl2",
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "remark": "",
        "material": {
            "_id": "dsfklasdafkasfa",
            "id": 1491451593158,
            "type": "tester",
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 2,
            "status": 300,
            "from_repository": 2,
            "from_location": 12,
            "from_layer": 1,
            "to_repository": -1,
            "to_location": 0,
            "to_layer": 0,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }, ...]

## Error [/error/task/{id}]

+ Parameters
    + id (String, required) - 任务的_id
    
### 完成错误任务 [PATCH]

> 完成任务的同时，需要修改相应错的fixed字段。

+ Response 200 (application/json)
    {}

## Staff's Task [/staff/{sid}/task/{id}]

+ Parameters
    + sid (String, required) - 职员的_id
    + id (String, required) - 任务的_id
    
    
### 开始执行任务 [PATCH]

> 当微信扫到货物时，开始任务。修改status。如果任务为出库，则直接结束，结束时间为当前时间加上某一个固定值；

+ Response 200 (application/json)
    {}

## Staff's Tasks [/staff/:sid/tasks]

+ Parameters
    + sid (String, required) - 职员的_id

### 删除所有任务 [DELETE]
> 删除该staff所有任务的staff字段，同时修改status为0

+ Response 200 (application/json)
    {}
    
### 完成多个搬运任务 [PATCH]

> 该接口在用户完成搬运扫货架上的条形码时调用，接口通过摄像头拍摄相应货架，通过图形技术获得该货架上所有货物，然后过滤得到处于搬运状态且为该职员的货物，然后完成这些货物相应的任务。在完成这些任务时，需要给相应migration的date赋值。

**Request Body**

- repository `Number` (required) -- 所在仓库id
- location `Number` (required) -- 所在位置id

**Response 200**

+ (array)
    + (string) -- 完成的任务的_id 

+ Request (application/json)
    {
        "repository": 2,
        "location": 3,
    }

+ Response 200 (application/json)
    ["dsaf32141", "adsfkihuihbj3241908"]


### 获取特定职员的任务数量 [HEAD]

> 当你希望一页一页地获取任务时，你可以先通过该接口获取所有任务的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 任务数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20
    
### 获取特定职员的任务 [GET /staff/{sid}/tasks{?page, limit}]

**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 任务_id
      - action `Number` --  动作，详见 db doc
      - status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
      - publish_time `Date`-- 任务发布时间
      - start_time  `Date` --   任务开始时间
      - end_time   `Date`  --   任务结束时间
      - remark     `Date`  --   任务附加评语, 一般用于任务取消时
      - material(object) -- 当action为5开头时才会有这个键值
        - _id `String` -- 物资_id
        - id `Number` -- 物资编号
        - type `String` -- 物资类型
        - description `Date(String)` -- 物资描述
        - import_time `Date(String)` -- 入库时间
        - estimated_export_time   `Date(String)` -- 估计出库时间
        - height `Number` -- 物资长度，该系统中固定为1
        - width `Number` -- 物资长度，该系统中固定为1
        - length `Number` -- 物资长度，该系统中固定为1
        - status `Number` -- 状态码，详见 db doc
        - last_migrations `String` -- 最近一次搬运记录_id
        - location_update_time `Date(String)` -- 位置更新时间
        - from_repository `number` -- 原仓库, 仓库id, 0表示入库
        - from_location `number` -- 原位置, 原位置的id
        - from_layer `number` -- 原层
        - to_repository `number` -- 目标仓库, 仓库id, -1表示出库
        - to_location `number` -- 目标位置, 目标位置的id
        - to_layer `number` -- 目标层
      - error(object) -- 当action为6开头时才会有这个键值
        - _id `String` -- 错误_id
        - repository `Number` --  错误仓库
        - location `Number` --  错误位置
        - layer `Number` -- 错误所在的层
        - material `Number` -- 物资id, 如果错误码为2，则为空值
        - image `Number` --  照相图片，错误照片，圈出错误
     

+ Parameters
    + sid (String, required) - 职员的_id
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
        "_id": "dsafdsadsaf32413141kl2",
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": "",
        "material": {
            "_id": "dsafdsadsaf32413141kl2",
            "id": 1491451593158,
            "type": "tester",
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 2,
            "status": 300,
            "from_repository": 2,
            "from_location": 12,
            "from_layer": 1,
            "to_repository": -1,
            "to_location": 0,
            "to_layer": 0,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }, ...]

## Task with Migration [/migration/{id}/task]

+ Parameters
    + id (String, required) - migration数据的 _id
    
### 获取特定搬运信息的相应任务 [GET]

**Response 200 Body**

- _id `String` -- 任务_id
- action `Number` --  动作，详见文档 db doc
- status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
- publish_time `Date`-- 任务发布时间
- start_time  `Date` --   任务开始时间
- end_time   `Date`  --   任务结束时间
- remark     `Date`  --   任务附加评语, 一般用于任务取消时
- staff(object)
  - _id `String` -- 职员_id
  - name `String`  -- 职员名
  - account `String`  -- 职员账户
  - passwd `String`  -- 职员密码
  - sex `Number`  -- 性别，0女1男
  - age `Number`   -- 年龄
  - permission `Number`  -- 职员权限，0管理员1员工99root
  - signup_time `Date`  -- 注册时间
  - last_login_time `Date`  -- 最近登录时间
- material(object)
  - _id `String` -- 物资_id
  - id `Number` -- 物资编号
  - type `String` -- 物资类型
  - description `Date(String)` -- 物资描述
  - import_time `Date(String)` -- 入库时间
  - estimated_export_time   `Date(String)` -- 估计出库时间
  - height `Number` -- 物资长度，该系统中固定为1
  - width `Number` -- 物资长度，该系统中固定为1
  - length `Number` -- 物资长度，该系统中固定为1
  - repository_id `Number` -- 储存仓库的id
  - location_id `Number` -- 仓库中位置的id
  - layer `Number` -- 位置中的层
  - status `Number` -- 状态码，详见 db doc
  - last_migrations `String` -- 最近一次搬运记录_id
  - location_update_time `Date(String)` -- 位置更新时间

+ Response 200 (application/json)

    {
        "_id": "dsafdsadsaf32413141kl2",
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": "",
        "staff": {
            "_id": "dsafdsadsaf32413141kl2",
            "name": "因幡帝",
            "account": "inaba_tewi",
            "passwd": "123456",
            "sex": 0,
            "age": 222,
            "permission": 1,
            "signup_time": 1491451593158,
            "last_login_time": 1491451593158
        },
        "material": {
            "_id": "dsafdsadsaf32413141kl2",
            "id": 1491451593158,
            "type": "tester",
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 2,
            "repository_id": 3,
            "location_id": 2,
            "layer": 0,
            "status": 300,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }

# Group Staff


## Staff [/staff/{id}]

### 获取特定用户的信息 [GET]

+ Parameters
    + id (String, required) - 职员的_id

**Response 200 Body**

- _id `String` -- 职员_id
- name `String` -- 职员名
- account `String` -- 职员账户
- passwd `String` -- 职员密码
- sex `Number` -- 性别，0女1男
- age `Number`  -- 年龄
- permission `Number` -- 职员权限，0管理员1员工99root
- signup_time `Date` -- 注册时间
- last_login_time `Date` -- 最近登录时间

+ Response 200 (application/json)

    {
        "_id": "dsfa3241lk32j41",
        "name": "因幡帝",
        "account": "inaba_tewi",
        "passwd": "123456",
        "sex": 0,
        "age": 222,
        "permission": 1,
        "signup_time": 1491451593158,
        "last_login_time": 1491451593158
    }

### 修改职员的部分信息 [PATCH]

>该接口用于修改部分信息。请求体为一个对象，它的keys都是要修改的键，values是新的值

**Request Body**

- key: value  -- key都是要修改的键，value是新的值，可多对


+ Request (application/json)
    {
        "age": 4,
        "permission": 99,
    }

+ Response 200 (application/json)
    {}

### 删除特定职员 [DELETE]

+ Response 200 (application/json)
    {}

## Auth [/staff/auth]

### 登录 [POST]

**Request Body**

- account `String` (required) -- 职员账户
- passwd `String` (required) -- 职员密码

**Response 200 Body**

- _id `String` -- 职员_id
- name `String` -- 职员名
- account `String` -- 职员账户
- sex `Number` -- 性别，0女1男
- age `Number`  -- 年龄
- permission `Number` -- 职员权限，0管理员1员工99root
- signup_time `Date` -- 注册时间
- last_login_time `Date` -- 最近登录时间

+ Request (application/json)
    {
        "account": "fuck1234",
        "passwd": "sqs21995"
    }
    
+ Response 200 (application/json)
    {
        "_id": "dsfa3241lk32j41",
        "name": "因幡帝",
        "account": "inaba_tewi",
        "sex": 0,
        "age": 222,
        "permission": 1,
        "signup_time": 1491451593158,
        "last_login_time": 1491451593158
    }

### 修改密码 [PATCH /staff/auth/{id}]

**Request Body**

- passwd `String` (required) -- 职员密码

+ Parameters
    + id (String, required) - 职员的_id

+ Request (application/json)
    {
        "passwd": "sqs21995"
    }
    
+ Response 200 (application/json)
    {}

## Staffs [/staffs]

### 创建一个职员 [POST]

> 该接口会对职员的permission进行检测，职员只能创建比直接permission低的职员

**Request Body**

- name `String` (optional, default: "无名") -- 职员名
- account `String` (required) -- 职员账户
- passwd `String` (required) -- 职员密码
- sex `Number` (optional, default: 1) -- 性别，0女1男
- age `Number`  (optional, default: 1) -- 年龄
- permission `Number` (optional, default: 1) -- 职员权限，0管理员1员工99root
- signup_time `Date` (optional, default: now) -- 注册时间
- last_login_time `Date` (optional) -- 最近登录时间

**Response 201 Body**

- _id `String` -- 职员_id
- name `String` -- 职员名
- account `String` -- 职员账户
- passwd `String` -- 职员密码
- sex `Number` -- 性别，0女1男
- age `Number`  -- 年龄
- permission `Number` -- 职员权限，0管理员1员工99root
- signup_time `Date` -- 注册时间
- last_login_time `Date` -- 最近登录时间

+ Request (application/json)
    {
        "name": "铃仙·优昙华院·因幡",
        "account": "reisen_udongein_inaba",
        "passwd": "123456",
        "sex": 0,
        "age": 130,
        "permission": 1,
        "signup_time": 1491451593158,
        "last_login_time": 1491451593158
    }

+ Response 201 (application/json)
    {
        "_id": "dsfa3241lk32j41",
        "name": "因幡帝",
        "account": "inaba_tewi",
        "passwd": "123456",
        "sex": 0,
        "age": 222,
        "permission": 1,
        "signup_time": 1491451593158,
        "last_login_time": 1491451593158
    }


### 获取职员的数量 [HEAD]

> 当你希望一页一页地获取职员时，你可以先通过该接口获取所有职员的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 职员数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20


### 获取所有职员信息 [GET /staffs{?page, limit}]


**Response 200 Body**

- (array)
    - (object)
      - _id `String` -- 职员_id
      - name `String`  -- 职员名
      - account `String`  -- 职员账户
      - passwd `String`  -- 职员密码
      - sex `Number`  -- 性别，0女1男
      - age `Number`   -- 年龄
      - permission `Number`  -- 职员权限，0管理员1员工99root
      - signup_time `Date`  -- 注册时间
      - last_login_time `Date`  -- 最近登录时间

+ Parameters
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
        "_id": "dsfa3241lk32j41",
        "name": "因幡帝",
        "account": "inaba_tewi",
        "passwd": "123456",
        "sex": 0,
        "age": 222,
        "permission": 1,
        "signup_time": 1491451593158,
        "last_login_time": 1491451593158
    }, ...]

### 删除所有职员 [DELETE]

> 该接口只提供给root用户，或用于开发测试，慎用！

+ Response 200 (application/json)
    {}
    
# Group Exportinfo

## Exportinfos [/exportinfos]

### 获取出库信息数量 [HEAD]

> 当你希望一页一页地获取时，你可以先通过该接口获取所有的数量，然后使用下一个接口。

**Response 200 HEAD**

- num `Number` -- 出库信息数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    + Headers
        num: 20


### 获取所有出库信息 [GET /exportinfos{?page, limit}]


**Response 200 HEAD**

- (array)
    - (object)
      - _id `String` -- 出库信息的_id
      - actual_export_time `Date` -- 实际出库时间
      - destination `String` -- 去向 
      - from_repository `Number` -- 原仓库
      - material (object) -- 物资
        - _id `String` -- 物资_id
        - id `Number` -- 物资编号
        - type `String` -- 物资类型
        - description `Date(String)` -- 物资描述
        - import_time `Date(String)` -- 入库时间
        - estimated_export_time   `Date(String)` -- 估计出库时间
        - height `Number` -- 物资长度，该系统中固定为1
        - width `Number` -- 物资长度，该系统中固定为1
        - length `Number` -- 物资长度，该系统中固定为1
        - repository_id `Number` -- 储存仓库的id
        - location_id `Number` -- 仓库中位置的id
        - layer `Number` -- 位置中的层
        - status `Number` -- 状态码，详见 db doc
        - last_migrations `String` -- 最近一次搬运记录_id
        - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + page (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
        + Default: 0
    + limit (Number, optional) - 每页几项
        + Default: 10
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
        "_id": "dsfkaljh3214",
        "actual_export_time": "2017-04-06T04:57:36.801Z",
        "destination": "幻想乡",
        "from_repository": 2,
        "material": {
          "_id": "dsfkaljh3214",
          "id": 1491451593158,
          "type": "tester",
          "description": "wonderful repository",
          "import_time": "2017-04-06T04:57:36.801Z",
          "estimated_export_time": "2017-04-06T04:57:36.801Z",
          "height": 1,
          "width": 1,
          "length": 2,
          "repository_id": 3,
          "location_id": 2,
          "layer": 0,
          "status": 300,
          "last_migrations": "1234",
          "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }, ...]
