# OpenClaw 机械臂控制 Skill

- 作者：刘艺
- 邮箱：i_liuyi@cvte.com
- 修改时间：2026年3月26日

这个仓库用于分发 **OpenClaw** 的机械臂控制 Skill，面向希望通过 **OpenClaw** 调用 **Python SDK** 控制机械臂的用户。

仓库中的 `arm-control/pycarm` 以 **Git submodule** 方式管理：
- **pycarm** 仓库: [https://github.com/cvte-robotics/pycarm](https://github.com/cvte-robotics/pycarm)
- **OpenClaw Skill** 仓库: [https://github.com/cvte-robotics/openclaw](https://github.com/cvte-robotics/openclaw)

这意味着：
- 在源码仓库里，`pycarm` 是 `arm-control/pycarm` 下的 submodule
- 在用户安装后的 skill 目录里，实际使用的是 `pycarm/` 这一份 SDK
- 机械臂控制默认依赖 skill 自带的 `pycarm`，而不是用户 Home 目录中的外部缓存副本

## 1. 安装和配置 OpenClaw（Linux为例）

### 1.1 配置 API Key（MiniMax 为例）
**申请步骤：**
1. 访问 [https://platform.minimaxi.com/subscribe/coding-plan](https://platform.minimaxi.com/subscribe/coding-plan)
2. 注册/登录账号
3. 完成实名认证（需要身份证）
4. 订阅 Coding Plan
5. 进入 "API 管理" 页面
6. 创建 API Key 并复制

### 1.2 配置安装环境
**检查 Node.js 版本**  
打开你的终端（Terminal），输入：
```bash
node --version
```

**期望看到的结果：**  
v22.x.x

**判断标准：**
- ✅ 版本 >= v22：可以继续
- ❌ 版本 < v22：需要升级
- ❌ 提示 "command not found"：需要安装

**如果 Node.js 不符合要求**  
**Ubuntu/Debian：**
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**CentOS/RHEL/Fedora：**
```bash
sudo dnf install nodejs
```

### 1.3 安装 OpenClaw
**安装命令：**
```bash
npm install -g openclaw@latest
```

**验证安装成功：**
```bash
openclaw --version
```

**期望看到类似输出：**
```
2026.2.17
```

❌ **如果提示 "command not found"**

**可能原因：** npm 全局路径未加入系统 PATH

**解决方法：**
1. 重启终端
2. 如果还不行，检查 npm 全局路径：`npm prefix -g`
3. 把返回的路径加入 PATH 环境变量

### 1.4 运行配置向导
安装完成后，运行初始化向导：
```bash
openclaw onboard --install-daemon
```

启动向导后，通常会先看到：
```
? I understand this is powerful and inherently risky. Continue?
❯ Yes
  No
```
这里选 **Yes** 继续。

如果你之前装过 OpenClaw，还会看到：
```
Existing config detected
...
? Config handling
❯ Use existing values
  Update values
  Reset
```
**怎么选：**
- **首次安装：** 通常不会出现这一段，直接进入下一步。
- **复装/换模型：** 选 **Update values**（推荐，保留其余稳定配置）。
- **完全重来：** 选 **Reset**（谨慎）。

**向导第二步：选择模式（Onboarding mode）**  
向导会问你：
```
? Onboarding mode
❯ QuickStart - Minimal setup, get running fast
  Manual - Full control over all settings
```
选 **QuickStart**

**向导第三步：先选模型厂商（Provider）**  
向导会提示你选择模型提供商：
```
? Which model provider would you like to use?
  OpenAI
  Anthropic
  KIMI
❯ MiniMax
  GLM
  Other (custom endpoint)
```
选择**minimax** 

**向导第四步：再选鉴权方式（Auth method，容易漏）**  
在你选完厂商后，通常不会立刻要 Key，而是先进入该厂商的鉴权方式选择。

常见会看到类似：
```
? <Provider> auth method
  Coding Plan / OAuth
❯ API Key
```
**建议：** 选择 **API Key**

- **MiniMax：** 优先 MiniMax API key（CN）路径

**向导第五步：输入 API Key 并选模型**  
选择 MiniMax
```
? Enter your MiniMax API Key: [粘贴你的 Key]
? Select model: ❯ MiniMax-M2.7
```

**向导第六步：配置 Channel（先 Skip！）**  
在 QuickStart 路径下，向导会直接进入渠道单选列表：

```
? Select channel (QuickStart)
  ...
  Feishu/Lark (飞书)
  ...
❯ Skip for now (You can add channels later via `openclaw channels add`)
```

**建议：** 直接选 **Skip for now**。

**向导第七步：配置 Search Provider**  
```
Search provider
│  ○ Brave Search
│  ○ DuckDuckGo Search (experimental)
│  ○ Exa Search
│  ○ Firecrawl Search
│  ○ Gemini (Google Search)
│  ○ Grok (xAI)
│  ○ Kimi (Moonshot)
│  ○ Perplexity Search
│  ○ Tavily Search
│  ● Skip for now (Configure later with openclaw configure --section web)
```

**建议：** 直接选 **Skip for now**。

**向导第八步：配置 Skills**  
Channel 之后，向导会进入 Skills 检查与可选安装：

```
? Configure skills now? (recommended)
  Yes
❯ No
```

**建议：** 选 **NO**

**向导第九步：配置 Hooks**  
Skills 后会进入 Hooks 配置：

```
◆  Enable hooks?
│  ◻ Skip for now
│  ◻ 🚀 boot-md
│  ◻ 📎 bootstrap-extra-files
│  ◻ 📝 command-logger
│  ◻ 💾 session-memory
│  ◻ 🧠 self-improvement
```

**建议：** 空格 + enter 直接跳过

向导第十步：选择 Hatch 方式（关键）
在收尾阶段，向导会给你一个启动入口选择：

```
? How do you want to hatch your bot?
  Hatch in TUI (recommended)
❯ Open the Web UI
  Do this later
```

选择 **Open the Web UI**

### 1.5 首次对话和 Bootstrap（真正初始化）
**onboard** 完成后，建议先在 **TUI** 里完成第一轮 **bootstrap** 对话。

官方流程会把它当成"把 Agent 变成你的 Agent"的关键动作（源码里有 **Wake up, my friend!** 引导）。

这一步建议你主动讲清楚下面 **5 件事**：

1. **你是谁**：怎么称呼你、你的时区和工作语言。
2. **你要它扮演什么角色**：比如"我的技术写作助手"。
3. **你平时的工作场景**：常用工具、文件目录、沟通方式（飞书/邮件等）。
4. **你的偏好**：回答风格、长度、是否先给结论。
5. **你的边界**：哪些操作必须先确认、哪些内容不要碰。

这一步做得越清楚，后续它越像"**你自己的实例**"，而不是"一个通用聊天机器人"。

如果有疑问请参考下面两个链接：

 [OpenClaw官方安装指南](https://docs.openclaw.ai/zh-CN/start/getting-started) 

如果官方安装指南描述不够清晰，也可以参考此[教程](https://mp.weixin.qq.com/s/0Xq9XOfTjnQYwqXOVe9rZg?sessionid=1774408441&scene=231&clicktime=1774408629&enterid=1774408629&subscene=10000&ascene=3&fasttmpl_type=0&fasttmpl_fullversion=8184212-zh_CN-zip&fasttmpl_flag=0&realreporttime=1774408629739&devicetype=android-36&version=2800455b&nettype=ctnet&abtest_cookie=AAACAA==&lang=zh_CN&session_us=gh_d0f76b973041&countrycode=CN&exportkey=n_ChQIAhIQbThkYFnG2CTPmIDLxXM0FRLxAQIE97dBBAEAAAAAAHHkCVIG6FIAAAAOpnltbLcz9gKNyK89dVj0eVSdICOymYoHmG2nPozJiTJHZUcRw6J9tI4QtGV5/2AR86nprBdaPjYJTJzAU2GptXAYUCF78rb3mRr2hxKf3NB2QCzB83bw5iw+fvKEzl0/ktjIfqu3wW2RyRIFwqnPP6IajTOB5tEFKnWMkRYnPppZVQCn3/vhHFBrE6l3YSxsj980n+eyNoTWghzoK/KJycQ3VySizQei7iyMiQFeHJ/kwhVpXkJ0KqElqosjhk/t7H3CC2IClIFGqRdLJw9FonVodVsFYQV4cSA=&pass_ticket=5RNP3Cp4btkJqq+LcEQas8kvjdVuOH/pxZxI9+xlVzHt/O1AEURHuz5F+/Vvynho&wx_header=3&color_scheme=light)。

## 2. 在 OpenClaw 中安装 Skill

可以让 OpenClaw 根据 Git 仓库自动安装 Skill。推荐提示词：

> 请安装此 Git 仓库 [https://github.com/cvte-robotics/openclaw](https://github.com/cvte-robotics/openclaw) 中的 `arm-control` 这个 skill。安装完成后，检查已安装 skill 目录下的 `pycarm` 是否存在且可用；如果该目录保留了 git 元数据，请同步初始化并更新它；如果没有 git 元数据但目录缺失或内容不完整，请重新从 `https://github.com/cvte-robotics/pycarm.git` 拉取到该 skill 目录下；后续控制机械臂时统一从 skill 目录内的 `pycarm` 导入 `carm`。

安装完成后，Skill 通常位于：

```text
~/.openclaw/workspace/skills/arm-control/
```

## 3. pycarm 管理方式

### 仓库维护方式

在本仓库源码中，`pycarm` 位于：

```text
arm-control/pycarm
```

并由 Git submodule 管理。协作者在克隆主仓库后，建议执行：

```bash
git clone --recurse-submodules https://github.com/cvte-robotics/openclaw.git
```

如果仓库已经克隆完成，再执行：

```bash
git submodule update --init --recursive
```

### 已安装 Skill 的运行方式

OpenClaw 安装 skill 后，运行时默认使用已安装 skill 目录下的：

```text
pycarm/
```

如果安装流程没有自动带下 `pycarm` 内容，可以在已安装的 skill 目录中补齐或更新它。

## 4. 版本使用策略

### 默认方式：跟随当前 skill 仓库记录的 submodule 版本

这是最稳妥的默认方式。它表示当前 skill 文档和示例默认对应当前仓库记录的 `pycarm` 提交版本。

### 升级到 `pycarm` 最新主线

如果你希望在源码仓库中升级 submodule 指向：

```bash
git -C arm-control/pycarm pull
git add arm-control/pycarm
```

如果你希望在已安装的 skill 目录中刷新 `pycarm`，并且该目录保留了 git 元数据：

```bash
git -C pycarm pull
```

如果 `pycarm` 没有 git 元数据，则直接重新拉取：

```bash
git clone https://github.com/cvte-robotics/pycarm.git pycarm
```

### 固定到指定 tag 或 commit

如果你需要可复现环境，可以在 `pycarm` 目录中固定版本：

```bash
git -C arm-control/pycarm fetch --tags
git -C arm-control/pycarm checkout <tag-or-commit>
git add arm-control/pycarm
```

## 5. 使用 Skill

机械臂控制 Skill 允许通过 WebSocket 直连控制机械臂，包括连接、运动控制、夹爪控制和轨迹示教。

### 激活条件

当你提到以下关键词时，Skill 会被激活：
- 机械臂
- robot arm
- 控制机械臂
- arm control

### 使用前准备

1. 安装依赖：`pip install websocket-client`
2. 确认已安装 skill 目录中的 `pycarm` 可用
3. 控制机械臂前，先提供机械臂 IP 地址

### 快速开始

下面示例显式使用 OpenClaw 默认安装路径，并从该 skill 目录中的 `pycarm` 导入：

```python
import sys
from pathlib import Path

skill_root = Path.home() / ".openclaw" / "workspace" / "skills" / "arm-control"
pycarm_root = skill_root / "pycarm"
sys.path.insert(0, str(pycarm_root))

from carm import Carm

arm = Carm(addr="机械臂IP")
arm.set_ready()
arm.move_joint([0, 0, 0, 0, 0, 0])
arm.set_gripper(0.04)
arm.set_gripper(0.0)
```

### 注意事项

- 连接前需先确认机械臂 IP
- 默认以当前 skill 自带的 `pycarm` 为准
- 如果已安装的 `pycarm` 没有 git 元数据，升级方式应改为重新拉取，而不是 `git pull`
- 如果你升级了 `pycarm`，最好同步验证 skill 示例是否仍然适用
- 关节位置单位为弧度
- 夹爪位置单位为米，`0` 表示闭合，`0.08` 表示全开
- 笛卡尔位姿格式为 `[x, y, z, qw, qx, qy, qz]`
