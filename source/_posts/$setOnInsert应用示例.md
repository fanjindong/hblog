未使用`$setOnInsert`之前
```python
	mongo_update = {"status": 1}
    if topic_id:
        await mongo.topic.update_one({"_id": to_object(topic_id)}, {"$set": mongo_update})
    else:
        topic_id = await Generate.generate_id(redis, config.ID_TOPIC)
        mongo_update.update({"_id": to_object(topic_id)})
        mongo_update.update({'createdAt': int(time.time())})
        mongo_update.update({'viewCount': 0})
        mongo_update.update({'postCount': 0})
        try:
            await mongo.topic.insert_one(mongo_update)
        except DuplicateKeyError:
            return json({"code": 15001, "message": "DuplicateKeyError"})
```

使用`$setOnInsert`之后
```python
	topic_id = topic_id or await Generate.generate_id(redis, config.ID_TOPIC)
	mongo_update = {"status": 1}

	await mongo.topic.update_one(
		{"_id": to_object(topic_id)},
		{"$set": mongo_update, "$setOnInsert": {'createdAt': int(time.time()), 'viewCount': 0, 'postCount': 0}},
		upsert=True
	)
```