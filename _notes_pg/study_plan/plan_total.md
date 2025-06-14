1. 启动入口（main）
文件：src/backend/main/main.c
学习重点：理解PostgreSQL进程的启动逻辑

main() // 主入口，根据参数判断启动模式（postmaster/postgres）
PostmasterMain() // 主服务器进程入口
PostgresMain()   // 单个后端进程入口


2. 进程管理（postmaster）
目录：src/backend/postmaster
关键文件：
postmaster.c // 主服务器进程管理（连接监听、子进程派生）
autovacuum.c // 自动清理进程实现
bgwriter.c    // 后台写入进程

3. 查询分发（tcop）
目录：src/backend/tcop
关键文件：
postgres.c  // 查询请求主处理逻辑
fastpath.c  // 快速路径函数处理
utility.c   // 工具类命令处理


4. 查询解析（parser）
目录：src/backend/parser
关键文件：
scan.l      // 词法分析器（将SQL拆分为token）
gram.y      // 语法分析器（生成解析树）
analyze.c   // 语义分析

5. 查询优化（optimizer）
目录：src/backend/optimizer
学习顺序：
path/ // 生成访问路径
plan/ // 生成执行计划
geqo/ // 遗传算法优化器
prep/ // 预处理逻辑


6. 查询执行（executor）
目录：src/backend/executor
关键文件：
nodeHash.c      // 哈希连接实现
nodeSort.c      // 排序实现
nodeAgg.c       // 聚合函数处理
spi.c           // 服务器端编程接口


7. 存储管理（storage）
目录：src/backend/storage
核心子目录：
buffer/    // 共享缓冲区管理
ipc/       // 进程间通信
lmgr/      // 锁管理
page/      // 数据页管理


8. 系统表（catalog）
目录：src/backend/catalog
学习顺序建议：
heap.c // 表存储管理
index.c // 索引管理
namespace.c// 命名空间
pg_type.c // 类型系统实现


9. 实用命令（commands）
目录：src/backend/commands
典型示例：
tablecmds.c // CREATE/ALTER TABLE实现
vacuum.c    // VACUUM命令实现
copy.c      // COPY命令处理

-----

## 启动入口 main.c

```
main()
  PostgresMain()   // 单个后端进程入口
    exec_simple_query
  PostmasterMain() // 主服务器进程入口
    SysLogger_Start
    pgstat_init 统计初始化
    autovac_init 自动回收初始化
    ServerLoop
      BackendStartup
      SysLogger_Start
      StartBackgroundWriter
      StartWalWriter
      StartAutoVacLauncher
      pgarch_start
      pgstat_start
```

## 进程管理
### postmaster.c 主服务器进程管理
```
BackendStartup 主要负责创建新的后端服务进程来处理客户端连接请求
  fork_process
  BackendRun //child
    PostgresMain
```

```
exec_simple_query
  start_xact_command
  pg_parse_query  //解析出parsetree_list
  pg_analyze_and_rewrite //分析和重写查询树
  pg_plan_queries //生成查询计划
  CreatePortal // 执行
  PortalStart
  CreateDestReceiver
  PortalRun
```

### autovacuum.c 自动清理进程实现
```
StartAutoVacLauncher
  fork_process
  AutoVacLauncherMain
    for (;;)
      launch_worker
```

### bgwriter.c 后台写入进程
```
BackgroundWriterMain
  for (;;)
    BgBufferSync()
      SyncOneBuffer
```

## 查询分发（tcop）

### postgres.c  // 查询请求主处理逻辑

### fastpath.c  // 快速路径函数处理

### utility.c   // 工具类命令处理






























































































































