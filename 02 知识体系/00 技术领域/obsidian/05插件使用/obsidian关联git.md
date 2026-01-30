看到你的Git状态显示有中文文件名乱码问题，以及 `.obsidian` 文件夹未跟踪。我来帮你一步步解决：
> [!tip]
> Obsidian具体使用见： [[obsidian学习资料|Obsidian]]
## 第一步：解决中文乱码问题

在终端中运行：
```bash
# 1. 配置Git正确处理中文
git config --global core.quotepath false

# 2. 设置UTF-8编码
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

# 3. 设置Git Bash（如果是Windows）
# 在Git Bash中运行：
export LESSCHARSET=utf-8

# 4. 重新查看状态，现在应该显示正常中文
git status
```

## 第二步：创建合理的`.gitignore`文件

在Obsidian库根目录创建`.gitignore`文件：
```bash
# 使用命令行创建
echo "# Obsidian系统文件" > .gitignore
echo ".obsidian/" >> .gitignore
echo ".trash/" >> .gitignore
echo ".DS_Store" >> .gitignore
echo "Thumbs.db" >> .gitignore
```

更完整的`.gitignore`内容（推荐）：
```gitignore
# Obsidian特定文件
.obsidian/workspace
.obsidian/workspace-mobile
.obsidian/backups/
.obsidian/icon.svg
.obsidian/plugins/obsidian-git/data.json  # 如果有敏感信息
.obsidian/plugins/dataview/data.json      # 数据缓存

# 系统文件
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
*.tmp
*.swp
*.swo
*~

# 临时文件
.cache/
.temp/

# 大文件/媒体文件（建议单独管理）
*.mp4
*.mov
*.avi
*.mp3
*.wav
*.zip
*.rar
*.7z
*.iso
*.dmg

# 排除某些插件生成的文件
# 但保留配置，你可以根据需要调整
```

## 第三步：选择性添加`.obsidian`配置

如果你**想备份部分配置**（推荐）：
```bash
# 1. 先创建空的.obsidian文件夹的忽略规则
cat > .gitignore << 'EOF'
# 默认忽略所有.obsidian内容
.obsidian/*

# 但保留以下重要配置
!.obsidian/app.json
!.obsidian/core-plugins.json
!.obsidian/community-plugins.json
!.obsidian/appearance.json
!.obsidian/themes/
!.obsidian/snippets/
!.obsidian/plugins/
!.obsidian/plugins/*/manifest.json
!.obsidian/plugins/*/data.json
EOF

# 2. 创建插件数据忽略规则（如果需要）
echo "!.obsidian/plugins/obsidian-git/data.json" >> .gitignore
```

如果你**不想备份任何配置**（更简单）：
```bash
# 直接忽略整个.obsidian文件夹
echo ".obsidian/" > .gitignore
```

## 第四步：添加和提交文件

```bash
# 1. 添加.gitignore文件
git add .gitignore

# 2. 添加所有其他文件（排除.gitignore中的）
git add .

# 3. 查看添加的文件（确保没有添加不该添加的）
git status

# 4. 如果是首次提交，设置用户名和邮箱
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

# 5. 提交
git commit -m "初始提交：Obsidian笔记库"

# 6. 查看提交历史
git log --oneline --graph
```

## 第五步：处理特殊文件夹

从你的文件列表看，有些文件夹可能需要特殊处理：

```bash
# 如果某些文件夹是空的，Git不会跟踪空文件夹
# 可以在空文件夹中创建.gitkeep文件
touch 工作汇报/.gitkeep
touch 接口文档/.gitkeep
touch 知识库/.gitkeep
touch 问题排查/.gitkeep

# 然后添加这些占位文件
git add */.*keep 2>/dev/null || true
```

## 第六步：配置Obsidian Git插件（可选但推荐）

如果你已经安装了Obsidian Git插件，还需要配置：

1. 在Obsidian中：设置 → Obsidian Git
2. 禁用自动提交`.obsidian`文件夹（避免配置冲突）
3. 设置提交间隔（建议每10分钟或手动）
4. 配置提交消息模板

## 第七步：创建远程仓库并推送

```bash
# 1. 在GitHub/Gitee创建新仓库
# 例如在GitHub：https://github.com/new

# 2. 关联远程仓库（复制仓库URL）
git remote add origin https://github.com/你的用户名/仓库名.git

# 3. 重命名分支（如果默认为master，而GitHub使用main）
git branch -M main  # 或者保持master

# 4. 推送代码
git push -u origin main  # 或 master

# 如果提示认证错误，可能需要：
# - 配置SSH密钥
# - 或使用GitHub Personal Access Token
```

## 检查你的最终状态

运行以下命令检查一切是否正常：
```bash
# 检查状态（应该显示正常中文）
git status

# 检查忽略规则
git check-ignore -v .obsidian/workspace

# 查看已跟踪的文件
git ls-tree -r main --name-only | head -20
```

## 快速一键配置脚本

如果你想快速完成所有配置，创建一个`setup-git.sh`文件：

```bash
#!/bin/bash
# Obsidian Git初始化脚本

echo "🔧 配置Git中文支持..."
git config --global core.quotepath false
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

echo "📝 创建.gitignore文件..."
cat > .gitignore << 'EOF'
# Obsidian
.obsidian/workspace*
.obsidian/workspace-mobile
.obsidian/backups/
.obsidian/icon.svg

# 系统
.DS_Store
Thumbs.db
*.tmp
*~

# 临时文件
.cache/
.trash/
EOF

echo "👤 设置Git用户信息..."
read -p "输入你的名字: " username
read -p "输入你的邮箱: " useremail
git config --global user.name "$username"
git config --global user.email "$useremail"

echo "📦 添加文件到Git..."
git add .
git commit -m "初始提交: $(date '+%Y-%m-%d %H:%M')"

echo "✅ 完成！现在可以关联远程仓库了"
echo "运行: git remote add origin <仓库URL>"
echo "然后: git push -u origin main"
```

运行脚本：
```bash
chmod +x setup-git.sh
./setup-git.sh
```

现在你的Obsidian库应该已经正确关联Git了!！