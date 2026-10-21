## 核心思路
并发控制的核心是把”提交“和”执行“解耦。

依旧拿 客户端-服务端 的建模举例。最简单基础、没有任何防备的情况就是”客户端提交一个请求，服务端就试图执行一个请求。“

![](../assets/Pasted%20image%2020261021134902.png)
一个请求在数据库里一般表示为一行记录：

| job_id | user_id | state     |
| ------ | ------- | --------- |
| 1      | 1       | submitted |
这就是提交到平台的一个任务了。在我们的前端可能展示为”已提交“、

但，客户端可能一次性提交非常多的请求。
![](../assets/Pasted%20image%2020261021135037.png)

| job_id | user_id | state     |
| ------ | ------- | --------- |
| 1      | 1       | submitted |
| 2      | 1       | submitted |
| 3      | 1       | submitted |
| ...    | ...     | ...       |
| 100    | 1       | submitted |

一旦服务端接不住那么多的请求（因为每个请求都会占用一定的资源，例如CPU/GPU等）可能服务端就被卡死，服务崩溃，死机了。但此时，我们的客户端还什么都不知道，只剩了一堆显示”已提交“的任务。

这时候我们想给提交加一个数量上的限制，比如说，每次只运行4个任务在运行。
这里我们就可以加入一个Worker。

Worker 的核心逻辑非常简单：死循环 $\to$ 查一下现在跑了几个 $\to$ 没满 4 个就去拉一个新任务 $\to$ 丢给平台执行。

```python
import time
from sqlalchemy import select, func

def run_worker_loop(db_session_factory):
    MAX_SLOTS = 4  # 全局并发上限

    while True:
        with db_session_factory() as db:
            
            # 第一步：查一下现在正在外部执行的任务有几个
            # 状态为 "submitted" 的任务代表已经占用了名额
            active_count = db.scalar(
                select(func.count(Job.id))
                .where(Job.state == "submitted")
            ) or 0
            
            # 第二步：如果名额已经满 4 个，本轮不处理新任务
            if active_count >= MAX_SLOTS:
                db.commit()
                time.sleep(2)  # 休息一会儿再进入下一轮检查
                continue
                
            # 第三步：名额没满，去数据库里捞一个最早排队（queued）的任务
            job_to_run = db.scalar(
                select(Job)
                .where(Job.state == "queued")
                .order_by(Job.created_at.asc())  # 按创建时间排序，先到先得
                .limit(1)
            )
            
            # 如果队列里空了，没有等待中的任务，就跳过
            if not job_to_run:
                db.commit()
                time.sleep(2)
                continue
                
            # 第四步：把这个任务的状态从 "queued" 改成 "submitted"
            # 这代表它正式取得了名额，离开等待队列
            job_to_run.state = "submitted"
            db.commit()  # 提交事务，把修改写入数据库
            
            # 第五步：丢给外部平台执行
            # 此时后端发起 HTTP 请求，获取 request_id，平台开始在后台处理
            execute_on_external_platform(job_to_run)
            
        # 循环节奏控制
        time.sleep(1)
```

BTW:可以看到目前接触的大多架构是 WebAPI + 任务队列 + Worker。
>Web API + 任务队列 + Worker（生产者-消费者模型）是后端工程里处理异步、耗时、重资源任务（如 AIGC、音视频转码、大批量数据报表）时最经典、最主流的架构范式。

关于Worker相关的可以看 [Worker](2026-10-21-Worker.md)



