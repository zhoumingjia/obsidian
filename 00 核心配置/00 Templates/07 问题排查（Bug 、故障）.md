<%*
const now = moment();
const bugName = await tp.system.prompt("问题名称", "双录系统音视频卡顿");
const system = await tp.system.prompt("涉及系统", "音视频集群");
const severity = await tp.system.suggester(["致命", "严重", "一般", "轻微"], ["致命", "严重", "一般", "轻微"]);

// 现在可以安全地使用 bugName
const fileName = `问题排查-${bugName.replace(/\//g, "-")}-${now.format("YYYYMMDD")}.md`;
const folder = "03 工作项目";

// 创建文件夹（如果不存在）
if (!app.vault.getAbstractFileByPath(folder)) {
  await app.vault.createFolder(folder);
}

const fullPath = `${folder}/${fileName}`;

// 检查文件是否已存在
if (app.vault.getAbstractFileByPath(fullPath)) { 
  new Notice("⚠️ 记录已存在！"); 
  return; 
}

// 创建文件内容
const content = `---
title: 问题排查 - ${bugName}
系统: ${system}
严重程度: ${severity}
排查日期: ${now.format("YYYY-MM-DD")}
状态: 待排查
tags: [问题排查, ${system}, ${severity}]
---

# 问题排查：${bugName}
## 一、问题基本信息
| 项目 | 内容 |
|------|------|
| 涉及系统 | ${system} |
| 严重程度 | ${severity} |
| 出现场景 | （如：并发量1000+时） |
| 影响范围 | （如：所有用户/特定地区用户） |

## 二、排查过程
1.  **初步定位**：
    - 现象：
    - 猜测原因：
2.  **排查步骤**：
    - 步骤1：（如：查看日志/监控）
    - 结果：
    - 步骤2：
    - 结果：

## 三、根因分析
- 根本原因：
- 关联因素：

## 四、解决方案
1.  临时方案：
2.  永久方案：
3.  验证结果：

## 五、复盘总结
- 预防措施：
- 优化建议：
- 经验教训：
`;

// 创建文件
await app.vault.create(fullPath, content);
new Notice("✅ 问题排查记录已创建！");

// 打开新创建的文件
const newFile = app.vault.getAbstractFileByPath(fullPath);
if (newFile) {
  await app.workspace.getLeaf().openFile(newFile);
}
%>