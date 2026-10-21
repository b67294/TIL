
数据库的事务是一种保护机制。

```
-- 1. 开始事务（相当于开启安全箱）
BEGIN TRANSACTION;

-- 2. 执行插入操作（此时数据只在临时缓存里，还没真正写入磁盘）
INSERT INTO todo_tasks (title, status) VALUES ('学习 SQL 事务', 'pending');

-- 3. 检查一下：在这行代码执行前或后，如果中途发现写错了，可以随时回滚：
-- ROLLBACK; 

-- 4. 确认无误，提交事务（正式写入磁盘，永久生效）
COMMIT;
```

在事务之间的操作不会被立即执行。而是在提交后再执行。
一般对于写操作使用，读操作不使用。
如果不利用事务，则会立即执行语句。

AI更加全面的解释：
在 SQL 中，当你执行数据的修改（如 `INSERT`、`UPDATE`、`DELETE`）时，数据库不会立刻把结果永久写入文件，而是开启了一个**事务**。

- 在这个事务没有结束（没有执行 `COMMIT`）之前，所有的改动都只是临时保存在内存的缓冲区里。
    
- 只有当收到 **`COMMIT`** 命令时，数据库才会把这些改动永久写入磁盘。
    
- 如果中途出错，执行 **`ROLLBACK`**，数据库就会把内存里暂存的这部分改动全部丢弃，恢复到操作前原样。


这是事务在SQL中的样子，而在Python中，我们使用Session来做到事务。

```PYTHON
# 创建一个会话（Session），用来和数据库交互
SessionLocal = sessionmaker(bind=engine)
db = SessionLocal()
```

这里的 `db = SessionLocal()`就等于在底层运行了SQL语句中的`BEGIN TRANSCATION`

每次运行到`db = SessionLocal()`的时候，都是完全独立、互不干扰的全新`BEGIN TRANSACTION`,这样就做到了SQL语句的事务隔离。

一个事务的完整代码：
```python
SessionLocal = sessionmaker(bind=engine)
db = SessionLocal()

print("--- 步骤 1：创建一个任务对象并保存到数据库 ---")
# 实例化一个类，就像操作普通 Python 对象一样
new_task = TodoTask(title="学习 SQLAlchemy ORM", status="in_progress")

# 把对象加入会话，并提交（相当于执行 INSERT 语句）
db.add(new_task)
db.commit()
```

可以想象为，`BEGIN TRANSACTION`与`COMMIT`之间的sql语句是一个原子。不同的事务是不同的原子。在python层面，我们利用session创造事务，同样的实现sql语句之间的原子隔离。