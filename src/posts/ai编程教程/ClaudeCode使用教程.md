## 使用ClaudeCode 教程

## 1. 安装

### 1.1 选其中一种即可 终端命令行安装：
  + `irm https://claude.ai/install.ps1 | iex`
  + ![终端下载安装演示](/images/ai编程教程/ClaudeCode使用教程_1.png)

### 1.2 选其中一种即可 node环境安装
  + https://nodejs.cn/en/download
  + 建议：22版本 22.20.0LTS 
  + 打开终端输入： `npm install -g @anthropic-ai/claude-code`  
  + 输入：`claude -v` 查看版本号
  + 如图所示：![alt text](/images/ai编程教程/ClaudeCode使用教程_2.png)

## 2. 配置.claude目录
  + 打开目录 找到你的用户名 `C:\Users\用户名\.claude`
  + 打开 `settings.json` 没有就新建一个文件
  + 如图所示：
  + 配置如下：![alt text](/images/ai编程教程/ClaudeCode使用教程_3.png)
```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "api_key秘钥",
    "ANTHROPIC_BASE_URL": "请求url",
    "API_TIMEOUT_MS": "3000000"
  },
  "permissions": {
    "defaultMode": "bypassPermissions"
  },
  "alwaysThinkingEnabled": true
}
```

## 4. 使用 随便新建一个文件夹  终端进入到该文件夹 输入命令 `claude` 
  + 提问：`使用html开发一个飞机大战游戏`
  + 按下回车 
  + 如图所示： ![alt text](/images/ai编程教程/ClaudeCode使用教程_4.png)

## 5. 跑完任务自行进行使用游览器打开html查看效果 不满意则继续提问
  + ![alt text](/images/ai编程教程/ClaudeCode使用教程_5.png)
  + ![alt text](/images/ai编程教程/ClaudeCode使用教程_6.png)