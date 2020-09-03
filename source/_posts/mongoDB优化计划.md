---
title: mongoDB优化计划
---

为了应对暑期用户量的高增长，MongoDB数据库的优化迫在眉睫。

优化内容包换两点：
1. 索引优化
2. 语句优化

## 涉及collection


## 准备索引

各个collection的负责人根据项目中的字段使用，及张宇同学的[索引分享](https://git.neoclub.cn/Uki/uki-doc/src/master/wiki/%E7%B4%A2%E5%BC%95%E5%88%86%E4%BA%AB.md)，以一星，二星，三星索引为指导思想，设计新的索引体系。
维护至我们的[表结构文档](https://git.neoclub.cn/Uki/uki-doc/src/master/new_api/uki_%E8%A1%A8%E7%9A%84%E8%AE%BE%E8%AE%A10.1.md)。

格式如下：

**demo表结构**

| 字段名 | 类型 | 释义 | 索引 |
| ----- | --- | ---- | --- |
| name | str | 名字 |  |
| sex | int | 性别 | {"age": 1, "sex": 1} |
| age | int | 年龄 |  |

即，在原表结构上增加`索引`列，填入涉及索引。如`{"sex": 1}`表示在`sex`字段上建立正序排列的单键索引，`{"age": 1, "sex": 1}`表示在`age, sex`字段上建立联合索引。

各负责人完成表结构的索引设计后，分别由`---`, `---`初审，通过后于`周一晚七点`以投屏的形式向所有同事阐述索引设计。

## 优化查询

此时根据新的索引体系，优化collection的查询语句。各服务新建一个`feature/refactor_indexs`分支进行开发。代码review工作由`孙坤坤，樊金东`负责，`周二完成`。简述一些优化方式：

### sort查询

```python
await mongo.relationship.find(
	{"toUserId": user_id, "status": 1, "ignore": False, "black": {"$ne": True}},
	projection = {"_id": 0, "fromUserId": 1, "remark": 1},
	sort=[("updatedAt", -1)],
	limit=20)
```
此时至少要保证`relationship`存在 `{"toUserId": user_id, "status": 1, "updatedAt": -1}`的复合索引，以防止每次查询都要在数据库中遍历所有相关文档后，在内存中排序其前20条。

### 数据是否存在查询

```python
follow = await mongo.relationship.find_one(
	{
		"fromUserId": user_id,
		"toUserId": friend_id,
		"status": 1
	},
	projection={"_id": 1})
if follow:
	# 已关注
else:
	# 未关注
```
这里是为了知道两个人是否已经关注过。为了达到三星索引的形式，需要建立`{"fromUserId": 1,"toUserId": 1, "status": 1}`形式的复合索引，同时修改`projection={"_id": 0, "status": 1}`。此时即可达到仅查询索引即可返回(不需要查询集合文档), 如下：
```python
await mongo.relationship.find_one(
	{
		"fromUserId": user_id,
		"toUserId": friend_id,
		"status": 1
	},
	projection={"_id": 0, "status": 1}) # 返回索引中的任意字段均可
```

### count/count_documents查询

暂无好的解决方案

## 线上压测

基于线上数据库备份，构建一个新的mongo数据库。

1. 各collection负责人建立索引
1. 将相关mongo语句填入压测脚本

测试人员，开始对接口进行压测，各服务负责人依据数据库的慢日志，审计日志等观察数据库操作是否异常。

## 停服上线

停服时间为: `周五早上3点~8点`

各collection负责人于停服后：

1. 开始建立新的索引体系。
2. 发布feature/refactor_indexs分支代码

等待索引完成，测试回归验证，停服结束。