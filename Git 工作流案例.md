# GitHub Flow 完整实操案例

以一个模拟项目：`demo‑web`，仓库地址：`[https://github.com/demo-org/demo-web](https://github.com/demo-org/demo-web)`

> 
> 需求：实现「用户头像上传」功能，完整走一遍：拉仓库 → 创建分支 → 修改代码 → 推送 → PR → Review → 合并 → 清理分支。

## 前置条件

本地已经安装 Git；配置好 GitHub ssh key（推荐），或者使用 HTTPS。

---

## 步骤1：本地拉取 GitHub 项目

### 方式A：SSH（推荐，免密码）

```
# 克隆整个仓库到本地
git clone git@github.com:demo-org/demo-web.git

# 进入项目目录
cd demo-web

# 查看本地分支，默认只有 main
git branch
# * main
```

### 方式B：HTTPS

```
git clone [https://github.com/demo-org/demo-web.git](https://github.com/demo-org/demo-web.git)
cd demo-web
```

> 
> 首次开发前，确保本地 `main` 和远程同步

```
git checkout main
git pull origin main
```

`origin`：是远程仓库的别名，指向 GitHub 上的仓库。

---

## 步骤2：创建功能分支（GitHub Flow核心：不在main改代码）

需求：做头像上传功能，分支名：`feature/avatar‑upload`

```
# 基于最新 main 创建并切换到新分支
git checkout main
git pull origin main
git checkout -b feature/avatar-upload
```

> 
> `-b` 代表创建新分支。现在你工作在 `feature/avatar‑upload`，所有修改都在这里。

查看当前分支：

```
git branch
# main
# * feature/avatar-upload
```

## 步骤3：本地写代码、提交

修改代码：新增头像上传接口、页面组件。

查看改动文件：

```
git status
```

暂存 + 提交，遵循 Conventional Commits：

```
git add src/components/AvatarUpload.vue
git add src/api/user.js

git commit -m "feat: implement user avatar upload"
```

> 
> 可以多次提交，小步保存。

## 步骤4：把本地分支推送到 GitHub远程仓库

```
git push origin feature/avatar-upload
```

执行完，分支就出现在 GitHub 远程仓库。

> 
> ⚠️ 注意：push 的是**功能分支，不是 main**！main受保护，直接push会报错。

推送成功后，打开仓库网页，会出现提示：

> 
> `Compare & pull request` → 点击，直接新建 PR。

## 步骤5：创建 Pull Request

PR 表单填写示例

- Title：`feat: implement user avatar upload`
- Description：

```
## 改动说明
1. 新增头像上传组件
2. 对接上传接口
3. 增加文件大小校验

测试点：
- 正常图片上传
- 超大文件拦截

Closes #42  # 关联issue，合并后自动关闭issue
```

- Reviewers：选择同事A做评审
- Assignees：指派给自己
- 目标分支(base)：`main`；源分支(compare)：`feature/avatar‑upload`

点击 `Create pull request`。

创建PR之后，GitHub Actions自动触发CI：跑lint、单元测试、build。必须全部绿灯，才允许合并。

## 步骤6：代码评审，根据反馈修改

同事 Review 提出意见：需要增加图片格式校验。

回到本地，在**同一个功能分支**修改代码：

```
# 确认自己在功能分支
git checkout feature/avatar-upload

# 修改代码完成
git add .
git commit -m "fix: add image format validate"

# 直接推送到远程，PR会自动更新，不需要新建PR
git push origin feature/avatar-upload
```

### 场景：PR期间 main 有其他人合并了新代码，存在潜在冲突

需要把远程main的更新同步到自己分支，使用 rebase（保持线性历史，GitHub Flow推荐）

```
git checkout feature/avatar-upload
git fetch origin
git rebase origin/main
```

如果出现冲突：手动解决冲突文件，`git add`，然后

```
git rebase --continue
```

> 
> rebase完成推送，使用 `--force‑with‑lease`，禁止普通 `‑f` 暴力强制推送

```
git push --force-with-lease origin feature/avatar-upload
```

## 步骤7：合并PR

条件全部满足：

1. CI status checks 全部通过（绿灯）
2. Reviewer approve 通过

合并选项，**GitHub Flow推荐：Squash and merge**

- 将该分支所有提交压缩成1条干净commit写入main分支。
- 填写最终commit信息：`feat: implement user avatar upload`
- ✅ 勾选 `Delete branch`，合并完成自动删除远程 `feature/avatar‑upload`

点击确认合并。

## 步骤8：本地清理工作

远程分支已经删除，回到本地main拉取最新代码，并删除本地旧功能分支

```
git checkout main
git pull origin main

# 删除本地已经完成的功能分支
git branch -d feature/avatar-upload

# 清理本地无效远程引用
git fetch --prune
```

✅ 到此一轮 GitHub Flow 完整流程结束。
此时 main 已经包含头像上传代码，可以部署上线；如果需要版本：

```
git tag -a v1.5.0 -m "release v1.5.0: add avatar upload"
git push origin v1.5.0
```

---

# 案例：Hotfix线上bug完整示例

> 
> 场景：线上main发现头像上传会崩溃，紧急修复。GitHub Flow hotfix和普通流程完全一致，**从main拉出分支**。

```
git checkout main
git pull origin main
git checkout -b hotfix/fix‑avatar‑crash

# 修改bug
git commit -m "fix: fix avatar upload crash on empty file"
git push origin hotfix/fix‑avatar‑crash
```

1. GitHub网页创建PR到main
2. CI + Review
3. Squash合并main，删除分支
4. main更新后立刻部署生产，打补丁tag `v1.5.1`

---

# 常见踩坑

1. ❌ 在 main 分支直接改代码提交：main受保护，禁止这么做。
2. ❌ `git push origin main`：直接报错，分支保护拦截。
3. ❌ rebase公共main分支：只rebase自己私有feature分支。
4. ❌ 合并完忘记本地删除旧分支，堆积大量废弃本地分支。
5. ❌ 冲突用大量merge，造成历史混乱，优先 rebase。

---

# 常用命令速查表

```
git clone <url>                     # 拉取项目
git checkout main && git pull       # 更新本地主干
git checkout -b feature/xxx         # 创建功能分支
git status                          # 查看变更
git add . && git commit -m "msg"    # 提交
git push origin feature/xxx         # 推送分支到远程
git fetch origin                    # 获取远程最新信息，不改动本地
git rebase origin/main              # 把主干更新变基到当前分支
git branch -d feature/xxx           # 删除本地分支
git fetch --prune                   # 清理失效远程分支引用
```

如果你需要，我可以模拟一个冲突场景完整演示一遍冲突如何解决。