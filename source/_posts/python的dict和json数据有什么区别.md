
---
title: python的dict和json数据有什么区别
---

工作中和其他语言的工程师交流，合作与联调中经常会涉及到数据的传输，这个数据的传输通常为json字符串，这个json格式数据和python自身的dict数据对象非常像，所以很自然的会思考这两者究竟区别在哪里？

**首先结论:两者不一样**

### 区别

1. Python的`dict`是一种数据结构，`JSON`是一种数据格式。
1. `dict`的`key`可以是任意可hash对象，`json`只能是字符串。`{(1,2):1}` 在python里是合法的,因为`tuple`是`hashable type`; `{[1,2]:1}` 在python里`TypeError: unhashable "list"`
1. 形式上有些相像，但`json`是纯文本的，无法直接操作。
1. `dict`字符串用单引号，`json`强制规定双引号。
1. `dict`里可以嵌套`tuple`, `json`里只有`array`。 `json.dumps({1:2})` 的结果是 `{"1":2}`, `json.dumps((1,2))` 的结果是`[1,2]`
1. `json: true|false|null`; `dict:True|False|None`


### 联系
`dict` 存在于内存中，可以被序列化成 `json` 格式的数据（string），之后这些数据就可以传输或者存储了。

### 总结
`JSON` 是一种数据传输格式。

也就是说，这些字符串以 `JSON` 这样的格式来传输，至于你怎么 `parse` 这些信息，甚至是是否 `parse`, 是否储存，都不是 `JSON` 的事情。

用 Python 举个例子: 某段程序可以把字符串 `"{A:1, B:2}"` `parse` 成 一对 `tuple: ( ("A", 1), ("B", 2) ) `而不是 `dictionary: {"A": 1, "B": 2}`.

所以 `JSON` 它能被解析成 Python 的 `Dictionary` 或者其他形式，但解析成什么内容是和 `JSON` 这种格式无关的。

Python 的 `Dictionary` 则是 Python 对 Hash Table 的实现，一套从存储到提取都封装好了的方案。

## 参考
[讨论: 你觉得python的字典和json差不多吗？](https://www.zhihu.com/question/21097237)