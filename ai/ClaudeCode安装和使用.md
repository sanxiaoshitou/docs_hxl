# claude code 安装和使用

## 一、claude code安装(winds)
```shell
winget install Anthropic.ClaudeCode
```
查询版本命令
```shell
claude -v
```
注:需要vpn外网,要不然很慢

![安装](../web/img/claude1.png)
## 二、切换模型
1、下载切换工具
模型切换辅助工具cc-switch,安装包地址
https://github.com/farion1231/cc-switch/releases

2、目前主流模型
已接入Kimi K2.6为例 https://platform.kimi.com/
![安装](../web/img/claude2.png)

## 三、Claude Code 指令

| 指令 / 快捷键 | 作用 | 典型场景 |
|--------------|------|----------|
| `/compact` | 压缩对话上下文，保留核心摘要 | 对话过长 token 超限时 |
| `/clear` | 清空所有对话历史，全新开始 | 切换到完全不同的任务时 |
| `claude -c` | 启动时恢复上一次对话 | 次日继续昨天未完成的工作 |
| `Ctrl+B` | 将当前任务挂到后台运行 | 让 AI 后台编译，前台讨论下一步 |
| `ESC×2` (`/rewind`) | 回退或总结（回滚 AI 操作） | AI 改错了代码，快速撤销 |
| `↓ / /tasks` | 管理后台任务（查看/停止） | 检查后台编译/测试是否完成 |
| `Ctrl+T` | 显示/隐藏任务列表面板 | 查看 Claude 创建的任务进度 |
| `Alt+V` | 粘贴图像（Windows） | 截图粘贴给 Claude 分析 UI/报错 |
| `/memory` | 打开并编辑 CLAUDE.md | 修改项目规则或个人偏好 |
| `/init` | 自动生成 CLAUDE.md 初稿 | 新项目首次接入 Claude Code |
| `/hooks` | 配置工具钩子（自动化触发） | 写文件后自动 prettier 格式化 |
| `\+Enter` | 输入框内换行（不发送消息） | 输入多行代码或多段需求描述时 |
| `Shift+Tab` | 循环切换三种工作模式 | 从 Plan Mode 切到 Accept Edits 执行 |


