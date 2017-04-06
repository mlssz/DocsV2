FORMAT: 1A

# Wonderful Repository API

## Tips
- host: http://o1.mephis.me:8234
- db doc: [https://mlssz.github.io/DocsV2/dev_docs/db_design/index.html](https://mlssz.github.io/DocsV2/dev_docs/db_design/index.html)
- 响应吗 `501` 表示 `Not Implemented`, 即接口未写完
- `Date(String)` 表示 日期字符串
- _id 为 mongodb 中数据项的 pk

## readme

**url 中的others参数作为筛选要求**
> 下列 reponse body 结构表示，返回值为一个array，array的元素是object， 以及object的键值

others结构：

- (array)
    - (object)
        - key `String` (required) -- 需要筛选的键
        - value `T | Array<T>` (optional) -- 如果value为单个值，则直接匹配，如果value为数组，则进行多值匹配
        - region `Array<T>` (optional) -- 该值为包含两个元素的数组，[from, to]

# Group Material

## Material [/material/{id}]

+ Parameters
    + id (String) - 物资的id

### 获取一个特定的物资 [GET]

 **Response 200 Body**

- id `Number` -- 物资编号
- type `Number` -- 物资类型
- description `Date(String)` -- 物资描述
- import_time `Date(String)` -- 入库时间
- estimated_export_time   `Date(String)` -- 估计出库时间
- height `Number` -- 物资长度，该系统中固定为1
- width `Number` -- 物资长度，该系统中固定为1
- length `Number` -- 物资长度，该系统中固定为1
- repository_id `Number` -- 储存仓库的id
- location_id `Number` -- 仓库中位置的id
- status `Number` -- 状态码，详见 db doc
- last_migrations `String` -- 最近一次搬运记录_id
- location_update_time `Date(String)` -- 位置更新时间

+ Response 200 (application/json)
    {
        "id": 1491451593158,
        "type": 0,
        "description": "wonderful repository",
        "import_time": "2017-04-06T04:57:36.801Z",
        "estimated_export_time": "2017-04-06T04:57:36.801Z",
        "height": 1,
        "width": 1,
        "length": 1,
        "repository_id": 3,
        "location_id": 2,
        "status": 300,
        "last_migrations": "1234",
        "location_update_time": "2017-04-06T04:57:36.801Z"
    }

### 修改物资的部分信息 [PATCH]

>该接口用于修改物资的部分信息。请求体为一个对象，它的keys都是要修改的键，values是新的值

**Request Body**

- key: value  -- key都是要修改的键，value是新的值，可多对


**Response 400 Body**

- error `String` -- 请求体错误具体原因

+ Request (application/json)
    {
        "repository_id": 4,
        "location_id": 5,
    }

+ Response 200 (application/json)
    {}

+ Response 400 (application/json)
    {
        "error": "Your request body is wrong!"
    }

### 删除特定物资 [DELETE]

+ Response 200 (application/json)
    {}

## migrations [/material/:id/migrations]

+ Parameters
    + id (String) - 物资的id

### 查看物品移动记录数量 [GET /material/:id/migrations]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 移动记录数量

+ Response 200 (application/json)
    {
        "num": 2
    }
    
### 查看一个物品的移动记录 [GET /material/:id/migrations(?page, limit)]

**Response 200 Body**

- material `ObjectId`-- 物资_d
- date `date` -- 搬运时间
- from_repository `number` -- 原仓库, 仓库id, 0表示入库
- from_location `number` -- 原位置, 原位置的id
- to_repository `number` -- 目标仓库, 仓库id, -1表示出库
- to_location `number` -- 目标位置, 目标位置的id

+ Parameters
    + id (String) - 物资的id
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    
+ Response 200 (application/json)

    [{
        "date": "2017-04-06T04:57:36.801Z",
        "from_repository": 2,
        "from_location": 12,
        "to_repository": -1,
        "to_location": 0
    }, ...]

## Materials [/materials]

### 创建一个物资 [POST]

> 该接口在仓储系统管理人员输入物资编号、物资类型、物资描述、入库时间、估计出库时间，输入信息上报给服务器时调用，服务请应当创建一个相应的material实例储存到数据库。

**Request Body**

- id `Number` (optional, default: now) -- 物资编号，默认值为当前时间戳
- type `Number` (optional, default: 0) -- 物资类型
- description `String` (optional, default: "") -- 物资描述
- import_time `Date`  (optional, default: now) -- 物资入库时间
- estimated_export_time `Date` (optional) -- 该值为时间数值, date.getTime(), 估计出库时间
- height `Number` (optional, default: 1) --  物资高度，该系统中固定为1
- width `Number` (optional, default: 1) --  物资高度，该系统中固定为1
- length `Number` (optional, default: 1) --  物资高度，该系统中固定为1
- repository_id `Number` (required) --  储存仓库的id
- location_id `Number` (required) -- 仓库中位置的id

**Response 400 Body**

- error `String` -- 请求体错误具体原因

+ Request (application/json)
    {
        "id": 3214,
        "type": 0,
        "description": "wonderful repository",
        "import_time": 1491451593158,
        "estimated_export_time": 1491451593158,
        "height": 1,
        "width": 1,
        "length": 1,
        "repository_id": 2,
        "location_id":  3,
    }

+ Response 201 (application/json)
    {}

+ Response 400 (application/json)
    {
        "error": "Your request body is wrong!"
    }

### 获取所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    {
        "num": 20
    }


### 获取所有物资信息 [GET /materials(?page, limit)]


**Response 200 Body**

- (array)
    - (object)
      - id `Number` -- 物资编号
      - type `Number` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "id": 1491451593158,
         "type": 0,
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 1,
         "repository_id": 3,
         "location_id": 2,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]

### 删除所有物资 [DELETE]

> 该接口只提供给root用户，或用于开发测试，慎用！

+ Response 200 (application/json)
    {}

## Repository [/repository/:id/materials]

+ Parameters
    + id (required) - 仓库id

### 获取特定仓库所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    {
        "num": 20
    }

### 获取特定仓库中的物资 [GET /repository/:id/materials(?page, limit)]

**Response 200 Body**

- (array)
    - (object)
      - id `Number` -- 物资编号
      - type `Number` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + id (required) - 仓库id
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "id": 1491451593158,
         "type": 0,
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 1,
         "repository_id": 3,
         "location_id": 2,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]

## Location [/repository/:rid/location/:lid/materials]

+ Parameters
    + rid (required) - 仓库id
    + lid (required) - 位置id

### 获取特定位置所有物资的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 物资数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    {
        "num": 20
    }

### 获取特定位置的物资 [GET /repository/:rid/location/:lid/materials(?page, limit)]

**Response 200 Body**

- (array)
    - (object)
      - id `Number` -- 物资编号
      - type `Number` -- 物资类型
      - description `String` -- 物资描述
      - import_time `Date(String)` -- 入库时间
      - estimated_export_time   `Date(String)` -- 估计出库时间
      - height `Number` -- 物资长度，该系统中固定为1
      - width `Number` -- 物资长度，该系统中固定为1
      - length `Number` -- 物资长度，该系统中固定为1
      - repository_id `Number` -- 储存仓库的id
      - location_id `Number` -- 仓库中位置的id
      - status `Number` -- 状态码，详见 db doc
      - last_migrations `String` -- 最近一次搬运记录_id
      - location_update_time `Date(String)` -- 位置更新时间

+ Parameters
    + rid (required) - 仓库id
    + lid (required) - 位置id
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
         "id": 1491451593158,
         "type": 0,
         "description": "wonderful repository",
         "import_time": "2017-04-06T04:57:36.801Z",
         "estimated_export_time": "2017-04-06T04:57:36.801Z",
         "height": 1,
         "width": 1,
         "length": 1,
         "repository_id": 3,
         "location_id": 2,
         "status": 300,
         "last_migrations": "1234",
         "location_update_time": "2017-04-06T04:57:36.801Z"
    }, ...]


# Group Repository

## Locations [/repository/:id/locations]

### 返回仓库中所有位置的信息 [GET]

**Response 200 Body**

- (array)
    - (object)
      - id `Number` -- 位置id, 该值用于定位,顺时针算
      - place `Number` -- 块号, 该值仅用于显示，根据文档图，分四个大块，该值表示第几块（顺时针算）
      - label `String` -- 位置标记, 该值仅用于显示，显示location时其上的标记
      - available_space `Number` -- 剩余空间, 该位置剩余的高度。(2*3*10 立方米)
      - materials `Number` -- 存放的物资的数量

+ Response 200 (application/json)
    [{
        id: 2,
        place: 3,
        label: "23",
        available_space: 34,
        materials: 12
    }, ...]
    
# Group Task

## Staff [/staff/:accout/tasks]

+ Parameters
    + account (String, required) - 职员的账户

### 获取特定职员的任务数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 任务数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    {
        "num": 20
    }
    
### 获取特定职员的任务 [GET /staff/:account/tasks(?page, limit)]

**Response 200 Body**

- (array)
    - (object)
      - action `Number` --  动作, 动作码详细见下方
      - status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
      - publish_time `Date`-- 任务发布时间
      - start_time  `Date` --   任务开始时间
      - end_time   `Date`  --   任务结束时间
      - remark     `Date`  --   任务附加评语, 一般用于任务取消时
      - material(object) -- 当action为5开头时才会有这个键值
        - id `Number` -- 物资编号
        - type `Number` -- 物资类型
        - description `Date(String)` -- 物资描述
        - import_time `Date(String)` -- 入库时间
        - estimated_export_time   `Date(String)` -- 估计出库时间
        - height `Number` -- 物资长度，该系统中固定为1
        - width `Number` -- 物资长度，该系统中固定为1
        - length `Number` -- 物资长度，该系统中固定为1
        - repository_id `Number` -- 储存仓库的id
        - location_id `Number` -- 仓库中位置的id
        - status `Number` -- 状态码，详见 db doc
        - last_migrations `String` -- 最近一次搬运记录_id
        - location_update_time `Date(String)` -- 位置更新时间
      - err_repository `Number` --  错误仓库，当action为6开头时才会有这个键值
      - err_location `Number` --  错误位置，当action为6开头时才会有这个键值
     

+ Parameters
    + account (String, required) - 职员的账户
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": ""
        "material": {
            "id": 1491451593158,
            "type": 0,
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 1,
            "repository_id": 3,
            "location_id": 2,
            "status": 300,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }, ...]

## Migration [/migration/:id/task]

+ Parameters
    + id (String, required) - migration数据的 _id
    
### 获取特定搬运信息的相应任务 [GET]

**Response 200 Body**

- action `Number` --  动作, 动作码详细见下方
- status `Number` -- 任务状态, 0未开始1进行中2完成3任务取消
- publish_time `Date`-- 任务发布时间
- start_time  `Date` --   任务开始时间
- end_time   `Date`  --   任务结束时间
- remark     `Date`  --   任务附加评语, 一般用于任务取消时
- staff(object)
  - name `String`  -- 职员名
  - account `String`  -- 职员账户
  - passwd `String`  -- 职员密码
  - sex `Number`  -- 性别，0女1男
  - age `Number`   -- 年龄
  - permission `Number`  -- 职员权限，0管理员1员工99root
  - signup_time `Date`  -- 注册时间
  - last_login_time `Date`  -- 最近登录时间
- material(object)
  - id `Number` -- 物资编号
  - type `Number` -- 物资类型
  - description `Date(String)` -- 物资描述
  - import_time `Date(String)` -- 入库时间
  - estimated_export_time   `Date(String)` -- 估计出库时间
  - height `Number` -- 物资长度，该系统中固定为1
  - width `Number` -- 物资长度，该系统中固定为1
  - length `Number` -- 物资长度，该系统中固定为1
  - repository_id `Number` -- 储存仓库的id
  - location_id `Number` -- 仓库中位置的id
  - status `Number` -- 状态码，详见 db doc
  - last_migrations `String` -- 最近一次搬运记录_id
  - location_update_time `Date(String)` -- 位置更新时间

+ Response 200 (application/json)

    {
        "action": 500,
        "status": 1,
        "publish_time": "2017-04-06T04:57:36.801Z",
        "start_time": "2017-04-06T04:57:36.801Z",
        "end_time":  "2017-04-06T04:57:36.801Z",
        "remark": ""
        "staff": {
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
            "id": 1491451593158,
            "type": 0,
            "description": "wonderful repository",
            "import_time": "2017-04-06T04:57:36.801Z",
            "estimated_export_time": "2017-04-06T04:57:36.801Z",
            "height": 1,
            "width": 1,
            "length": 1,
            "repository_id": 3,
            "location_id": 2,
            "status": 300,
            "last_migrations": "1234",
            "location_update_time": "2017-04-06T04:57:36.801Z"
        }
    }

# Group Staff

## Staff [/staff/:account]

### 获取特定用户的信息 [GET]

+ Parameters
    + account (String, required) - 职员的账户

**Response 200 Body**

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


**Response 400 Body**

- error `String` -- 请求体错误具体原因

+ Request (application/json)
    {
        "age": 4,
        "permission": 99,
    }

+ Response 200 (application/json)
    {}

+ Response 400 (application/json)
    {
        "error": "Your request body is wrong!"
    }

### 删除特定职员 [DELETE]

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

**Response 400 Body**

- error `String` -- 请求体错误具体原因

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
    {}

+ Response 400 (application/json)
    {
        "error": "Your request body is wrong!"
    }

### 获取职员的数量 [HEAD]

> 当你希望一页一页地获取物资时，你可以先通过该接口获取所有物资的数量，然后使用下一个接口。

**Response 200 Body**

- num `Number` -- 职员数量

+ Parameters
    + others (optional) - 见开头readme

+ Response 200 (application/json)
    {
        "num": 20
    }


### 获取所有职员信息 [GET /staffs(?page, limit)]


**Response 200 Body**

- (array)
    - (object)
      - name `String`  -- 职员名
      - account `String`  -- 职员账户
      - passwd `String`  -- 职员密码
      - sex `Number`  -- 性别，0女1男
      - age `Number`   -- 年龄
      - permission `Number`  -- 职员权限，0管理员1员工99root
      - signup_time `Date`  -- 注册时间
      - last_login_time `Date`  -- 最近登录时间

+ Parameters
    + page: 0 (Number, optional) - 当前第几页, 当该值为-1时，返回所有的物资
    + limit: 10 (Number, optional) - 每页几项
    + others (optional) - 见开头readme

+ Response 200 (application/json)

    [{
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
