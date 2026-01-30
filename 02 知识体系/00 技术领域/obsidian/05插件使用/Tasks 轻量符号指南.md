## 核心思路
用 `!` `?` `~` `@` 四个符号快速标注任务，配合Tasks插件直接筛选。无需复杂配置，书写与查询都简单高效。


---

## 一、基础用法
### 书写格式
1. `- [ ] 符号 任务内容`  （符号前后均为**英文空格**）
2. `- 符号 任务内容`  （符号前后均为**英文空格**）

==2种方式都可以==
### 符号含义
| 符号 | 含义         | 适用场景                       |
|------|--------------|--------------------------------|
| `!`  | 高优先级     | 紧急任务、核心工作             |
| `?`  | 待确认       | 需核实信息、依赖他人的任务     |
| `~`  | 低优先级     | 不紧急、可延后处理的任务       |
| `@`  | 需跟进/服务  | 客户服务、业务对接、等待反馈   |

### 示例
```markdown
- [ ] ! 紧急修复线上问题 #task
- [ ] ? 确认会议时间 #task
- [ ] ~ 整理文档 #task
- [ ] @ 跟进客户需求 #task
```
✅ 建议添加 `#task` 标签，便于插件精准筛选。

---

## 二、进阶：符号 + 时间功能
符号后可直接追加Tasks的时间属性。

### 示例
```markdown
- [ ] ! 完成报告 📅 2026-02-05 #task
- [ ] @ 客户回访 🔁 every week #task
- [ ] ? 确认需求 ⏳ tomorrow #task
```

---

## 三、插件配置（1步）
让插件原生识别 `!` 和 `~` 为优先级：
1. **设置 → 第三方插件 → Tasks**
2. 找到 **Priority Settings**
3. 设置：**High** = `!`，**Low** = `~`
4. 保存即可

✅ `?` 和 `@` 无需配置，通过查询筛选。

---

## 四、查询模板（复制即用）
粘贴到 ` ```tasks ` 代码块中。

### 1. 高优先级（`!`）
```tasks
not done
priority is high
tag includes #task
```

### 2. 需跟进（`@`）
```tasks
not done
description includes "@ "
tag includes #task
```

### 3. 待确认（`?`）
```tasks
not done
description includes "? "
tag includes #task
```

### 4. 低优先级（`~`）
```tasks
not done
priority is low
tag includes #task
```

### 5. 核心工作汇总（`!` + `@`）
```tasks
not done
((priority is high) OR (description includes "@ "))
(tag includes #task)
group by folder
```

### 6. 全符号任务看板
```tasks
not done
((priority is high) OR (priority is low) OR (description includes "? ") OR (description includes "@ "))
tag includes #task
group by priority
sort by due desc
```
### 7. 近7天完成的符号任务
```tasks
done
done after 7 days ago
((priority is high) OR (priority is low) OR (description includes "? ") OR (description includes "@ "))
tag includes #task
sort by done desc
group by folder
```
---

## 五、视觉高亮（可选CSS）
让符号在阅读模式中显示颜色。

1. **设置 → 外观 → CSS片段 → 打开文件夹**
2. 新建 `task-symbols.css`，粘贴以下代码：
```css
/* 符号高亮 */
.markdown-rendered li:has(input[type="checkbox"]) > p:contains("! ") {
  color: #dc2626; font-weight: 700;
}
.markdown-rendered li:has(input[type="checkbox"]) > p:contains("@ ") {
  color: #2563eb; font-weight: 700;
}
.markdown-rendered li:has(input[type="checkbox"]) > p:contains("? ") {
  color: #06b6d4; font-weight: 600;
}
.markdown-rendered li:has(input[type="checkbox"]) > p:contains("~ ") {
  color: #64748b; font-weight: 600;
}
```
3. 返回设置，**启用该片段**。

---

## 六、常见问题
- **`!`/`~` 筛选无效**：检查插件优先级设置是否正确。
- **`@`/`?` 查不到**：查询中确保符号后带空格（如 `"@ "`）。
- **符号高亮不显示**：确认CSS片段已启用，且任务使用英文符号。
- **一个任务只用一种符号**，避免混合书写（如 `!@`）。

---

## 七、最佳实践
1. **随记随标**：按格式 `- [ ] 符号 任务` 快速记录。
2. **集中看板**：将常用查询模板放在一个笔记中，作为工作入口。
3. **善用`@`**：客户服务、跟进类任务统一用 `@`，方便单独筛选。

---
简洁高效，聚焦执行。