# 系统架构

## 总体结构

```mermaid
flowchart TB
    subgraph UX["桌面交互层"]
        CHAT["助理对话"]
        ANALYSIS["智能分析"]
        ACCOUNTS["交易账户"]
        ALERTS["重要提醒"]
        PERF["投资绩效"]
    end

    subgraph CORE["AUREVAN 核心"]
        ROUTER["意图识别与能力路由"]
        CONTEXT["统一上下文构建"]
        WATCH["持续盯守与状态机"]
        SCHED["报告与定时任务"]
        AUTH["授权边界"]
    end

    subgraph DEPTS["专业分析部门"]
        FOREX["外汇实盘分析"]
        MT["MT4 / MT5 Bridge"]
        MARGIN["外汇保证金分析（建设中）"]
    end

    subgraph MODEL["分析与数据"]
        LOCAL["本地精确查询"]
        LLM["大模型综合分析"]
        DB["SQLite 本地数据"]
    end

    UX <--> CORE
    ROUTER --> CONTEXT
    CONTEXT <--> FOREX
    CONTEXT <--> MT
    MARGIN -. "完成后按同一契约接入" .-> CONTEXT
    CONTEXT <--> LOCAL
    CONTEXT <--> LLM
    WATCH <--> DEPTS
    SCHED --> WATCH
    AUTH --> SCHED
    LOCAL <--> DB
    WATCH <--> DB
```

## 核心原则

### 统一上下文

所有对话入口必须使用同一套上下文构建器。当前工具、账户、品种、页面和最近有效快照作为结构化上下文传入，避免模型仅依赖用户短句猜测对象。

### 数据与分析分离

持仓数量、账户余额、交易流水和工具状态由本地精确查询返回；原因、影响、风险和建议再交给模型分析。模型不能修改工具事实，也不能把缺失字段推断为零。

### 状态驱动提醒

提醒以“事项”为核心而不是以“每次轮询”为核心。同一事项更新状态和时间，不重复创建卡片；只有跨越风险阈值、方向变化、工具恢复或数据重新有效等实质变化才重新触达。

### 专业部门隔离

专业工具随应用发布，但继续在独立进程与模块中运行。AUREVAN 通过适配器和 MCP 获取标准化结果，不把工具业务代码复制到对话核心。

### 本地数据边界

数据库、备份、工具日志、模型密钥和交易数据留在用户本机。对模型只发送完成当前问题所需的最小摘要，并在外发前裁剪或脱敏敏感字段。
