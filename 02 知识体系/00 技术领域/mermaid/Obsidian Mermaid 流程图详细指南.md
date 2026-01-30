
## 📋 **基础启用**
Mermaid 是 Obsidian 的**核心插件**，无需安装：
1. 打开 **设置 → 核心插件**
2. 找到 **"绘图"**（或 "Mermaid"）并启用
3. 在笔记中创建代码块：````mermaid````

## 🔤 **基础语法结构**

### 1. **基本流程图**
```mermaid
graph TD
    A[开始] --> B{是否登录?}
    B -->|是| C[进入首页]
    B -->|否| D[跳转登录页]
    C --> E[浏览内容]
    D --> F[输入凭证]
    F --> C
```

**代码：**
````
```mermaid
graph TD
    A[开始] --> B{是否登录?}
    B -->|是| C[进入首页]
    B -->|否| D[跳转登录页]
    C --> E[浏览内容]
    D --> F[输入凭证]
    F --> C
```
````

### 2. **方向控制**
| 方向代码 | 含义 | 英文 |
|---------|------|------|
| `TD` 或 `TB` | 从上到下 | Top to Bottom |
| `BT` | 从下到上 | Bottom to Top |
| `RL` | 从右到左 | Right to Left |
| `LR` | 从左到右 | Left to Right |

## 🎨 **节点样式详解**

### **节点形状**
```mermaid
graph TD
    A[方形] --> B(圆角)
    B --> C((圆形))
    C --> D{菱形}
    D --> E>非对称]
    E --> F{{六边形}}
```

**代码：**
````
```mermaid
graph TD
    A[方形] --> B(圆角)
    B --> C((圆形))
    C --> D{菱形}
    D --> E>非对称]
    E --> F{{六边形}}
```
````

### 自定义节点
```mermaid
graph LR
    id1[方形] --> id2(圆角方形)
    id2 --> id3([体育场形])
    id3 --> id4[[子程序形]]
    id4 --> id5[(圆柱形)]
    id5 --> id6((圆形))
    id6 --> id7{菱形}
    id7 --> id8{{六边形}}
    id8 --> id9>非对称形]
    id9 --> id10[/平行四边形/]
    id10 --> id11[\反向平行四边形\]
    id11 --> id12[/梯形\]
    id12 --> id13[\反向梯形/]
```

## 🔗 **连接线样式**

### **基础连接**
```mermaid
graph LR
    A -- 直线 --> B
    A -->|带文字| C
    A -. 虚线 .-> D
    A -. 带文字 .-> E
    A ==> 粗箭头 ==> F
    A -- 普通 --- G
    A --- 无箭头 --- H
```

### **连接线长度**
```mermaid
graph LR
    A -- 单线 --> B
    A === 三线 === C
    A -.- 点线 -.- D
    A -.- 带文字点线 -.-> E
```

## 🏷️ **文本和样式**

### **1. 节点内换行**
```mermaid
graph TD
    A[第一行<br/>第二行] --> B{决策<br/>节点}
```

### **2. 使用HTML标签**
```mermaid
graph LR
    A[<u>下划线</u><br/><b>加粗</b>] --> B[<i>斜体</i><br/><font color="red">红色</font>]
```

## 🎨 **样式美化**

### **1. 类定义**
```mermaid
graph TD
    A[开始] --> B{检查}
    B --> C[正常流程]
    B --> D[异常流程]
    
    classDef default fill:#f9f,stroke:#333,stroke-width:2px
    classDef process fill:#bbf,stroke:#66f,stroke-width:2px,color:#000
    classDef decision fill:#fbf,stroke:#f6f,stroke-width:2px
    classDef exception fill:#fbb,stroke:#f66,stroke-width:2px
    
    class A,B process
    class C decision
    class D exception
```

### **2. 直接样式**
```mermaid
graph LR
    id1(Start)-->id2(Stop)
    style id1 fill:#f9f,stroke:#333,stroke-width:4px
    style id2 fill:#bbf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5 5
```

## 🔄 **子图和分组**

### **子图示例**
```mermaid
graph TB
    subgraph 用户模块
        A1[注册] --> A2[登录]
        A2 --> A3[个人中心]
    end
    
    subgraph 管理模块
        B1[审核] --> B2[配置]
        B2 --> B3[统计]
    end
    
    A3 --> B1
    B3 -.-> A1
```

## 📊 **复杂流程图示例**

### **登录流程**
```mermaid
graph TD
    Start[访问网站] --> CheckCookie{有登录Cookie?}
    CheckCookie -->|有| DirectLogin[直接登录]
    CheckCookie -->|无| ShowLogin[显示登录页面]
    
    ShowLogin --> Input[输入用户名密码]
    Input --> Validate{验证凭证}
    
    Validate -->|成功| SetCookie[设置Cookie]
    Validate -->|失败| Error[显示错误信息]
    
    SetCookie --> Success[登录成功]
    DirectLogin --> Success
    Error --> ShowLogin
    
    Success --> End[进入系统]
    
    classDef startEnd fill:#90EE90,stroke:#006400
    classDef process fill:#E0FFFF,stroke:#1E90FF
    classDef decision fill:#FFFACD,stroke:#FFD700
    classDef error fill:#FFB6C1,stroke:#DC143C
    
    class Start,End startEnd
    class ShowLogin,Input,SetCookie,DirectLogin process
    class CheckCookie,Validate decision
    class Error error
```

## ⚡ **实用技巧**

### **1. 注释使用**
```mermaid
graph LR
%% 这是一条注释，不会显示
    A --> B
    %% 在B之后处理
    B --> C
```

### **2. 链接功能**
```mermaid
graph LR
    A[查看详情] --> B[[内部链接]]
    click A "https://obsidian.md" _blank
    click B "obsidian://open?vault=myvault&file=笔记名" _self
```

### **3. 多行代码美化**
```mermaid
graph TD
    A[开始处理] --> B{
        这是一个很长的<br/>
        决策描述文本，<br/>
        需要换行显示
    }
    B -->|选项1| C[操作1]
    B -->|选项2| D[操作2]
```

## 🚀 **高级功能**

### **时序图结合**
```mermaid
graph TD
    subgraph 准备阶段
        A[需求分析] --> B[技术选型]
        B --> C[环境搭建]
    end
    
    subgraph 开发阶段
        D[模块开发] --> E[单元测试]
        E --> F[集成测试]
    end
    
    subgraph 部署阶段
        G[部署上线] --> H[监控运维]
    end
    
    C --> D
    F --> G
```

## 💡 **最佳实践建议**

1. **命名规范**：使用有意义的节点ID，如 `login_start`、`check_auth`
2. **保持简洁**：一个流程图不要超过15个节点
3. **颜色编码**：用颜色区分不同类型节点
4. **方向一致**：尽量使用 `TD`（从上到下）或 `LR`（从左到右）
5. **注释说明**：复杂逻辑添加注释

## 🔧 **问题排查**

常见问题：
- **不显示图表**：检查是否启用了Mermaid核心插件
- **语法错误**：注意缩进和符号（特别是空格）
- **样式无效**：确认类名是否正确应用

Obsidian的Mermaid支持实时预览，边写边看效果，非常方便！