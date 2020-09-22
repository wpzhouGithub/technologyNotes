备选： sanic、tornado、flask

flask暂时不支持异步，被淘汰



sanic 、tornado对比

|                  | tornado                   | sanic  |      |
| ---------------- | ------------------------- | ------ | ---- |
| 支持的python版本 | 3.X                       | 3.6+   |      |
| 异步的底层实现   | asyncio（可以使用uvloop） | uvloop |      |
| rps              | 较差                      | 较优   |      |
| 文档             | 完善                      | 较少   |      |
| 更新             | 差不多                    | 差不多 |      |
| uwsgi            | 支持                      | 不支持 |      |



综上，基于sanic目前还太新、文档还不完善且tornado可以使用uvloop事件循环，**决定采用tornado6.0； 将asyncio的事件循环替换为uvloop**

