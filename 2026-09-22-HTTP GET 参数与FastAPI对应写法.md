

对`get`请求的理解一直蛮浅显的，大概理解为对一个URL发送获取信息的请求。通俗点来说，"向一个地址索要一些东西"。所以它大概是一个只读的请求。

扩展理解一下，`get`作为一个获取信息的请求，也是可以有用于筛选信息的参数的。

```Python
import requests

res = requests.get('http://127.0.0.1:8000/users') #没有参数的get请求

res = requests.get('http://127.0.0.1:8000/users/42')#参数为42的get请求 路径参数

res = requests.get('http://127.0.0.1:8000/search?keyword=python&page=2') #查询参数 keyword page
```

对应在FastAPI中：

```Python
from fastapi import FastAPI

app = FastAPI()


# 1. 路径参数 (Path Parameter)
# URL 形如: http://127.0.0.1:8000/users/42
# 此时 42 直接嵌入在 URL 路径中
@app.get("/users/{user_id}")
async def get_user_by_id(user_id: int):
    return {"message": f"正在查询 ID 为 {user_id} 的用户"}


# 2. 查询参数 (Query Parameter)
# URL 形如: http://127.0.0.1:8000/search?keyword=python&page=1
# 只要函数参数没有写在 @app.get() 的路径大括号里，FastAPI 就会自动把它当作 ? 后的查询参数解析
@app.get("/search")
async def search_items(keyword: str, page: int = 1):  # page 设置了默认值 1
    return {"message": f"搜索关键词: {keyword}", "当前页码": page}
```