---
name: AI Agent 提示词工程
description: AI Agent 的 Prompt 设计、LangGraph 架构、工具链配置最佳实践。当需要设计 Agent 系统、编写高质量 Prompt、配置思维链流程时使用。适用于涉及 AI Agent 的项目。
---

# AI Agent 提示词工程

本规范适用于基于 **LangChain / LangGraph** 的 AI Agent 开发。

---

## Agent 设计流程

1. **明确 Agent 角色与能力边界**
2. **设计工具链（Tools）**
3. **编写 System Prompt**
4. **构建思维链流程（LangGraph）**
5. **测试与迭代**

---

## 1. 明确 Agent 角色与能力边界

### 角色定位

**示例**：
- ❌ 模糊：\"一个智能助手\"
- ✅ 清晰：\"一个专注于直播电商数据分析的 AI 助手，能够回答用户关于销售额、转化率、商品表现的问题\"

### 能力边界

**能做什么**：
- 查询数据库获取实时销售数据
- 生成数据分析报告
- 回答常见问题（FAQ）

**不能做什么**：
- 修改数据库（只读权限）
- 预测未来趋势（需要专门的模型）
- 处理敏感个人信息

---

## 2. 设计工具链（Tools）

### 什么是 Tool？

Tool 是 Agent 可以调用的函数，用于完成特定任务（如查询数据库、搜索网页、发送邮件）。

### Tool 设计原则

1. **单一职责**：一个 Tool 只做一件事
2. **清晰的输入输出**：使用 Pydantic 定义参数
3. **错误处理**：Tool 应优雅处理失败情况

### 示例：查询销售数据

```python
from langchain.tools import tool
from pydantic import BaseModel, Field

class QuerySalesInput(BaseModel):
    start_date: str = Field(description="开始日期，格式 YYYY-MM-DD")
    end_date: str = Field(description="结束日期，格式 YYYY-MM-DD")
    event_id: str | None = Field(default=None, description="可选：活动ID")

@tool(args_schema=QuerySalesInput)
def query_sales_data(start_date: str, end_date: str, event_id: str | None = None) -> dict:
    """
    查询指定日期范围内的销售数据。
    
    Args:
        start_date: 开始日期
        end_date: 结束日期
        event_id: 可选，指定活动ID
    
    Returns:
        包含销售数据的字典
    """
    # 实际查询逻辑
    return {
        "total_revenue": 12345.67,
        "order_count": 100,
        "average_order_value": 123.45
    }
```

### 常用 Tool 类型

| Tool 类型 | 示例 | 使用场景 |
|----------|------|---------|
| 数据库查询 | `query_sales_data` | 获取业务数据 |
| 搜索引擎 | `search_knowledge_base` | RAG（检索增强生成） |
| API 调用 | `get_weather` | 获取外部数据 |
| 文件操作 | `read_file`, `write_file` | 处理文档 |
| 计算工具 | `calculate_roi` | 复杂计算 |

---

## 3. 编写 System Prompt

### Prompt 结构

一个好的 System Prompt 应包含：

1. **角色定位**：你是谁？
2. **能力描述**：你能做什么？
3. **工作流程**：如何完成任务？
4. **输出格式**：返回什么格式的内容？
5. **约束条件**：不能做什么？

### 示例 Prompt

```python
SYSTEM_PROMPT = """
你是一个专业的直播电商数据分析助手。

# 你的能力
- 查询销售数据（销售额、订单量、转化率等）
- 生成数据分析报告
- 回答用户关于业务数据的问题

# 工作流程
1. 理解用户的问题
2. 如果需要数据，使用 `query_sales_data` 工具查询
3. 分析数据并生成清晰的回答
4. 如果无法回答，诚实告知用户

# 输出格式
- 使用简洁、专业的语言
- 数据以表格或列表形式呈现
- 关键数字加粗显示

# 约束条件
- 不要编造数据，只使用工具查询到的真实数据
- 不要泄露用户隐私信息
- 对于超出能力范围的问题，引导用户联系人工客服

# 示例对话
用户："上周的销售额是多少？"
你的思路：
1. 确定"上周"的日期范围（例如 2026-02-03 到 2026-02-09）
2. 调用 `query_sales_data(start_date="2026-02-03", end_date="2026-02-09")`
3. 返回结果："上周（2月3日-2月9日）的总销售额为 **12,345.67 元**，共 100 笔订单。"
"""
```

---

## 4. 构建思维链流程（LangGraph）

### 什么是 LangGraph？

LangGraph 是一个用于构建**有状态、多步骤** Agent 的框架。它通过**节点（Node）**和**边（Edge）**定义 Agent 的思维流程。

### 基本架构

```python
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 定义 Agent 状态
class AgentState(TypedDict):
    messages: list
    next_step: str

# 构建图
workflow = StateGraph(AgentState)

# 添加节点
workflow.add_node("router", router_node)
workflow.add_node("query_db", query_db_node)
workflow.add_node("generate_response", generate_response_node)

# 添加边（流程）
workflow.set_entry_point("router")
workflow.add_conditional_edges(
    "router",
    lambda state: state["next_step"],
    {
        "query": "query_db",
        "answer": "generate_response"
    }
)
workflow.add_edge("query_db", "generate_response")
workflow.add_edge("generate_response", END)

# 编译
agent = workflow.compile()
```

### 节点函数示例

#### Router Node（路由节点）

```python
def router_node(state: AgentState) -> AgentState:
    """
    判断用户问题类型，路由到不同的处理节点
    """
    user_message = state["messages"][-1]
    
    # 使用 LLM 判断意图
    prompt = f"用户问题: {user_message}\n这个问题需要查询数据库吗？回答'是'或'否'"
    response = llm.invoke(prompt)
    
    if "是" in response.content:
        state["next_step"] = "query"
    else:
        state["next_step"] = "answer"
    
    return state
```

#### Query DB Node（查询数据库节点）

```python
def query_db_node(state: AgentState) -> AgentState:
    """
    调用工具查询数据库
    """
    # 从用户消息中提取参数
    user_message = state["messages"][-1]
    
    # 使用 Tool
    result = query_sales_data(start_date="2026-02-01", end_date="2026-02-07")
    
    # 将结果添加到状态
    state["messages"].append({
        "role": "system",
        "content": f"查询结果: {result}"
    })
    
    return state
```

#### Generate Response Node（生成回复节点）

```python
def generate_response_node(state: AgentState) -> AgentState:
    """
    生成最终回复
    """
    messages = state["messages"]
    response = llm.invoke(messages)
    
    state["messages"].append({
        "role": "assistant",
        "content": response.content
    })
    
    return state
```

---

## 5. 多 Agent 协作

### 场景：复杂任务分工

**示例**：用户问"帮我分析上周的销售数据并生成报告"

**解决方案**：设计 3 个专门的 Agent：

1. **Data Analyst Agent**：负责查询和分析数据
2. **Report Writer Agent**：负责根据数据生成报告
3. **Coordinator Agent**：协调上述两个 Agent

```python
workflow = StateGraph(MultiAgentState)

workflow.add_node("coordinator", coordinator_agent)
workflow.add_node("analyst", data_analyst_agent)
workflow.add_node("writer", report_writer_agent)

workflow.set_entry_point("coordinator")
workflow.add_edge("coordinator", "analyst")
workflow.add_edge("analyst", "writer")
workflow.add_edge("writer", END)
```

---

## 6. Prompt 优化技巧

### 使用 Few-Shot Examples（少量示例）

```python
PROMPT = """
你是数据分析助手。

# 示例 1
用户："上周销售额是多少？"
你：调用 query_sales_data(start_date="2026-02-03", end_date="2026-02-09")
结果：总销售额 12,345.67 元

# 示例 2
用户："对比上月和本月的订单量"
你：
1. 调用 query_sales_data(start_date="2026-01-01", end_date="2026-01-31")
2. 调用 query_sales_data(start_date="2026-02-01", end_date="2026-02-29")
3. 对比结果

# 现在处理用户的问题
"""
```

---

### Chain of Thought（思维链）

引导 Agent 逐步思考：

```python
PROMPT = """
在回答问题前，请按以下步骤思考：

1. **理解问题**：用户想要什么信息？
2. **识别所需数据**：需要查询哪些数据？
3. **调用工具**：使用哪个工具？参数是什么？
4. **分析结果**：数据说明了什么？
5. **生成回答**：如何清晰地呈现结果？

请在回答中展示你的思考过程。
"""
```

---

### 输出格式控制

```python
PROMPT = """
请以 JSON 格式返回结果：

{
  "answer": "你的回答",
  "confidence": 0.95,  // 0-1 的置信度
  "sources": ["query_sales_data"]  // 使用的工具
}
"""
```

---

## 7. 常见 Agent 模式

### Pattern 1: React (Reason + Act)

1. **Reasoning**：分析问题
2. **Action**：调用工具
3. **Observation**：观察结果
4. **循环**：直到得出答案

### Pattern 2: Plan-and-Execute

1. **Plan**：制定多步骤计划
2. **Execute**：逐步执行
3. **Verify**：验证结果

### Pattern 3: Multi-Agent Debate

多个 Agent 对同一问题给出不同答案，通过辩论得出最优解。

---

## 8. 错误处理与降级

### Tool 调用失败

```python
try:
    result = query_sales_data(...)
except Exception as e:
    return {
        "error": f"数据查询失败: {str(e)}",
        "fallback": "请稍后重试或联系客服"
    }
```

### LLM 无法回答

```python
if "不知道" in response or "无法回答" in response:
    return "抱歉，我暂时无法回答这个问题。您可以尝试：\n1. 重新描述问题\n2. 联系人工客服"
```

---

## 9. 性能优化

### 缓存常见问题

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def query_faq(question: str) -> str:
    # 缓存 FAQ 答案
    pass
```

### 流式输出

```python
async for chunk in llm.astream(messages):
    print(chunk.content, end="", flush=True)
```

---

## 最佳实践

1. **Prompt 版本控制**：将 Prompt 存为文件，使用 Git 管理
2. **A/B 测试**：对比不同 Prompt 的效果
3. **日志记录**：记录 Agent 的每一步决策，便于调试
4. **安全性**：避免 Prompt 注入攻击（如用户输入 "忽略之前的指令"）

---

## 常见错误

- ❌ **Prompt 过长**：超过 4000 字导致 Token 浪费
  - ✅ 精简 Prompt，使用 Few-Shot 而非大量示例

- ❌ **Tool 参数不清晰**：Agent 不知道如何调用
  - ✅ 使用 Pydantic 严格定义参数类型

- ❌ **无限循环**：Agent 反复调用同一个 Tool
  - ✅ 设置最大步数限制
