# Git‑Flow (Vincent Driessen) 完整落地实践
> 核心背景：适合**版本化发布软件**，比如客户端、SDK、有计划版本迭代，需要同时维护多个线上版本；**不适合高频持续部署**。
> 两大长期常驻分支：
> - `main`：对应生产环境，每一个提交都应该可以打版本发布，只接受 merge，禁止直接提交代码，版本打 tag。
> - `develop`：开发主分支，下一个版本的开发集合，所有新功能全部合并到此分支。
>
> 三类临时分支（**用完必须删除**）：
> 1. `feature/*`：新功能，从 `develop` 切出
> 2. `release/*`：版本发布准备分支，从 `develop` 切出
> 3. `hotfix/*`：线上紧急故障修复，从 `main` 切出

> ⚠️ Git‑Flow 和 GitHub Flow 最大区别：
> - Git‑Flow：有两条永久分支 `main + develop`；有 release 发布分支；hotfix 修改完要同时合并回 main **和 develop**。
> - GitHub Flow：只有 main，没有 develop、release。

> 两种使用方式：
> 1. 原生 git 命令手动操作（下面案例，推荐理解原理）
> 2. git‑flow 工具封装命令 `git flow feature start xxx`

## 准备工作：初始化仓库
仓库地址：`git@github.com:myorg/demo‑sdk.git`

### 1）克隆项目
```bash
git clone git@github.com:myorg/demo-sdk.git
cd demo-sdk
```

### 2）创建长期 develop 分支
> 新建仓库默认只有 main，需要手动创建 develop 长期分支
```bash
git checkout main
git checkout -b develop
git push -u origin develop
```

> **分支保护配置（必须配置）**
仓库设置 → Branches 保护规则
1. `main`：禁止直接push；只能接受 merge；合并进来的内容来自 release/*、hotfix/*；打 tag。
2. `develop`：禁止直接push；只能接受 feature/*、release/* 的合并；所有功能走 PR/MR。

---

# 完整实操案例流程
> 项目现状：
> - main 当前版本 `v1.2.0`（线上版本）
> - develop 正在开发下一个大版本 `v1.3.0`
>
> 完整走一遍：开发功能 → release版本准备 → 发布上线 → 线上紧急hotfix修复。

## 场景一：开发新功能 feature
需求：增加文件导出功能 `feature/file‑export`，**从 develop 分支拉出**。

```bash
# 1. 切到develop，同步最新代码
git checkout develop
git pull origin develop

# 2. 创建feature功能分支
git checkout -b feature/file-export
```

编写代码，多次提交：
```bash
git add .
git commit -m "feat: add file export module"
git commit -m "feat: support csv export"
```

推送到远程，创建 PR，目标分支为 **`develop`**（⚠️不是main）
```bash
git push origin feature/file-export
```

PR评审，CI通过后，合并进 `develop`，删除 feature 分支。
```bash
git checkout develop
git merge --no-ff feature/file-export
git branch -d feature/file-export
git push origin develop
```
> `--no‑ff`：禁止快进合并，生成一个 merge commit，明确记录分支合并轨迹，Git‑Flow 规范要求。

> ✔️ feature 生命周期结束，分支删除。多个feature依次合并到develop，积累下一个版本的全部功能。

## 场景二：版本发布准备 release/v1.3.0
当 develop 上积累完 v1.3.0 需要的全部功能，准备发布。
> release 分支规则：**只改bug、版本号；禁止新增新功能**。新功能留给下一轮 feature。

```bash
# 1. 基于最新develop创建release分支
git checkout develop
git pull origin develop
git checkout -b release/v1.3.0
```

在 release/v1.3.0 做：
- 修改项目版本号 `package.json` → `1.3.0`
- 测试、修复bug；**不新增功能**

提交修改：
```bash
git commit -m "chore: bump version to 1.3.0"
git commit -m "fix: fix export filename bug"
```

推送到远程，测试团队基于这个 release/v1.3.0 做回归测试。
```bash
git push origin release/v1.3.0
```

> 测试期间发现bug，直接在 release/v1.3.0 分支修复提交。**不要回到develop改**。

✅ 测试全部通过，正式发布：
> release 分支需要**双向合并**：合并到 main（生产），同时合并回 develop。

### 第一步：合并 release/v1.3.0 → main
```bash
git checkout main
git pull origin main
git merge --no-ff release/v1.3.0
# main上打版本tag
git tag -a v1.3.0 -m "release v1.3.0"
git push origin main
git push origin v1.3.0
```

### 第二步：合并 release/v1.3.0 → develop
> ⚠️非常关键！把release上所有bug修复、版本号变更同步回开发分支develop，避免后续版本丢失修复。
```bash
git checkout develop
git pull origin develop
git merge --no-ff release/v1.3.0
git push origin develop
```

### 删除临时release分支（用完销毁）
```bash
git branch -d release/v1.3.0
git push origin --delete release/v1.3.0
```
> release/v1.3.0 工作全部完成。此时 main=v1.3.0；develop 继续迭代未来 v1.4.0。

## 场景三：线上紧急bug修复 hotfix
> 线上生产 main(v1.3.0) 发现严重崩溃bug，需要紧急出补丁 v1.3.1。
> ✅ hotfix **从 main 分支拉出，不是develop**。

```bash
# 基于main创建hotfix分支
git checkout main
git pull origin main
git checkout -b hotfix/fix‑export‑crash
```

修复线上bug：
```bash
git commit -m "fix: fix export crash with empty data"
git commit -m "chore: bump version to 1.3.1"
git push origin hotfix/fix‑export‑crash
```

> hotfix同样双向合并：合并 main；同时合并回 develop。

### 合并 hotfix → main，打补丁tag
```bash
git checkout main
git merge --no-ff hotfix/fix‑export‑crash
git tag -a v1.3.1 -m "release v1.3.1 hotfix crash fix"
git push origin main
git push origin v1.3.1
```

### 合并 hotfix → develop（重点：同步修复到开发分支）
```bash