# 个人 Git 项目开发完整实践
> 个人项目特点：没有团队评审、没有多人协作冲突，但依然要规范分支、提交、版本发布；不需要复杂的 GitFlow，**优先轻量版 GitHub Flow**。
> 个人开发不建议完整 GitFlow（main+develop+release+hotfix 太重，个人维护两条长期分支徒增负担）。

适用场景：自己写Demo、开源个人项目、写工具、练手项目，单人维护。

## 推荐个人工作流方案
**简化版 GitHub Flow**
- 长期分支：`main`，main 始终保证可编译、可运行。
- 所有新功能、bugfix 全部新建短期分支，在分支开发。
- 自己本地合并到 main；重要版本打 tag。
- 不需要强制PR（自己一个人），但依然把分支和main分开。

> 只有当你的个人项目需要做多版本维护（比如同时维护 v1、v2 两个版本），才用 GitFlow。

---

# 完整实操步骤（从零开始，两种场景：全新初始化 / 克隆已有GitHub仓库）

## 场景A：全新本地项目，推送到GitHub
### 1. 本地初始化git
```bash
mkdir my‑tool
cd my‑tool
git init
# 创建初始文件
echo "# my‑tool 个人工具项目" > README.md
```

配置用户名邮箱（仅第一次本机配置）
```bash
git config user.name "你的名字"
git config user.email "你的github邮箱"
```

首次提交
```bash
git add README.md
git commit -m "docs: init project"
```

现在本地已经有 main 分支。

### 2. GitHub上新建空仓库
不要勾选 `Add README`，保持空仓库。
拿到仓库地址，关联远程：
```bash
# SSH 推荐
git remote add origin git@github.com:xxx/my‑tool.git

# 推main到远端
git push -u origin main
```

## 场景B：拉取已经存在的个人GitHub项目
```bash
git clone git@github.com:xxx/my‑tool.git
cd my‑tool
git checkout main
git pull origin main
```

---

# 个人开发完整循环（日常开发）
> 需求：开发一个新功能：实现日志导出功能

1. **先同步main，从main切功能分支**
```bash
git checkout main
git pull origin main
# 创建功能分支
git checkout -b feature/log‑export
```

> 分支命名（个人也建议遵守）
- `feature/xxx`：新功能
- `bugfix/xxx`：修复bug
- `refactor/xxx`：重构
- `docs/xxx`：文档修改

2. 在分支开发，小步提交
```bash
# 修改代码
git status
git add .
git commit -m "feat: 实现日志导出基础逻辑"

# 继续改，继续提交
git commit -m "feat: 增加日志文件按时间分割"
```

> 个人项目也建议用 **Conventional Commits**，以后看历史、生成CHANGELOG非常舒服。

3. 推送到远程备份（非常重要！防止本地电脑丢失代码）
```bash
git push origin feature/log‑export
```

> 个人项目：可以随时push分支到github做云端备份，不一定要合并。

4. 开发完成，合并回main
> 个人没有PR，本地合并。合并前最好把main最新代码rebase过来，保持历史干净。
```bash
git checkout feature/log‑export
git fetch origin
git rebase origin/main

# 切换回main，合并功能分支
git checkout main
git merge feature/log‑export
```

> 如果你想要干净线性历史，也可以squash合并，把多条提交压缩成一条：
```bash
git checkout main
git merge --squash feature/log‑export
git commit -m "feat: add log export function"
```

推main到远程
```bash
git push origin main
```

5. 删除已经完成的临时分支
```bash
# 删除本地分支
git branch -d feature/log‑export
# 删除远程分支
git push origin --delete feature/log‑export
```

✅ 一轮开发完成。

---

# 个人项目Hotfix（线上main有bug）
main已经发布，发现bug：
```bash
git checkout main
git pull origin main
git checkout -b bugfix/fix‑log‑crash
# 修改bug
git commit -m "fix: 修复日志导出空指针崩溃"
git push origin bugfix/fix‑log‑crash

# 修复完成合并回main
git checkout main
git merge bugfix/fix‑log‑export
git push origin main

git branch -d bugfix/fix‑log‑crash
git push origin --delete bugfix/fix‑log‑crash
```

---

# 版本发布打Tag
当完成一个可用版本，打语义化版本标签 `v主.次.补丁`：`v1.0.0`、`v1.0.1`
```bash
git tag -a v1.0.0 -m "release v1.0.0: 支持日志导出功能"
git push origin v1.0.0
```
之后可以在GitHub页面基于tag创建Release。

---

# 个人开发常见问题实操

## 1. 开发到一半，想临时切回main改别的东西，工作没做完
使用 `git stash` 暂存当前未提交改动
```bash
git stash save "暂存日志导出开发中代码"
git checkout main
# 在main或者别的分支做修复
# 做完切回来
git checkout feature/log‑export
git stash pop
```

## 2. 分支写乱了，想要撤销本地修改，回到远端main状态
```bash
git fetch origin
git reset --hard origin/main
```
> ⚠️ 会丢弃本地未提交改动，谨慎。

## 3. 个人项目要不要开develop分支？
**绝大多数个人项目不要。**
- 只有你需要并行维护多个版本，例如：v1继续修bug，v2做新大版本，才考虑用GitFlow模式增加develop分支。
- 普通个人项目：main + 短期feature分支足够。

## 4. 个人项目需要保护main分支吗？
GitHub仓库设置里可以开启分支保护：
- 禁止直接push main；
虽然是个人项目，开启后，强制你必须走分支开发，防止手滑直接在main乱提交。

## 5. 搭配GitHub Actions
个人项目也可以配置CI：push/pr自动跑测试、构建。
main分支保证绿灯，tag触发release打包制品。

---

# 两种个人项目模式对比
|模式|适用场景|
|---|---|
|简化GitHub Flow（推荐）|普通个人项目，持续迭代，只有一个活跃版本|
|完整GitFlow|个人开源库，需要同时维护v1、v2多个版本，正式版本发布|

---

# 极简个人开发命令速记
```bash
# 拉项目
git clone xxx
git checkout main && git pull

# 创建功能分支
git checkout -b feature/xxx

# 提交
git add .
git commit -m "feat: xxx"
git push origin feature/xxx

# 开发完合并
git checkout main
git pull
git merge feature/xxx
git push origin main

# 删除分支
git branch -d feature/xxx
git push origin --delete feature/xxx

# 打版本
git tag -a v1.0.0 -m "xxx"
git push origin v1.0.0
```

如果你需要，我可以给一份个人项目 `.gitignore` 模板，以及个人项目配套的最小GitHub Actions CI配置。