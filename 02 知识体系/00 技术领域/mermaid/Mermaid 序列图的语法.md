> [!info]
> 我来为你详细讲解 Mermaid 序列图的语法，包括你示例中的用法和其他重要语法。

## 📊 你的示例语法解析

```mermaid
sequenceDiagram
    Alice->>+John: Hello John, how are you?
    Alice->>+John: John, can you hear me?
    John-->>-Alice: Hi Alice, I can hear you!
    John-->>-Alice: I feel great!
```

**符号含义：**

- `->>` 实线箭头（同步消息）
- `-->>` 虚线箭头（异步消息）
- `+` 激活会话开始（目标参与者被激活）
- `-` 激活会话结束
- `:` 后面的文字是消息内容

## 📝 完整语法详解

### 1. **基础结构**

```mermaid
sequenceDiagram
    participant A as 客户端
    participant B as 服务器
    A->>B: 请求数据
    B-->>A: 返回数据
```

### 2. **参与者定义**

```mermaid
sequenceDiagram
    participant A
    participant B as 后端服务
    actor C as 管理员
    Note over A,B: 这两个是主要交互方
```

### 3. **消息类型**

```mermaid
sequenceDiagram
    A->>B: 同步消息
    A-->>B: 异步消息
    A-xB: 超时/失败消息
    A--xB: 异步超时
    A->>+B: 激活会话
    B-->>-A: 结束会话
```

### 4. **激活框（生命线激活）**

```mermaid
sequenceDiagram
    activate A
    A->>+B: 激活B
    B->>+C: 激活C
    C-->>-B: 响应
    B-->>-A: 响应
    deactivate A
```

### 5. **循环与判断**

```mermaid
sequenceDiagram
    loop 每次查询
        A->>B: 检查状态
        alt 状态正常
            B-->>A: 返回数据
        else 状态错误
            B-->>A: 返回错误
        end
    end
    
    opt 可选操作
        A->>B: 可选请求
    end
```

### 6. **并行处理和注释**

```mermaid
sequenceDiagram
    par 并行处理
        A->>B: 请求1
    and
        A->>C: 请求2
    end
    
    Note right of A: 这是注释<br>支持多行
    Note over B,C: 跨参与者注释
```

### 7. **消息序号和颜色**

```mermaid
sequenceDiagram
    autonumber
    A->>B: 消息1
    B-->>A: 响应1
    A->>B: 消息2
    Note over A,B: 自动编号
    
    rect rgb(200, 220, 255)
        A->>B: 高亮区域
        B-->>A: 响应
    end
```
