## 一、项目概述

### 1.1 描述

利用 python 语言强大的爬虫库获取第三方网站的信息，然后通过分词处理和特征提取等手段将获取到的信息规范化后。随后使用优秀的机器学习算法[^1]将其规划为最适合当前游客的旅游路线。

### 1.2 功能

#### 1.2.1 概述

1. 利用爬虫或第三方 API 请求获取旅行地点的景点、交通、住宿等信息，整合后存入数据库，方便随时调用；通过用户的请求实时获取景点信息，并存入数据库。
2. 通过1获取到旅行地点的攻略信息，对其进行分词处理，提取实用的旅行路径信息再进行规划；获取到的评论信息通过语言情感库对其进行估算后，给出该景点、酒店的评分，用以作为规划时的路径点权值。
3. 对通过1、2获取的信息使用一种路径规划算法得到比较正确的规划方案
4. 用户访问网站的后台数据接口
   1. 用户接口：登录、注册
   2. 路线查询
   3. 景点信息查询：分类、分数、评论
   4. 路径规划
5. 前端页面展示
   1. 热门路线推荐：使用网络上热门的旅行攻略来指定旅行计划
   2. 景点分类查询：对热门景点进行种类划分，根据自身喜好选择旅行地点
   3. 自定义旅行计划：自定义旅行地点，通过后台路径规划算法自行规划旅行计划

#### 1.2.2 运行流程

前端操作：

![img](./img/fontendflow.png)

爬虫与分词系统：

![img](./img/spider.png)

后台 API 系统：

![img](./img/flowchart.png)

### 1.3 架构

后台框架：[TinkPHP5.0](https://github.com/top-think/think/tree/5.0) + [Dawn-API](https://github.com/liushoukun/dawn-api-demo) + [jwt](https://github.com/lcobucci/jwt/blob/3.2/README.md)

![backend](./img/backend.png)

### 1.4 约束

1. 本系统提供一个 web 端网站访问后台接口
2. 后台 API 运行于：nginx-1.13.9, PHP-7.1.14, MySQL-14.14
3. 框架版本：ThinkPHP5.0

## 二、具体需求

### 1.1 功能

#### ① 爬虫模块（5）

​	a）概述：通过爬虫获取旅游地点的必要信息，便于后期调用。

​	b）输入输出

​	c）处理

* 12306实时查询火车票（并返回座位票价，出发抵达历时信息等）
* 携程机票实时查询（未完善，需要整理数据）
* 去哪儿景点月销量查询（并使用 echart 生成热力图）
* 携程（特定地点，如厦门）景点信息查询
* 酒店信息查询、酒店信息实时对比查询推荐
* 攻略信息文本分析，分词，提取出现频率最多的地点等

#### ② 文本处理模块（5）

​	a）概述：对获取的旅行攻略进行文本分析，提取关键词、路径点、情感倾向等信息。

#### ③ 用户信息模块（3）

​	a）概述：提供用户登录注册通道，并保存登录用户规划的旅行计划。

| 功能     | 说明                   | 输入                      | 输出                           | 处理                                                         |
| -------- | ---------------------- | ------------------------- | ------------------------------ | ------------------------------------------------------------ |
| 登录     | 提供用户登录操作       | 用户名，密码              | 登录信息，认证 cookie、session | 验证用户名和密码，成功则返回Token 用于验证，失败返回失败信息 |
| 注册     | 提供注册操作           | 手机/邮箱，密码，确认密码 | 注册信息                       | 对信息进行验证，存入数据库                                   |
| 用户资料 | 获取用户的详细信息     | （opt）用户id             | 用户信息                       | 若有指定用户 id，检查权限后读取该用户信息，否则获取当前登录用户信息 |
| 用户计划 | 获取用户保存的旅游计划 | 用户 id，（opt）计划 id   | 旅行计划信息                   | 通过用户 id 读取数据库中的该用户的旅行计划                   |
| 修改信息 | 修改用户信息           | 用户 id，修改的信息       | 操作状态                       | 通过用户 id，更改数据库信息                                  |

#### ④ 景点信息模块（5）

​	a）概述：查询景点各类信息，利用景点的分类更快速的使用户确定旅行地点

| 功能         | 说明                   | 输入                           | 输出         | 处理                         |
| ------------ | ---------------------- | ------------------------------ | ------------ | ---------------------------- |
| 热门景点     | 获取热门景点的摘要信息 | 无                             | 景点摘要信息 | 查询数据库中的热门景点       |
| 景点摘要查询 | 查询景点的摘要信息     | 查询条件（分类，地点，时长等） | 景点摘要信息 | 通过查询条件在数据库中查询   |
| 景点信息查询 | 查询景点的详细信息     | 景点 id                        | 景点详细信息 | 通过提供的景点 id 查询数据库 |
| 景点分类查询 | 查询景点分类表         | （opt）查询热门                | 景点分类表   | 查询数据库                   |

#### ⑤ 路线信息模块（3）

​	a）概述：提供用户发布的已完成的旅游路线或热门的旅游攻略路线

| 功能             | 说明                       | 输入                           | 输出              | 处理                       |
| ---------------- | -------------------------- | ------------------------------ | ----------------- | -------------------------- |
| 热门路线查询     | 获取热门旅行路线的摘要信息 | 无                             | 路线摘要信息      | 查询数据库中的热门路线     |
| 路线摘要查询     | 获取路线的摘要信息         | 查询条件（分类，评分，时常等） | 路线摘要信息      | 通过查询条件在数据库中查询 |
| 路线详细信息查询 | 获取路线的详细信息         | 路线 id                        | 路线详细信息      | 通过 id 查询数据库中的信息 |
| 添加路线         | 添加用户自定义旅行路线     | 用户 id，路线表                | 操作状态，路线 id | 添加路线到数据库中         |
| 修改路线         | 修改用户自定义旅行路线     | 路线 id，修改信息              | 操作状态          | 根据路线 id 修改数据库信息 |
| 删除路线         | 删除用户自定义旅行路线     | 路线 id                        | 操作状态          | 删除信息                   |

#### ⑥ 路径规划模块（5）

​	a）概述：通过若干的景点坐标信息和距离信息，使用算法完成最优路径规划

| 功能     | 说明                   | 输入               | 输出     | 处理                 |
| -------- | ---------------------- | ------------------ | -------- | -------------------- |
| 路线规划 | 对输入的路径点进行规划 | 景点坐标，景点评分 | 路径信息 | 使用算法进行路径规划 |

### 1.2 接口（API）

前置版本号：v1

使用 RESTful API 设计模式，以下为接口通用标识（在请求`\user`接口的情况下）：

| 请求类型 | 生成路由规则     | 对应操作方法（默认） | 默认权限 |      |      |
| -------- | ---------------- | -------------------- | -------- | ---- | ---- |
| GET      | v1/user          | index                | 全体     |      |      |
| POST     | v1/user          | save                 | 全体     |      |      |
| GET      | v1/user/:id      | read                 | 操作对象 |      |      |
| PUT      | v1/user/:id      | update               | 操作对象 |      |      |
| DELETE   | v1/user/:id      | delete               | 管理员   |      |      |
| GET      | v1/user/:id/edit | edit                 | 操作对象 |      |      |
| GET      | v1/user/create   | create               | 全体     |      |      |

1. `/auth`：登录接口
   1. 只提供 POST 访问
   2. 传输用户名、密码
   3. 返回登录成功或失败的信息，同时生成认证 cookie
2. `/user`：用户信息接口
   1. 注册接口为`/create`
   2. 提供增删查改，有权限限制
3. `/plan`：路线信息接口
4. `/destination`：景点信息接口
   1. `/type`：分类信息接口
   2. `/room`：住宿信息接口
   3. `/traffic`：交通信息接口
   4. `/flight`：航班信息接口
5. `/design`：路径规划接口

### 1.3 数据库

user

| 字段名      | 描述     | 类型    | 长度 | 其他         |
| ----------- | -------- | ------- | ---- | ------------ |
| uid         | 主键     | int     | 11   | 主键         |
| name        | 用户名   | varchar | 32   |              |
| password    | 密码     | varchar | 128  | 编码后的数据 |
| root        | 权限     | int     | 3    | 1，2         |
| status      | 用户状态 | int     | 3    | 0，1         |
| create_time | 创建时间 | int     | 10   |              |
| update_time | 更新时间 | int     | 10   |              |
| is_delete   | 删除     | int     | 3    | 0，1         |

destination_type

| 字段名 | 描述   | 类型    | 长度 | 其他  |
| ------ | ------ | ------- | ---- | ----- |
| id     | 主键   | int     | 10   | 主键  |
| name   | 类型名 | varchar | 32   |       |
| pid    | 父 id  | int     | 10   | 顶级0 |

destinations

| 字段名      | 描述         | 类型    | 长度 | 其他 |
| ----------- | ------------ | ------- | ---- | ---- |
| id          | 主键         | int     | 10   | 主键 |
| name        | 景点名       | varchar | 32   |      |
| province    | 所在省       | int     | 10   | 外键 |
| city        | 所在市       | int     | 10   | 外键 |
| area        | 所在区       | varchar | 32   |      |
| type_id     | 景点类型     | int     | 10   | 外键 |
| rank        | 地区排名     | varchar | 32   |      |
| level       | 景点等级     | varchar | 16   |      |
| hot         | 热门         | int     | 3    | 0，1 |
| comment     | 评论数       | int     | 10   |      |
| cover_url   | 景点封面图片 | varchar | 256  |      |
| status      |              |         |      |      |
| create_time |              |         |      |      |
| update_time |              |         |      |      |
| is_delete   |              |         |      |      |

destinations_detail

| 字段名      | 描述         | 类型    | 长度 | 其他 |
| ----------- | ------------ | ------- | ---- | ---- |
| id          | 主键         | int     | 10   | 主键 |
| des_id      | 景点 id      | int     | 10   | 外键 |
| position    | 坐标         | varchar | 32   | x,y  |
| description | 详细描述     | varchar | 256  |      |
| location    | 位置         | varchar | 128  |      |
| cost_time   | 游玩时间     | numeric | 10   |      |
| open_time   | 开放时间     | varchar | 32   |      |
| ticket_msg  | 票价信息     | varchar | 32   |      |
| photo_url   | 景点照片地址 | varchar | 256  |      |
| status      |              |         |      |      |
| create_time |              |         |      |      |
| update_time |              |         |      |      |
| is_delete   |              |         |      |      |

pathplan

| 字段名      | 描述     | 类型    | 长度 | 其他 |
| ----------- | -------- | ------- | ---- | ---- |
| id          | 主键     | int     | 10   | 主键 |
| name        | 路线名   | varchar | 32   |      |
| des_id      | 目的地   | int     | 10   | 外键 |
| uid         | 创建用户 | int     | 10   | 外键 |
| cost_time   | 时长     | int     | 10   |      |
| go_off      | 出发时间 | varchar | 32   |      |
| description | 描述     | varchar | 256  |      |
| cover_url   | 封面地址 | varchar | 256  |      |
| hot         | 热门     | int     | 3    | 0，1 |
| like        | 点赞数   | int     | 10   |      |
| status      |          |         |      |      |
| create_time |          |         |      |      |
| update_time |          |         |      |      |
| is_delete   |          |         |      |      |

pathplan_step_index

| 字段名      | 描述    | 类型    | 长度 | 其他                    |
| ----------- | ------- | ------- | ---- | ----------------------- |
| id          | 主键    | int     | 10   | 主键                    |
| plan_id     | 路线 id | int     | 10   | 外键                    |
| steps       | 步骤数  | int     | 3    |                         |
| distance    | 总路程  | varchar | 32   | 转换为 km 或 m 之后插入 |
| node        | 景点数  | int     | 11   |                         |
| total_time  | 总时长  | int     | 3    | 天                      |
| create_time |         |         |      |                         |
| update_time |         |         |      |                         |

pathplan_step_detail

| 字段名      | 描述     | 类型    | 长度 | 其他                    |
| ----------- | -------- | ------- | ---- | ----------------------- |
| id          | 主键     | int     | 10   | 主键                    |
| step_id     | 步骤 id  | int     | 10   | 外键                    |
| number      | 序号     | int     | 3    |                         |
| start       | 起点坐标 | varchar | 32   | x,y                     |
| end         | 终点坐标 | varchar | 32   | x,y                     |
| start_des   | 起点景点 | int     | 10   | 外键                    |
| end_des     | 终点景点 | int     | 10   | 外键                    |
| now         | 所在时间 | int     | 3    | 天                      |
| distance    | 距离     | varchar | 32   | 转换为 km 或 m 之后插入 |
| create_time |          |         |      |                         |
| update_time |          |         |      |                         |

room

flight

### 1.4 性能

## 三、附录

[^1]: 参考蚁群算法