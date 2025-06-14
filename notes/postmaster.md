# postmaster目录

## postmaster.c/PostmasterMain 入口函数
- 创建内存上下文 AllocSetContextCreate
- 信号处理 
- 解析命令行参数
- 创建锁文件 CreateDataDirLockFile
- 初始化SSL库
- 处理预加载库 process_shared_preload_libraries
- 创建监听套接字
- 初始化共享内存和信号量 reset_shared
- 启动syslogger子进程【重要】SysLogger_Start //负责日志收集和管理，将日志写入文件或发送到系统日志服务
- 初始化统计收集子系统【重要】 pgstat_init();
- 初始化自动清理子系统 【重要】 autovac_init();
- 启动数据库 【重要】 StartupDataBase(); //负责数据库的启动和恢复
- 处理客户端请求【重要】 status = ServerLoop();

## postmaster.c/ServerLoop
- StartBackgroundWriter //负责后台写入，定期将脏页刷新到磁盘
- StartWalWriter() //负责 WAL 的写入，确保事务日志的持久化。
- StartAutoVacLauncher() //负责自动清理（Vacuum）操作，回收死元组并优化数据库性能。
- pgarch_start() //负责 WAL 归档，将旧的 WAL 文件归档到指定位置。
- pgstat_start() //负责统计信息的收集和报告。

## autovacuum.c


## bgwriter.c
