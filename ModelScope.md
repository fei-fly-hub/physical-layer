# ModelScope使用

## ModelScope 存储规则明确划分
1. **临时容器分区（重启销毁）**
    - `/usr` `/bin` `/lib` `/root` `/home/admin`
    - `apt / dpkg` 安装的所有系统包都写在这里
    - 实例停机、重启、重新创建开发环境 → 系统恢复干净出厂镜像，之前装的软件全部没了

2. **持久化存储（重启保留）**
    - 唯一目录：`/mnt/workspace`
    - 这里只能存**文件、源码、二进制程序、虚拟环境、自己解压的软件**
    - **不能持久化 apt 系统包**（apt 修改的是系统根目录，不在持久盘）

## 三种软件的生命周期对比
|安装方式|保存位置|重启后是否保留|
|---|---|---|
|`apt install xxx`|系统临时镜像|❌ 丢失，每次重启重新安装|
|`uv / pip install`（全局）|`/root/.local` 临时目录|❌ 丢失|
|`uv venv` 放在 `/mnt/workspace`|持久盘|✅ 保留|

## 工程上两种解决办法
### 方案1：开机自动脚本重装系统依赖
在 `/mnt/workspace/bootstrap.sh` 写脚本，每次启动实例执行一次自动安装
```bash
#!/bin/bash
sudo apt update
sudo apt install -y cmake g++ lksctp-tools
```

### 方案2：把预编译好的可执行文件放到持久目录
如果你不想每次 apt 编译，可以编译好二进制放到 `/mnt/workspace`，后续直接运行，不再依赖 apt。

## uv环境安装
### 方案B：安装到持久化目录（推荐，重启保留）
1. 把 uv 安装脚本、二进制、虚拟环境全部放在 `/mnt/workspace`
```bash
# 1. 指定安装路径到持久目录
export UV_INSTALL_DIR=/mnt/workspace/.uv-bin
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 把 uv venv 放在持久盘
cd /mnt/workspace
uv venv .venv
```
2. **开机自动加载环境变量**
在 `/mnt/workspace/startup.sh` 写初始化脚本；每次实例启动手动执行脚本恢复环境。

## 工程最佳实践（ModelScope固定套路）
1. **代码、工程、虚拟环境全部放在 `/mnt/workspace`**
2. 导出依赖列表 `uv pip freeze > requirements.txt`
3. 编写开机一键初始化脚本 `init_env.sh`
```bash
#!/bin/bash
export PATH="/mnt/workspace/.uv-bin:$PATH"
source /mnt/workspace/.venv/bin/activate
```
4. 每次新实例运行一次初始化脚本即可恢复完整环境
