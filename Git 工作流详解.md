# Git 工作流详解
Git 工作流是团队使用 Git 进行分支管理、代码合并、发布上线的一套规范流程。不同于 GitHub Actions Workflow（CI自动化），**Git工作流是人为约定的分支协作模型**。

> 主流模型：
> 1. Git Flow（传统成熟项目）
> 2. GitHub Flow（简单主干开发，GitHub原生）
> 3. GitLab Flow（环境分支，多环境部署）
> 4. Trunk‑Based Development 主干开发（高频迭代，大厂常用）

## 基础概念回顾
- `main/master`：主分支，代表生产环境代码，**随时可发布**
- feature：功能分支，开发新功能
- bugfix/hotfix：缺陷修复，hotfix 生产紧急修复
- release：发布准备分支
- tag：版本标签，标记发布版本 `v1.2.0`

---

# 1. Git Flow（Vincent Driessen 经典模型）
最经典，适合**版本周期长、有规划发布、不持续部署**的项目。
### 永久长期分支
1. `main`：生产代码，每一个提交对应一个上线版本，只接受 merge，不在此直接提交代码，打版本tag
2. `develop`：开发主分支，存放下一个版本的开发集合，所有feature合并到此

### 临时分支（用完即删）
- `feature/*`：新功能，从 `develop` 拉出，完成合并回 `develop`，删除feature分支
- `release/*`：发布准备分支，从 `develop` 拉出，只做bug修复、版本号修改，不新增功能；完成后合并到 `main` 和 `develop`，打tag，删除release
- `hotfix/*`：线上紧急bug修复，**从main拉出**；修复完合并回 `main`（打tag），同时合并回 `develop`，删除hotfix

### Git Flow 完整流程示例
1. **开发新功能**
```bash
git checkout develop
git checkout -b feature/user-login
# ...开发、提交
git checkout develop
git merge feature/user-login
git branch -d feature/user-login
```

2. **准备版本发布 release**
```bash
git checkout develop
git checkout -b release/v1.3.0
# 修改版本号，修复测试bug
git checkout main
git merge release/v1.3.0
git tag v1.3.0

git checkout develop
git merge release/v1.3.0
git branch -d release/v1.3.0
```

3. **线上紧急修复 hotfix**
```bash
git checkout main
git checkout -b hotfix/fix-crash
# 修复bug
git checkout main
git merge hotfix/fix-crash
git tag v1.3.1

git checkout develop
git merge hotfix/fix-crash
git branch -d hotfix/fix-crash
```

✅ 优点：版本清晰，适合固定版本迭代、软件打包发布
❌ 缺点：分支多，流程重；不适合持续部署、每日多次发布

---

# 2. GitHub Flow（简单轻量，GitHub推荐）
> 核心思想：**main永远是可上线状态；所有改动走feature分支 + Pull Request；保护main分支禁止直接push**
没有 `develop`、release、hotfix 长期分支。

### 流程
1. 从 `main` 拉取 feature/bugfix 分支：`feature/xxx`
2. 在分支提交代码，推送到远程，创建 **Pull Request(PR)**
3. Code Review + CI自动化检查通过
4. 合并 PR 到 main，**删除功能分支**
5. 合并到 main 后，即可部署上线；需要版本就打 tag

hotfix 处理：同样新建 bugfix 分支从 main 拉出，走PR合并回main。

> 适合：Web服务、持续部署，小团队，迭代快。

```bash
git checkout main
git pull
git checkout -b feature/pay
# 开发提交
git push origin feature/pay
# 网页创建PR → review → merge to main → delete branch
```

✅ 优点：极简，上手快，配合CI持续部署非常舒服
❌ 缺点：不支持多版本并行维护；如果不能做到main随时可发布，这套就会崩掉。

---

# 3. GitLab Flow
在 GitHub Flow 基础上增加**环境分支**，适配多环境：`main → staging → prod`；也支持维护多版本。
流程：
1. feature分支从main开发，MR合并main
2. main合并到 `staging`（测试环境），测试验证
3. staging验证通过合并到 `prod`（生产）

也兼容 Git‑flow 的 release、hotfix。适合多环境（测试、预发、生产）企业项目。

---

# 4. Trunk‑Based Development 主干开发（TBD，大厂常用）
> 核心：**所有人尽量向主干(main/trunk)提交；短期存活分支，分支存活时间几小时~最多一两天，禁止长期分支**。
- 不搞长期 develop、release；
- 功能没做完用**功能开关(feature flag)**隐藏，而不是新建长期feature分支；
- 短生命周期feature分支，合并到主干；主干稳定随时发布；
- 如果做多版本维护，使用 release tag 打出版本，从tag拉出修复分支。

✅ 优点：减少大量分支合并冲突，适合高频持续集成持续部署
❌ 要求：强CI保障、单元测试、功能开关；对团队质量要求高。

---

# 各工作流对比总结表
|工作流|长期分支|适用场景|
|---|---|---|
|Git Flow|main + develop|传统软件，周期版本发布，需要维护多个版本|
|GitHub Flow|仅main|持续部署web项目，简单团队|
|GitLab Flow|main + staging + prod|多环境测试预发生产|
|Trunk主干开发|仅main/trunk|高频迭代，CI完善，互联网大厂|

---

# 团队实操最佳实践（通用）
1. **禁止直接 push main/master**，开启分支保护，所有变更走 PR/MR + Code Review
2. 分支命名规范（团队约定）
    - `feature/xxx‑desc`：新功能
    - `bugfix/xxx‑desc`：普通bug修复
    - `hotfix/xxx‑desc`：线上紧急修复
    - `refactor/xxx`：重构
    - `docs/xxx`：文档修改
3. 提交信息规范 `feat: add login api` / `fix: fix login crash` (Conventional Commits)
4. 合并前尽量 `git rebase main`，把主干更新拿过来，减少合并冲突，保持提交历史干净
5. 临时分支合并完成务必删除远程+本地分支
6. 版本使用 tag：`vMAJOR.MINOR.PATCH` 语义化版本 `v2.1.3`

## rebase vs merge
- `merge`：保留完整分支历史，Git Flow 多用merge；会产生merge commit
- `rebase`：把你的提交挪到主干最新节点，历史线性干净；GitHub Flow 常用rebase合并PR
> ⚠️ **不要对已经推送到远程公共分支执行rebase！只rebase自己本地私有分支**

---

# Git Flow vs GitHub Actions Workflow 区分（容易混淆）
- **Git工作流**：Git分支、合并、PR的**人工协作流程**（代码怎么组织）
- **GitHub Actions Workflow**：`.github/workflows/*.yml`，**CI/CD自动化脚本**（代码推送后自动跑构建、测试、部署）

两者经常搭配：
> Git工作流规定分支怎么写；GitHub Actions监听分支push/PR自动执行CI校验、部署。

如果你需要，我可以提供：
1. 一份团队直接落地可用的分支规范模板
2. Git Flow / GitHub Flow 常用命令速查表
3. 结合 GitHub Actions 的完整工程示例。