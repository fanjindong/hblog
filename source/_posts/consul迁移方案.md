---
title: consul 迁移至 ACM
---

## 1.问题背景

目前公司后端在动态配置这块使用的是consul。主要问题是：

1. 没有权限管理，任何人都能修改配置
2. 自建服务，且没有机制来保证高可用

## 2.调研结果

经过调研，选择阿里云的`应用配置管理ACM`。优点如下：

1. 基于阿里云保障权限控制
2. 云服务保障高可用
3. 配置监听后，可实现配置变更自动推送
4. 所有配置监听、更改和版本信息自动记录在案

ACM页面示例

![](/Users/fanjindong/Downloads/acm页面.jpg)

## 3.服务接入

为了方便各服务接入，已经将先关逻辑组件化，[配置监听文档](http://192.168.3.3:20000/neoclub/discovery.html#sanic-config)

详细接入步骤：

1.访问阿里云ACM在三个环境下建立项目配置，并将项目的consul配置移到ACM，并记录下DATA ID

2.依照文档修改项目代码，举例如下

```python
# 新增代码
from neoclub.discovery import add_config_watcher

@app.listener('before_server_start')
async def setup_db(app, loop):
    ...
    add_config_watcher(app, "uki-user") # 各自服务的Data ID
```

```python
# 移除代码
from apscheduler.schedulers.asyncio import AsyncIOScheduler
from utils.discovery import query_config

@app.listener('after_server_start')
async def notify_server_started(app, loop):
    # 定时任务
    scheduler = AsyncIOScheduler()
    scheduler.add_job(query_config, 'interval', seconds=10, args=[app])
    scheduler.start()

# 删除utils.discovery.py 文件
```

3.测试环境自测无误后，可申请发布至线上

## 4.进度跟踪

| 服务              | 负责人 | 状态  |
|:---------------:| --- | --- |
| party           |     |     |
| record          |     |     |
| user            | 樊金东 | 已上线 |
| voice           |     |     |
| risk-background |     |     |
| sms             |     |     |
| appearance      |     |     |
| push            |     |     |
| notice          |     |     |
| relationship    |     |     |
| post            |     |     |
| block           |     |     |
| lover           |     |     |
| friend          |     |     |
| street          |     |     |
| netease         |     |     |
| storage         |     |     |
| order           |     |     |
| msg-queue       |     |     |
| task            |     |     |
| gift            |     |     |
| callback        |     |     |
| titan           |     |     |
| pay             |     |     |
| im              |     |     |
| medal           |     |     |
| ucoin           |     |     |
| privilege       |     |     |
| album           |     |     |
