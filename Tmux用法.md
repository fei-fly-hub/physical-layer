# Tmux 完整简明使用手册

## 一、基础概念

- **会话 \(session\)**：一组后台终端，断开 SSH 不会终止程序

- **窗口 \(window\)**：会话里的标签页（类似浏览器 tab）

- **面板 \(pane\)**：一个窗口内分割成多个小终端

快捷键前缀：**先按 ****`Ctrl + b`****，松开，再按下后续按键**

## 二、会话操作（最常用）

```bash
# 新建会话（自动编号）
tmux new

# 新建命名会话（推荐）
tmux new -s dev

# 查看全部会话列表
tmux ls

# 重新接入会话
tmux a -t dev
tmux attach -t dev

# 脱离会话（后台保留运行程序）
Ctrl + b ，松开后按 d

# 删除指定会话
tmux kill-session -t dev

# 删除除当前之外所有会话
tmux kill-session -a

# 关闭全部tmux服务
tmux kill-server

# 会话重命名
tmux rename-session -t dev work
```

## 三、窗口操作（Tab 标签）

前缀：`Ctrl + b`

|按键|功能|
|---|---|
|c|新建窗口|
|w|列出所有窗口，方向键选择切换|
|0\~9|直接跳转到编号窗口|
|n|下一个窗口|
|p|上一个窗口|
|,|重命名当前窗口|
|\&|关闭当前窗口（会杀掉里面所有程序）|

## 四、面板分割（一个窗口分多屏）

前缀：`Ctrl + b`

|按键|功能|
|---|---|
|%|垂直左右分割面板|
|"|水平上下分割面板|
|方向键|切换光标到对应面板|
|z|面板全屏放大 / 恢复原来大小|
|x|关闭当前面板|
|Space|切换不同分割布局|
|q|短暂显示每个面板编号|

## 五、命令模式（冒号指令）

1. 按 `Ctrl + b`，松开，按 `:`

2. 输入指令回车
示例：

```Plain Text
new -s test          # 创建会话
kill-session -t test # 删除会话
split-window         # 水平分屏
```

## 六、复制滚动模式（查看历史日志）

1. `Ctrl + b` 松开，按 `[` 进入滚动模式

2. 上下方向键 / PageUp PageDown 翻页

3. 空格开始选中文本，回车复制

4. `]` 粘贴复制的内容

5. `ESC` 退出滚动模式

## 七、实用日常工作流模板

```bash
# 1. 创建开发会话
tmux new -s 5gdev

# 2. 分屏操作
# Ctrl+b % 左右分屏
# Ctrl+b " 上下分屏

# 3. 临时断开ssh，后台继续跑仿真
# Ctrl+b d

# 4. 下次登录重新连上继续看日志
tmux a -t 5gdev
```

## 八、区分两个容易混淆操作

1. `Ctrl+b d`：**脱离会话，程序继续后台运行**

2. `exit` / `Ctrl+d`：**关闭终端面板，程序终止**

## 九、最小常用速查表

```bash
tmux new -s name      # 创建命名会话
tmux ls               # 查看会话
tmux a -t name        # 接入会话
tmux kill-session -t name # 删除会话
Ctrl+b d              # 后台挂起会话
Ctrl+b c              # 新建窗口
Ctrl+b %              # 左右分屏
Ctrl+b "              # 上下分屏
Ctrl+b z              # 面板全屏切换
```
