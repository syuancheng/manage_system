# manage_system
Ref： https://qx3hit421k.feishu.cn/docx/L4hndPgSgopMWwxY2XTcWCuynQY

### 功能介绍
entry task实现了一个用户管理系统，它包括四个部分，分别是Client、Http Server、Tcp Server和后台存储（MySQL、Redis）。
- 用户可以通过在Web页面输入username和password登录
- Http Server会处理用户的输入并调用Tcp Server提供的RPC调用
- Tcp Server负责校验用户身份并返回RPC调用的结果。如果登录成功，页面将展示用户的相关信息，否则页面展示相关错误提示。成功登录后，用户可以修改用户信息中的picture和nickname选项。
- SQL参数化查询，可以防止sql注入

#### 总体结构图
#### 具体设计
##### 用户信息
- username（不可更改）
- nickname
- profile picture（图片）

Client
- raiseReqWithFile(client, url, file, method)

使用file中记录的用户信息，向Http Server发起名为method的请求，可用的方法包括：增加新用户、拉取用户信息、修改用户信息。

Http Server
- 用户登入
- 处理用户的登入，接收username、password参数
- 新增用户
- 处理新增用户的请求，接收username、password、nickname（可选）、filepath（可选）参数
- 更新用户参数
- 处理更新用户Profile--Nickname的请求，接收username、password、nickname参数；
- 处理更新用户Profile--Filepath的请求接收username、password、filepath参数

Tcp Server
- 用户登入
- 检查username和password是否正确，如果正确，返回一个token值
- 用户拉取Profile
- 首先在redis中查询是否存在token，如果token校验成功，先尝试从redis返回数据，再尝试从MySQL返回数据
- 用户更新Profile
- 首先校验token，接着使redis中缓存的数据失效，再将新值写入MySQL

Redis
key	value
username	token
username	{valid, nickname, filepath}

MySQL
column	type	null
id	int	no
username	varchar(255)	no
password	varchar(255)	no
nickname	varchar(255)	yes
Filepath	varchar(255)	yes

部署、运维文档
- 运行TCP Server

go run tcp_server.go

- 运行HTTP Server

go run http_server.go

- 运行Client

go run client.go

- 运行Redis Server

redis-server

性能测试
- 测试指标：HTTP API QPS
- 指标要求： 
  - 并发=200时，固定用户QPS>3000，随机用户QPS>1000
  - 并发=2000时，固定用户QPS>1500，随机用户QPS>800
