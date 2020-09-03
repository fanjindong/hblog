# redis优化方案

最近几天，redis内存使用超过80%，CPU高峰更是达到100%使用率。
针对这种情况，本次优化分两部分，第一部分为紧急止血，第二部分为长期优化。

## 1. 紧急止血

今日务必完成降低CPU使用率的目标。

### 1.1 优化redis使用

**减少非必需的查询**

以`hgetall`为例，获取hash结构的所有字段值，但是你可能只需要部分字段的值，请使用`hget`或`hmget`

一些不经常变动的数据，主动写到内存里，尽量不要绑定到sanic.app上。

```python
async def send_gift(request):
	...
    # 获取缓存中的礼物价格
    temp = await redis.get("gifts_all_dict")
    if not temp:
        temp = (await init_gift_mongo(mongo, redis))[2]
    this_gift_data = loads(temp)[str(gift_id)]
    price = int(this_gift_data["price"])

    # 礼物信息
    temp_gifts = await redis.get("gifts")
    if temp_gifts:
        gift_list = loads(temp_gifts)
    else:
        gift_list = loads((await init_gift_mongo(mongo, redis))[1])
```

建议的优化方式：

```python
# utils.cache.py

from task.main_actor import conn

redis = conn.redis

class MemoryCache(object):
    _gifts_dict = 0, None # expire, value

    @property
    def gifts_price(self):
        expire, value = self._gifts_dict
        if time.time() > expire:
            return self._init_gifts_price()
        return value

    def _init_gifts_price(self, ttl=600):
		temp = redis.get("gifts_all_dict")
		if not temp:
			temp = (init_gift_mongo(mongo, redis))[2]
		this_gift_data = loads(temp)[str(gift_id)]
		gifts_price = int(this_gift_data["price"])
        self._gifts_dict = time.time() + ttl, gifts_price
        return gifts_price

memory_cache = MemoryCache()


# handler.gift_contorl_v1.py
from utils.cache import memory_cache

async def send_gift(request):
	...
    # 获取缓存中的礼物价格
    temp = await redis.get("gifts_all_dict")
    if not temp:
        temp = (await init_gift_mongo(mongo, redis))[2]
    this_gift_data = loads(temp)[str(gift_id)]
    price = int(this_gift_data["price"])
	price = memory_cache.gifts_price

	...
```

**大value的key**

如下所示，占用内存20M+~200M+,且无过期时间。请各自梳理逻辑，对如下异常key做相应处理。

```
实例ID	DB	Key	占用内存(KB)	过期
r-bp1bce4fcac1c3e4	0	sync:voice:update:check	267778.31	no
r-bp1bce4fcac1c3e4	0	post_greet_count	250300.00	no
r-bp1bce4fcac1c3e4	0	post_attenuation_value	235373.85	no
r-bp1bce4fcac1c3e4	0	post_recent_attenuation_time	235359.91	no
r-bp1bce4fcac1c3e4	0	relationship_friend_last_read_at	166697.79	no
r-bp1bce4fcac1c3e4	0	blind_follow	124446.68	no
r-bp1bce4fcac1c3e4	0	already_like_7-7	77584.57	no
r-bp1bce4fcac1c3e4	0	temp_regis_user	72592.46	no
r-bp1bce4fcac1c3e4	0	sync:post:del:check	72204.06	no
r-bp1bce4fcac1c3e4	0	postUsersIn30m	61621.11	no
r-bp1bce4fcac1c3e4	0	sync:review:update	56696.79	no
r-bp1bce4fcac1c3e4	0	p:d:3h	47659.94	no
r-bp1bce4fcac1c3e4	0	blind_follow_voice	40930.29	no
r-bp1bce4fcac1c3e4	0	sync:post:score:check	40770.69	no
r-bp1bce4fcac1c3e4	0	p:topic	38981.81	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:signature	37769.91	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:name	37610.69	no
r-bp1bce4fcac1c3e4	0	top:892	30805.63	no
r-bp1bce4fcac1c3e4	0	temp_post_total_user	30795.93	no
r-bp1bce4fcac1c3e4	0	top:891	30792.83	no
r-bp1bce4fcac1c3e4	0	top:877	30780.75	no
r-bp1bce4fcac1c3e4	0	top:890	30779.43	no
r-bp1bce4fcac1c3e4	0	top:889	30742.94	no
r-bp1bce4fcac1c3e4	0	temp_post_pic_user	26899.20	no
r-bp1bce4fcac1c3e4	0	top:888	26610.11	no
r-bp1bce4fcac1c3e4	0	top:887	26582.57	no
r-bp1bce4fcac1c3e4	0	top:886	26553.83	no
r-bp1bce4fcac1c3e4	0	sync:voice:del:check	22272.51	no
r-bp1bce4fcac1c3e4	0	top:798	19974.91	no
r-bp1bce4fcac1c3e4	0	already_like_7-7	77287.00	no
r-bp1bce4fcac1c3e4	0	temp_regis_user	72592.46	no
r-bp1bce4fcac1c3e4	0	sync:post:del:check	72204.06	no
r-bp1bce4fcac1c3e4	0	sync:post:del:check	72169.69	no
r-bp1bce4fcac1c3e4	0	sync:post:del:check	72144.23	no
r-bp1bce4fcac1c3e4	0	sync:post:del:check	72097.97	no
r-bp1bce4fcac1c3e4	0	postUsersIn30m	61621.11	no
r-bp1bce4fcac1c3e4	0	postUsersIn30m	61495.84	no
r-bp1bce4fcac1c3e4	0	postUsersIn30m	61422.12	no
r-bp1bce4fcac1c3e4	0	postUsersIn30m	61172.93	no
r-bp1bce4fcac1c3e4	0	sync:review:update	56696.79	no
r-bp1bce4fcac1c3e4	0	sync:review:update	54550.16	no
r-bp1bce4fcac1c3e4	0	sync:review:update	53410.06	no
r-bp1bce4fcac1c3e4	0	already_like_7-7	50566.99	no
r-bp1bce4fcac1c3e4	0	sync:review:update	49651.35	no
r-bp1bce4fcac1c3e4	0	p:d:3h	47659.94	no
r-bp1bce4fcac1c3e4	0	p:d:3h	42961.82	no
r-bp1bce4fcac1c3e4	0	blind_follow_voice	40930.29	no
r-bp1bce4fcac1c3e4	0	blind_follow_voice	40930.25	no
r-bp1bce4fcac1c3e4	0	sync:post:score:check	40770.69	no
r-bp1bce4fcac1c3e4	0	p:d:3h	39734.91	no
r-bp1bce4fcac1c3e4	0	p:topic	38981.81	no
r-bp1bce4fcac1c3e4	0	p:topic	38973.19	no
r-bp1bce4fcac1c3e4	0	p:topic	38969.05	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:signature	37769.91	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:signature	37763.95	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:signature	37758.55	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:signature	37736.85	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:name	37610.69	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:name	37605.37	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:name	37600.87	no
r-bp1bce4fcac1c3e4	0	node:user:reset:count:name	37580.31	no
r-bp1bce4fcac1c3e4	0	p:topic	32811.38	no
r-bp1bce4fcac1c3e4	0	top:892	30805.63	no
r-bp1bce4fcac1c3e4	0	temp_post_total_user	30795.93	no
r-bp1bce4fcac1c3e4	0	top:891	30792.83	no
r-bp1bce4fcac1c3e4	0	top:877	30780.75	no
r-bp1bce4fcac1c3e4	0	top:890	30779.43	no
r-bp1bce4fcac1c3e4	0	top:889	30742.94	no
r-bp1bce4fcac1c3e4	0	top:892	30589.09	no
r-bp1bce4fcac1c3e4	0	top:892	30573.69	no
r-bp1bce4fcac1c3e4	0	top:891	30573.05	no
r-bp1bce4fcac1c3e4	0	top:891	30561.95	no
r-bp1bce4fcac1c3e4	0	top:890	30559.80	no
r-bp1bce4fcac1c3e4	0	top:890	30549.59	no
r-bp1bce4fcac1c3e4	0	top:877	30537.56	no
r-bp1bce4fcac1c3e4	0	top:877	30521.62	no
r-bp1bce4fcac1c3e4	0	top:889	30516.65	no
r-bp1bce4fcac1c3e4	0	top:889	30511.40	no
r-bp1bce4fcac1c3e4	0	p:d:3h	30143.79	no
r-bp1bce4fcac1c3e4	0	temp_post_pic_user	26899.20	no
r-bp1bce4fcac1c3e4	0	top:888	26610.11	no
r-bp1bce4fcac1c3e4	0	top:887	26582.57	no
r-bp1bce4fcac1c3e4	0	top:886	26553.83	no
r-bp1bce4fcac1c3e4	0	top:888	22308.08	no
r-bp1bce4fcac1c3e4	0	top:888	22283.90	no
r-bp1bce4fcac1c3e4	0	sync:voice:del:check	22272.51	no
r-bp1bce4fcac1c3e4	0	top:887	22258.47	no
r-bp1bce4fcac1c3e4	0	top:887	22255.85	no
r-bp1bce4fcac1c3e4	0	sync:voice:del:check	22254.24	no
```

**暂停未必要查询**

最坏的情况下，必需紧急停用一些redis的逻辑查询。
在降低一定的用户体验的情况下，保证线上环境redis的稳定性。

```python
# config.py
class BaseConfig(object):
    # 服务降级开关
    SVC_DOWNGRADE = 0 # 0: 正常，1: 一级降级，2: 二级降级，3: 三级降级，...


# utils.public.py

async def get_cache_remark(conf, redis, user_id, user_list):
    if conf.SVC_DOWNGRADE > 0:
        return {}

    """查询缓存中的备注"""
    dict_remark = {}
    keys_list_0 = set([])
    keys_list_1 = set([])
    if not isinstance(user_list, (list, set)):
        user_list = [user_list]
    for item in user_list:
        if item:
            result = cmp(str(user_id), str(item))
            if result == 1:
                relation_key = conf.RELATIONSHIP_USERS.format(item, user_id)
                keys_list_1.add(relation_key)
            else:
                relation_key = conf.RELATIONSHIP_USERS.format(user_id, item)
                keys_list_0.add(relation_key)
    key_list = keys_list_0 | keys_list_1
    if len(key_list) == 0:
        return dict_remark
    list_result = await redis.mget(*key_list, encoding=None)
    for i, item in enumerate(key_list):
        result = b""
        if list_result[i]:
            if item in keys_list_0:
                result = list_result[i][1:39]
            if item in keys_list_1:
                result = list_result[i][40:]
        result = result.replace(b'\u0000', b'')
        result = result.replace(b'\\x00', b'')
        result = result.replace(b'\x00', b'').decode('utf-8')
        dict_remark[item] = result
    return dict_remark



def init_label(conf, redis, mongo, mongo_cache, user_id):
    user_id_obj = to_object(user_id)
    cache_label_shield_key = conf.CACHE_LABEL_SHIELD_USER.format(user_id)

    topics = set()
    shield = set()
    black = set()
    fans = set()
    follow = set()
    like_user = set()
    unlike = set()
    unlike_post = set()

    if conf.SVC_DOWNGRADE > 1:  # 2级及以上服务降级
        # 关注的话题
        user = mongo_cache.user.find_one({'_id': user_id_obj}, projection={'topic': 1})
        topics = set([object_to_int(t['id']) for t in user.get('topic', [])])
        # 举报过的用户
        for _report in mongo.userReport.find({"userId": user_id_obj}, ['uid']).limit(50):
            shield.add(object_to_int(_report['uid']))

    if conf.SVC_DOWNGRADE > 0:  # 1级及以上服务降级
        # 拉黑， 被拉黑，关注，粉丝
        for _rel in mongo.relationship.find(
                {"$or": [{"fromUserId": user_id_obj}, {"toUserId": user_id_obj}]},
                ['fromUserId', 'toUserId', 'status', 'black']):
            if _rel['fromUserId'] == user_id_obj:
                if _rel.get('black', {}).get('status', False) is True:
                    black.add(object_to_int(_rel['toUserId']))
                    continue
                if _rel.get('status', 0) == 1:
                    follow.add(object_to_int(_rel['toUserId']))
            else:
                if _rel.get('black', {}).get('status', False) is True:
                    black.add(object_to_int(_rel['fromUserId']))
                    continue
                if _rel.get('status', 0) == 1:
                    fans.add(object_to_int(_rel['fromUserId']))

        # 动态广场不喜欢的用户
        for _item in mongo.unlike.find({"userId": user_id_obj, "count": {"$gt": 4}}, ['toId']).limit(50):
            unlike.add(object_to_int(_item['toId']))

        # 动态广场不喜欢的动态
        for _item in mongo.unlike.find({"userId": user_id_obj, "postId": {"$exists": 1}}, ['postId']).limit(50):
            unlike_post.add(object_to_int(_item['postId']))

        # 点赞但未关注的关系
        for _item in mongo.post.find({'like.userId': user_id_obj}, ['userId']).limit(50):
            like_user.add(object_to_int(_item['userId']))

    like_not_follow = like_user - follow
    redis.hmset(cache_label_shield_key, {"topics": str(topics),
                                         "report": str(shield),
                                         "fan": str(fans),
                                         "follow": str(follow),
                                         "black": str(black),
                                         "unlike": str(unlike),
                                         "unlikepost": str(unlike_post),
                                         "lnotf": str(like_not_follow)})

    if conf.SVC_DOWNGRADE > 0:  # 1级及以上服务降级
        redis.expire(cache_label_shield_key, 3600)
    else:
        redis.expire(cache_label_shield_key, 12 * 3600)
```


## 长期优化

多redis实例