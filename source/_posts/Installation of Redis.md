---
title: windows上安装Redis
date: 2024-12-21 10:10:45
tags: Redis,windows服务
---

## 一、**下载**：
[https://github.com/tporadowski/redis](https://github.com/tporadowski/redis)（官方不提供windows版）



## 二、**添加服务的命令**：
> **配置文件里设置密码**：requirepass 123456
> `redis.windows-service.conf`表示所使用的配置文件.
```bash
redis-server --service-install redis.windows-service.conf --loglevel verbose
```

## 启动停止和卸载服务

**启动服务**：`redis-server --service-start`
**停止服务**：`redis-server --service-stop`
**卸载服务**：`redis-server --service-uninstall`



## 三、**连接**：
`redis-cli.exe -h 127.0.0.1 -p 6379 -a 123456`
**或者**：
```
C:\Redis>redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```



## 四、多个Redis服务（可选，以上的步骤已经完成了Redis的安装）
> 首先修改配置文件，确保**端口不能冲突**。
> 还有就是**服务名**不能冲突！！

### **安装 Redis 服务**
```bash
redis-server --service-install redis.windows.conf --loglevel verbose --service-name Redis_2
```
### **启动 Redis 服务**
安装完毕后，你可以启动 Redis 服务。使用以下命令启动 Redis 服务：

```bash
redis-server --service-start --service-name Redis_2
```

###  **停止 Redis 服务**：

```bash
redis-server --service-stop --service-name Redis_2
```

### **卸载 Redis 服务**：
如果你不再需要 Redis 服务，可以通过以下命令卸载它：

```bash
redis-server --service-uninstall --service-name Redis_2
```



## 五、与windows服务相关的一些命令（使用cmd）

### `sc qc`  查看服务配置（Query Config）
```bash
sc qc Redis_2
```
![sc qc](https://i-blog.csdnimg.cn/direct/ecaa3362a3ff4c0eadd6d5c8d945b3d9.png)


### `sc query`  查询服务状态（Query State）
```bash
sc query Redis_2
```
![sc query](https://i-blog.csdnimg.cn/direct/df18161f61624165b667e0ad53db5c52.png)


### 使用 netstat 命令查看端口 (也能用于：查找特定 PID 对应的端口)
> 你可以使用 netstat 命令来列出系统上所有的活动连接和它们的端口。要查找特定服务的端口，可以过滤输出。

例如，要查看 Redis 默认的端口（6379）占用情况：
```bash
netstat -ano | findstr 6379
```
- -a：显示所有连接和监听端口。
- -n：以数字形式显示地址和端口号。
- -o：显示每个连接所属的进程 ID（PID）。

![netstat](https://i-blog.csdnimg.cn/direct/d5f3d6c60ceb4d9e8f7897faad085280.png)
> 依次为："`协议，本地地址，外部地址，状态，PID`",这条记录表示 Redis 正在本地监听端口 6379，等待客户端（本地或远程）连接。

```bash
netstat -ano | findstr 17160
```
![netstat](https://i-blog.csdnimg.cn/direct/a18c9361243646a69f5dd68720081f1f.png)


### 使用 tasklist 查看 PID 对应的进程名称
> 一旦你知道了进程 ID（PID），你可以使用 tasklist 命令查看该 PID 对应的进程名称。

```bash
tasklist /FI "PID eq 17160"
```
![tasklist](https://i-blog.csdnimg.cn/direct/3c319bc3cd8d4de8b182affe18087e29.png)


### 使用 tasklist 查看`进程名称`对应的 PID
```bash
tasklist /FI "IMAGENAME eq redis-server.exe"
```
![tasklist](https://i-blog.csdnimg.cn/direct/907dce9e9e0448f7bc89ef6072de5544.png)


----------


## 使用 PowerShell

### 由`服务名`输出`服务的进程 ID（PID）`(good good,使用真正的服务名而不是进程名)
`Get-WmiObject Win32_Service | Where-Object { $_.DisplayName -eq "Redis" } | Select-Object Name, ProcessId`
> 补充：`Get-WmiObject -Class Win32_Service | Where-Object { $_.ProcessId -eq 17160 }`



### 查找 PID 所占用的端口
`Get-NetTCPConnection -OwningProcess 17160`
![powershell](https://i-blog.csdnimg.cn/direct/569e9c106e82477d8c22042079507ee0.png)



## 一条命令（花里胡哨）

### 将 netstat 和 tasklist 输出结合起来，通过以下命令找出`某个端口对应的进程名称`：
`netstat -ano | findstr "6379" | for /f "tokens=5" %a in ('more') do tasklist /FI "PID eq %a"`
![cmd](https://i-blog.csdnimg.cn/direct/1caf9992ed9d4a46a9bd4a2edd7798e1.png)


### 使用powershell
`Get-NetTCPConnection -LocalPort 6379 | ForEach-Object { (Get-WmiObject Win32_Process -Filter "ProcessId = $($_.OwningProcess)").Name }`
![powershell](https://i-blog.csdnimg.cn/direct/ae17222af4454479b14158519e8ebf2e.png)

```powershell
Get-NetTCPConnection -LocalPort 6379 | ForEach-Object { (Get-WmiObject Win32_Process -Filter "ProcessId = $($_.OwningProcess)").Name }


Get-NetTCPConnection -LocalPort 6379 | ForEach-Object {
    $processId = $_.OwningProcess
    Get-WmiObject Win32_Process | Where-Object { $_.ProcessId -eq $processId }
}


Get-NetTCPConnection -LocalPort 6379 | ForEach-Object {
    $processId = $_.OwningProcess
    Get-WmiObject Win32_Service | Where-Object { $_.ProcessId -eq $processId }
}


Get-NetTCPConnection -LocalPort 6379 | ForEach-Object {
    Get-Process -Id $_.OwningProcess
}
```
