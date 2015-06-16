---
author: hzmangel
date: "2015-06-16T23:40:10+08:00"

categories:
- Happy coding

tags:
- Programming
- TIL
- Python

title: |
  Set timezone in Python
---

今天在写一个脚本的时候，发现使用`datetime.datetime.now()`输出的是UTC时间，而同样的命令在ipython中输入的就是本地的时间。找了好久才找到不用`pytz`的解决方案：

```python
import os
import datetime

print(datetime.datetime.now())
os.environ['TZ'] = 'Asia/Shanghai'
print(datetime.datetime.now())
```

`TZ`的环境变量让datetime输出了指定时区的时间。

