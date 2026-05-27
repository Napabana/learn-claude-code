# S01 Agent Loop 学习观察

## 循环控制机制：`stop_reason`

从三次测试结果中观察到的核心规律：

| stop_reason | 含义 | 循环状态 |
|-------------|------|----------|
| `tool_use` | 模型需要调用工具 | **继续** |
| `end_turn` | 模型完成回答，无需更多工具 | **结束** |

## 三次测试分析

### 测试 1：创建文件
```
迭代 1: stop_reason = tool_use → 调用 echo 创建文件
迭代 2: stop_reason = tool_use → 调用 python 验证文件
迭代 3: stop_reason = end_turn → 循环结束
```

### 测试 2：列出 Python 文件
```
迭代 1: stop_reason = tool_use → 调用 find 搜索
迭代 2: stop_reason = tool_use → 调用 ls 精准列出
迭代 3: stop_reason = end_turn → 循环结束
```

### 测试 3：查看 git 分支
```
迭代 1: stop_reason = tool_use → 调用 git branch
迭代 2: stop_reason = end_turn → 循环结束
```

## 核心发现

**Agent 循环的本质**：
```
while stop_reason == "tool_use":
    # 模型调用工具
    # 工具执行后结果返回给模型
    # 模型决定：继续调用工具？还是结束？
```

- 模型可以**连续多次**调用工具（测试 1 和 2 都调用了 2 次）
- 只有当 `stop_reason != "tool_use"` 时，循环才退出
- 这是 Agent 能力扩展的基础：通过工具调用突破模型自身限制
