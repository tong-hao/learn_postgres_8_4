# catalog目录
src/backend/catalog

## catalog.c 
IsSystemRelation()     // 判断是否为系统表
IsToastRelation()      // 判断是否为TOAST表
GetNewOid()            // 生成新OID

## heap.c 
heap_create()          // 创建新系统表
heap_drop_with_catalog() // 删除系统表

## index.c 索引管理
index_create()         // 创建系统表索引
index_drop()           // 删除索引

## aclchk.c 权限检查
pg_class_ownercheck()  // 表所有者检查
pg_database_ownercheck() // 数据库权限检查

## pg_xxx.c 每个文件对应一个系统表的操作 
pg_type.c       // pg_type 系统表操作
pg_proc.c       // pg_proc（函数）系统表操作
pg_class.c      // pg_class（表/索引）系统表操作

## toasting.c TOAST表管理
create_toast_table()  // 创建TOAST表

## dependency.c 依赖管理
deleteOneObject()       // 级联删除依赖对象
recordDependencyOn()    // 记录对象依赖

## namespace.c
RangeVarGetRelid()      // 通过名称获取OID
QualifiedNameGetCreationNamespace() // 获取创建命名空间

## 头文件
src/include/catalog
pg_class.h     // 系统表 pg_class 的列定义
pg_type.h      // 系统表 pg_type 的结构
catalog.h      // 系统目录公共API

## 系统表初始化
通过 genbki.sh 脚本生成 postgres.bki 文件

## 流程
1.  通过 initdb 使用 postgres.bki 创建初始系统表
2.  创建表
```
heap_create()  // 在 pg_class 插入记录
index_create() // 在 pg_index 插入记录
```

