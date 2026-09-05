
# Bash 命令行提示符缩短配置指南

## 原理说明
终端过长提示符由环境变量 `PS1` 控制，主要问题是完整文件路径展示导致前缀冗长。
- `\W`：仅显示最后一级目录（简短模式）
- `\w`：显示完整绝对路径（默认长模式）

## 方案A：临时生效（仅当前终端会话）
只展示虚拟环境名称 + 当前文件夹名称
```bash
export PS1="\[\033[32m\](python_5gtoolbox)\[\033[0m\] \W# "
效果示例：
(python_5gtoolbox) python_5gtoolbox‑1.0.0#
方案 B：永久生效（写入 bashrc 配置）
cat >> ~/.bashrc <<'EOF'
export PS1="\[\033[32m\](python_5gtoolbox)\[\033[0m\] \W# "
EOF
source ~/.bashrc
方案 C：通用自适应模板（推荐）
自动读取虚拟环境名称，无需硬编码
export PS1="(\$VIRTUAL_ENV_PROMPT) \W # "
恢复系统默认提示符
source /etc/profile
# 或者
unset PS1
极简美化推荐（彩色短提示符）
export PS1="\[\033[01;32m\]\$VIRTUAL_ENV\[\033[00m\] \[\033[01;34m\]\W\[\033[00m\] # "
