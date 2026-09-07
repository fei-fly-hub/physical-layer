# GitHub Flow 完整落地实践
> 核心原则：
1. `main` 分支永远是**可直接部署到生产**的稳定代码；禁止直接往 main push；开启分支保护。
2. 所有改动全部来自**短期特性分支**，通过 PR 合并进 main。
3. PR 必须经过：CI检查 → Code Review → 合并，合并完成删除特性分支。
4. 合并 main 之后，就可以发布上线；需要版本就打 tag。
5. 没有 `develop/release` 长期分支。hotfix 和普通 bugfix 流程完全一致。

> 适合：Web服务、持续部署、中小型团队；不适合需要并行维护多个大版本的项目。

## 一、仓库前置配置（必须先做）
### 1. 分支保护规则 main
仓库 → Settings → Branches → Branch protection rules → Add rule，匹配 `main`：
- ✅ Require a pull request before merging（必须PR才能合并）
- ✅ Require approvals（Code Review 审批，一般设置1‑2人）
- ✅ Require status checks to pass before merging（CI必须全部通过才能合并）
- ✅ Do not allow bypassing the above settings（管理员也不能绕过）
- ❌ 禁止 force push 到 main
- ❌ 禁止删除 main

> 配置完任何人（包括管理员）不能直接 `git push origin main`。

### 2. 配套CI（GitHub Actions）
`.github/workflows/pr-ci.yml`
PR、push到main自动跑构建、单元测试、lint；CI不通过PR不允许合并。
```yaml
name: PR‑CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  ci:
    runs-on: ubuntu‑latest
    steps:
      - uses: actions/checkout@v4
      - name: install & lint & test & build
        run: |
          npm ci
          npm run lint
          npm run test
          npm run build
```

---

## 二、标准完整工作流程（日常开发）
> 流程：**拉最新main → 创建特性分支 → 本地提交 → 推送远程 → 创建PR → Review + CI → 合并main → 删除分支 → 发布**

### 1. 本地开始开发
每次开发前，务必同步远程 main 最新代码：
```bash
# 更新本地main
git checkout main
git pull origin main

# 创建特性分支，命名规范：feature/简短描述，issue号可选
git checkout -b feature/user‑register
```

> 分支命名约定（团队统一）
- `feature/xxx`：新功能
- `bugfix/xxx`：普通缺陷修复
- `docs/xxx`：文档修改
- `refactor/xxx`：代码重构
- `hotfix/xxx`：线上故障修复（**流程和普通分支完全一样，也是从main拉出**）

### 2. 本地开发提交
遵循 Conventional Commits 提交规范：
```bash
git add .
git commit -m "feat: add user register api"
# 或者修复：fix: fix register param validate
```

> 小步提交，不要一次性堆巨大提交。

### 3. 推送远程，创建Pull Request
```bash
git push origin feature/user‑register
```
推送完成，GitHub页面会弹出提示 `Compare & pull request`，点击创建PR。

#### PR填写规范
- Title：简洁，遵循 commit 风格，例：`feat: add user register api`
- Description：写改动说明、测试点、关联issue `Closes #123`
- Assignees：指派自己
- Reviewers：指定评审同事
- Linked issues：关联需求/缺陷issue

### 4. PR阶段
1. GitHub Actions自动触发CI，跑lint、test、build；**CI失败必须修复**。
2. Code Review：同事给出评论，本地修改，继续提交推送到同一分支，PR自动更新。
```bash
# 在自己的特性分支修改
git add .
git commit -m "fix: address review comment"
git push origin feature/user‑register
```

> 不要在PR里做大量合并main；如果main有新改动，优先 **rebase main** 保持线性历史。
```bash
# 在特性分支执行，把main最新提交rebase到自己分支
git checkout feature/user‑register
git fetch origin
git rebase origin/main
# 出现冲突解决冲突，git add，git rebase --continue
git push --force‑with‑lease origin feature/user‑register
```
> ⚠️ 只对**自己的私有特性分支**做force push；永远不要force main。

### 5. 合并PR
Review通过 + CI全部绿灯。
GitHub PR页面三种合并方式：
1. **Squash and merge（推荐GitHub Flow）**
把分支所有提交压缩成**一个干净commit**合并进main，历史整洁；适合大多数业务项目。
2. Rebase and merge：rebase后逐条提交保留；适合提交粒度很规范的项目。
3. Create a merge commit：产生merge节点；Git Flow常用，GitHub Flow一般不用。

> ✅ 推荐：**Squash and merge**。合并完成，勾选 `Delete branch`，删除远程特性分支。

### 6. 合并完成之后
1. 远程分支已删除；清理本地旧分支
```bash
git checkout main
git pull origin main
git branch -d feature/user‑register
git fetch --prune # 清理本地已经不存在的远程分支引用
```
2. main已经包含你的代码，**此时main已经可部署**。
3. 发布：部署脚本监听main分支push自动部署；如果需要版本，打tag：
```bash
git tag -a v1.4.0 -m "release v1.4.0"
git push origin v1.4.0
```

---

## 三、Hotfix线上紧急修复实践
GitHub Flow **没有单独hotfix分支模型**，hotfix就是普通bugfix流程，全部从main拉出：
1. main是当前生产代码，从main切分支 `hotfix/fix‑pay‑crash`
2. 修复，提交，推送，创建PR
3. CI + Review通过，合并回main
4. main合并完成，立刻部署生产；打补丁版本tag `v1.4.1`

> 和Git Flow区别：Git Flow hotfix合并main后还要合并回develop；GitHub Flow没有develop，做完就结束。

---

## 四、遇到的典型场景实操
### 场景1：PR评审提出大量修改意见
直接在当前分支继续提交，push，PR自动更新，不用新建PR。
不要合并main，优先 `rebase origin/main`。

### 场景2：PR还没合并，main已经有别人提交
```bash
git checkout feature/xxx
git fetch origin
git rebase origin/main
# 解决冲突
git push --force‑with‑lease origin feature/xxx
```

### 场景3：PR不再需要，废弃改动
直接关闭PR，删除远程分支即可，不需要合并。

### 场景4：需要回滚线上变更
GitHub页面：PR页面 → revert按钮，会自动创建一个反向变更PR。
新PR走完整CI+Review流程，合并main完成回滚。
> 不要手动去main强行回滚commit。

---

## 五、GitHub Flow约束与不适用场景
### ✅ 适合
- 持续部署，main合并就上线
- Web后端、前端、云服务
- 迭代快，不需要维护多个并行版本

### ❌ 不适合
1. 需要同时维护多个发布版本（例如 v1.x、v2.x 并行修bug）→ 此时更适合 GitFlow。
2. main无法保证随时可发布（经常有未完成大功能往main合）。
> 如果有大功能短期不能上线：使用**功能开关(feature flag)** 在代码里关闭逻辑，而不是长期保留feature分支。

---

## 六、GitHub Flow 团队落地检查清单
1. main开启分支保护，禁止直接push，强制PR+Review+CI。
2. 所有改动来自短期分支，分支生命周期：几小时 ~ 最多几天，禁止长期存活分支。
3. PR必须CI绿灯、至少1人review。
4. 合并优先 Squash and merge，合并后删除分支。
5. Hotfix流程和普通功能流程一致，不从其他分支拉。
6. main合并完成代表随时可以部署。
7. 使用tag标记版本，不使用release分支。

## 七、GitHub Flow vs Git Flow 快速对比
|项|GitHub Flow|Git Flow|
|---|---|---|
|长期分支|只有 main|main + develop|
|release分支|无|release/* 临时分支|
|hotfix来源|main|main|
|hotfix合并后|只合入main|合入main + develop|
|发布时机|PR合并main即可发布|release分支完成才发布|
|版本维护|只维护最新版本|支持多版本维护|

如果你需要，我可以给一份：
1. 团队可直接复制的 PR 模板 `.github/PULL_REQUEST_TEMPLATE.md`
2. 配套 GitHub Actions，main合并自动打版本、自动发布的工作流示例。