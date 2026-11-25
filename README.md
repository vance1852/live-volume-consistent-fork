# 抢险地图工作卷的在线快照与分支

存储联调使用合成设备、时间轴和原始字节。设备记录由各客户端及介质的输入组成，每份记录可独立使用。

## 业务协议

各卷 `block_size` 一致。位置按从零开始的块号，擦除一块在该分支中读作零。卷组的同一 `checkpoint` 是制图端共识的业务切面，设备不会替业务端决定何时所有卷均达到它。写请求含 `session`、`seq`、`checkpoint` 和 `file`。设备在 `PUT /volumes/{volume}/blocks/{block}` 接收块正文，响应 `queued` 说明仅进缓存。
`flush` 回报的 `through` 是对应会话中最后已持久的序号，之前的写入才具持久证明。设备失电后另起 `generation`，旧会话的回报仍可迟到。请求的快照期限用合成时间轴，`until` 为右端点。页面和索引的字节都是设备交互原件，不用解析地理数据。

## 本地设备资料

运行环境为 Python 3.11.2，使用标准库。设备资料端默认仅在 127.0.0.1:8090 运行，使用 --port 可另设端口。

~~~sh
python3 device_fixture.py fixtures/asymmetric-checkpoint
~~~

GET /fixture/events 返回设备参数、字节目录和按到达顺序记录的输入。同一时刻按文件先后排列。GET /fixture/objects/文件名 取原始字节。GET /fixture/journal 查看资料端已收到的设备请求，每个请求记录实际正文长度和摘要。

events.json 中的 packets 是可重放的设备响应。资料端按实际 HTTP 方法、路径和完整原始请求正文匹配第一条尚未使用的记录。没有匹配时返回 404。lag_ms 是回包延迟，drop 代表设备收到请求后直接断开连接，接收方需要以其他报告确认结果。没有 HTTP 交互的设备和客户端动作直接取 events。重新启动资料端会回到记录开头。

在项目根目录运行 python3 -m unittest -v 检查事件、对应字节、设备响应及接收日志。
