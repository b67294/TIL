
在本地利用Docker - Image 安装Mysql数据库，并通过容器启用。

![](../assets/Pasted%20image%2020261012111825.png)

通过Navicat来创建链接和查看库。
![](../assets/Pasted%20image%2020261012111938.png)
注意左侧的并不是“库”，而是链接。

---
目前接触了两种创建数据库的方式，一种是用 Docker容器创建Mysql服务，另一种是在python中使用sqlite创建。

---
sqlite 创建的数据库相对简单很多，一般是`.db`文件。
Mysql 创建的数据库更复杂，是一个独立的服务。
二者创建的数据库都可以用Navicat进行连接。

---
Mysql创建的数据库会对外暴露端口，是一个独立服务；
而sqlite是本地文件，利用本地文件直接链接，没有对外暴露的端口。

---
用简单的python语句创建、连接一个数据库，并利用Base实现ORM：
```PYTHON
from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.orm import declarative_base, sessionmaker

# 1. 创建一个 SQLite 数据库文件（名字叫 test.db，代码运行后会自动在当前目录生成）
# 相当于连上了数据库
engine = create_engine("sqlite:///test.db", echo=False)# /test.db 表示相对于代码的相对路径 也可以写为绝对路径

# 2. 定义 Base（这就是我们前面说的“户口本/基类”）
# 所有的 ORM 模型类，都必须继承自这个 Base
Base = declarative_base()


# 3. 定义一个任务类（ORM 模型）
# 它继承了 Base，所以 SQLAlchemy 知道它是一个数据库表模型
class TodoTask(Base):
    # 指定它在数据库里对应的表名叫 todo_tasks
    __tablename__ = "todo_tasks"

    # 定义表的字段
    id = Column(Integer, primary_key=True, autoincrement=True)  # 主键 id
    title = Column(String(100), nullable=False)  # 任务标题
    status = Column(String(20), default="pending")  # 任务状态


# 4. 让 Base 根据上面的类定义，自动在数据库里创建对应的表
# 这一步就是把 Python 代码转换成 SQL 的 CREATE TABLE 语句
Base.metadata.create_all(engine)

# ==================== 下面测试如何像操作普通对象一样操作数据库 ====================

# 创建一个会话（Session），用来和数据库交互
SessionLocal = sessionmaker(bind=engine)
db = SessionLocal()

print("--- 步骤 1：创建一个任务对象并保存到数据库 ---")
# 实例化一个类，就像操作普通 Python 对象一样
new_task = TodoTask(title="学习 SQLAlchemy ORM", status="in_progress")

# 把对象加入会话，并提交（相当于执行 INSERT 语句）
db.add(new_task)
db.commit()
print(f"成功保存！新任务的 ID 是: {new_task.id}")

print("\n--- 步骤 2：从数据库里把任务查出来 ---")
# 就像写 Python 查询一样，把状态是 in_progress 的任务查出来
task_from_db = db.query(TodoTask).filter_by(status="in_progress").first()
print(f"查询结果 -> 标题: {task_from_db.title}, 状态: {task_from_db.status}")

# 关闭会话
db.close()
```

关于sqlalchemy：
![](../assets/Pasted%20image%2020261012114444.png)