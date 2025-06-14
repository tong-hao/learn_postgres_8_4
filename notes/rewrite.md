
一、主要文件结构
```
rewrite/
├── rewriteDefine.c    # 规则定义（CREATE RULE）
├── rewriteHandler.c   # 查询重写核心逻辑
├── rewriteManip.c      # 查询树操作工具
├── rewriteRemove.c     # 规则删除（DROP RULE）
└── rewriteSupport.c    # 规则支持函数
```


二、核心代码解析
1. 规则定义 (rewriteDefine.c)

```c
// 创建规则入口
void DefineRule(RuleStmt *stmt, const char *queryString) {
    transformRuleStmt();  // 语法解析
    DefineQueryRewrite(); // 写入 pg_rewrite 系统表
}

// 规则持久化存储
static Oid InsertRule(...) {
    // 将规则内容序列化存储
    char *evqual = nodeToString(event_qual);
    char *actiontree = nodeToString((Node *) action);
    // 写入 pg_rewrite 系统表
    CatalogTupleInsert(pg_rewrite_desc, tup);
}
```

2. 查询重写引擎 (rewriteHandler.c)

```c
// 主入口函数
List *QueryRewrite(Query *parsetree) {
    // 阶段1：应用非 SELECT 规则
    querylist = RewriteQuery(parsetree, NIL);
    
    // 阶段2：应用 RIR 规则（行级安全）
    query = fireRIRrules(query, NIL);
    
    // 处理视图替换逻辑
    if (rte->rtekind == RTE_SUBQUERY) {
        ereport(ERROR,...); // 抛出视图操作错误
    }
}

// 规则触发核心
static List *fireRules(...) {
    // 处理 INSTEAD 规则
    if (rule->isInstead) {
        *instead_flag = true;
        // 生成替换查询
        rule_action = rewriteRuleAction(...);
    }
    
    // 处理带条件的规则
    if (event_qual != NULL) {
        *qual_product = AddInvertedQual(...);
    }
}
```

3. 查询树操作 (rewriteManip.c)

```c
// 调整变量引用（处理规则展开后的变量冲突）
void OffsetVarNodes(Node *node, int offset, int sublevels_up) {
    // 递归遍历查询树
    query_tree_walker(...);
}

// 处理 INSERT 的 DEFAULT 值
void rewriteTargetList(Query *parsetree, Relation target_relation) {
    // 将 DEFAULT 替换为实际默认值
    new_expr = build_column_default(...);
}
```

三、核心数据结构

```c
// 规则内存表示 (prs2lock.h)
struct RewriteRule {
    Oid         ruleId;         // 规则 OID
    CmdType     event;          // 触发事件类型 (SELECT/INSERT/等)
    Node       *qual;           // 规则条件
    List       *actions;        // 规则动作查询列表
    bool        isInstead;      // 是否为 INSTEAD 规则
};

// 查询重写上下文 (rewriteHandler.c)
struct rewrite_event {
    Oid         relation;       // 相关表 OID
    CmdType     event;          // 当前处理的事件类型
};
```

四、关键处理流程
1. 语法解析 → 生成 RuleStmt
2. 规则持久化 → 写入 pg_rewrite
3. 查询重写阶段：
   a. 应用非 SELECT 规则（UPDATE/INSERT/DELETE）
   b. 应用 RIR 规则（行级安全策略）
4. 变量调整 → 解决规则展开后的变量冲突
5. 条件处理 → 合并原始查询与规则条件

五、典型场景示例
视图处理 (rewriteHandler.c:1900+)

```c
// 当尝试直接操作视图时
case CMD_UPDATE:
    ereport(ERROR, 
        errmsg("cannot update a view"),
        errhint("You need an unconditional ON UPDATE DO INSTEAD rule."));
```
该模块通过复杂的查询树操作实现了 PostgreSQL 强大的规则系统，是视图、行级安全等功能的底层支撑。
