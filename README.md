# 2026最全免费AI API盘点！每月白嫖16亿Token，零成本实现大模型自由！

* 1. 想要自己动手搭建 AI 应用、测试大模型，却总是被高昂的 API 计费账单或者 429（请求超限）错误劝退？很多折腾过 sub2api 或 newapi 这类传统中转站的玩家都知道，手动去维护不同平台的渠道连通性有多头疼，而且单独一家的免费额度根本不够用。

* 2. 这个项目堪称“白嫖党的终极武器”，它能把从各家平台申请到的免费 API 额度，全部聚合成一个“无限 Token 池”（完美兼容 OpenAI 格式）。不仅支持自动切换和负载均衡，每月整合下来的免费 Token 总量甚至能高达 **16 亿**！

* 3.  今天，整理了市面上最核心的免费 API 获取渠道，还要手把手教你在 Linux 服务器上部署这套系统，彻底实现算力自由。

## 服务器部署

把上面的 Key 准备好后，我们直接在 Linux 服务器（以 Debian/Ubuntu 为例）上把这个聚合系统跑起来。项目基于 Node.js，自带 SQLite 数据库，部署非常轻量。

### 第一步：环境准备与基础安装

### 零基础前置准备（安装 Node.js 与 Git）

如果你手里是一台刚买的纯净服务器（以最常用的 **Ubuntu / Debian** 系统为例），在正式开始之前，我们需要先给它装好必备的运行环境：**Git**（用来拉取代码）和 **Node.js**（用来运行项目）。

**1. 更新系统并安装 Git：** 首先，我们把系统原有的软件包列表更新一下，并顺手装上 Git。在终端中复制并回车执行：

Bash

```bash
sudo apt update
sudo apt install git curl -y

# 检查 Git 是否安装成功（会输出 git version x.x.x）
git --version
```

**2. 安装 Node.js (核心：安装 v20+ 最新版)：** Linux 系统默认源自带的 Node.js 版本通常太老，跑不起来新项目。我们需要通过 NodeSource 官方源来安装推荐的 20.x 版本。依次执行以下两行命令：

Bash

```bash
# 1. 下载并配置 Node.js v20 的官方源
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 2. 执行一键安装
sudo apt install -y nodejs
```

**3. 验证安装结果：** 安装完成后，跑一下这两行命令，确认版本号是否达标：

Bash

```bash
# 检查 node 版本，如果输出 v20.x.x，说明大功告成！
node -v

# 顺便检查 npm（Node 包管理器）版本
npm -v
```

之后确保你的服务器安装了 Node.js (推荐 v20+ 版本) 和 Git。

接下来克隆项目与拉取依赖

Bash

```bash
# 将项目克隆到你习惯的目录，例如 /opt 
cd /opt
git clone https://github.com/tashfeenahmed/freellmapi.git

# 进入目录并安装依赖 (会自动配置好本地数据库)
cd freellmapi
npm install 
cp .env.example .env
echo "ENCRYPTION_KEY=$(node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")" >> .env
npm run build
# 启动服务和面板
npm run dev
```

### 第二步：配置 Systemctl 后台守护进程

为了防止关掉 SSH 终端后服务断开，我们需要把它挂载为系统后台服务，实现 24 小时稳定运行及开机自启。

**1. 创建服务配置文件：**

Bash

```bash
sudo nano /etc/systemd/system/freellmapi.service
```

**2. 填入以下配置：**

_(注意：请确保_ `WorkingDirectory` _是你实际克隆代码的路径)_

Ini, TOML

```bash
[Unit]
Description=FreeLLMAPI Proxy Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/freellmapi
# 若使用 nvm 管理 Node，请将下方路径替换为 npm 的绝对路径
ExecStart=/usr/bin/npm run dev
Restart=always
RestartSec=5
# 默认面板端口为 3001
Environment=PORT=3001

[Install]
WantedBy=multi-user.target
```

**3. 启动并设置开机自启：**

Bash

```bash
sudo systemctl daemon-reload
sudo systemctl start freellmapi
sudo systemctl enable freellmapi
sudo systemctl status freellmapi # 查看是否显示绿色的 active (running)
```

大功告成！现在浏览器访问 `http://你的服务器IP:3001`，进入管理后台填入你所有的 Key。系统会自动为你生成唯一的令牌，并开启**降级路由 (Fallback Chain)**：一旦某个渠道遇到限流或宕机，自动无缝切换到下一个可用节点，你的终端业务永不断线！

### 更新步骤

后续更新只需三步，无需重配环境：

Bash

```bash
sudo systemctl stop freellmapi
cd /opt/freellmapi
git pull origin main && npm install
sudo systemctl start freellmapi
```

## 全网最全免费 API 渠道盘点

### 1. 核心“正规军” (大厂免费额度)

这 9个平台是稳定的主力军，建议全部注册拿下：

- **Google AI Studio (Gemini)**：免费额度天花板，支持超长上下文，极高频率调用。👉 [申请官网](https://aistudio.google.com/)
    
- **Groq**：采用自研 LPU 芯片，全网推理速度最快，极低延迟。👉 [申请官网](https://console.groq.com/)
    
- **GitHub Models**：程序员专属羊毛，有 GitHub 账号即可免费调用 GPT-4o 等顶级模型。👉 [申请官网](https://github.com/marketplace/models)
    
- **OpenRouter**：最强聚合平台的“Free Models”专区，常有限免惊喜。👉 [申请官网](https://openrouter.ai/)
    
- **Mistral AI**：欧洲之光，提供优质的免费 Experiment 试用计划。👉 [申请官网](https://console.mistral.ai/)
    
- **Together AI**：最强开源云算力，注册赠送额度，且部分小参数模型完全免费。👉 [申请官网](https://api.together.ai/)
    
- **NVIDIA NIM**：英伟达官方免费算力，跑分极高，适合高强度逻辑测试。👉 [申请官网](https://build.nvidia.com/)
    
- **Cohere**：RAG 与文本专家，开发者永久免费试用 Key。👉 [申请官网](https://dashboard.cohere.com/)
    
- **Hugging Face Inference**：开源社区的大本营，海量模型免部署直接调用。👉 [申请官网](https://huggingface.co/settings/tokens)
    

- **Cerebras**：全网推理速度天花板，专攻 Llama 3 系列极速推理，每秒能吐出上千个 Token，体验极其丝滑。
    
    - **申请官网**：[https://cloud.cerebras.ai/](https://cloud.cerebras.ai/)
        
- **SambaNova**：同样是堆算力的狂魔，提供了极高配的开源大模型免费调用额度。
    
    - **申请官网**：[https://cloud.sambanova.ai/](https://cloud.sambanova.ai/)
        

- **Cloudflare Workers AI**：白嫖 Cloudflare 遍布全球的边缘计算节点。需要你有 CF 账号，在控制台的“Workers & Pages -> AI”板块生成 API Token。每天都有免费调用额度。
    
    - **申请官网**：[https://dash.cloudflare.com/](https://dash.cloudflare.com/)
        
- **Zhipu AI (智谱)**：国产大模型的绝对第一梯队（GLM 系列）。对新用户极其大方，注册就送海量 Token，而且它家最新的小参数模型（如 GLM-4-Flash）经常开放完全免费的调用。
    
    - **申请官网**：[https://open.bigmodel.cn/](https://open.bigmodel.cn/)
        
- ollama cloud [申请地址](https://signin.ollama.com/?client_id=client_01JX0QMHD43PFFCCNXH82A6K8B&redirect_uri=https%3A%2F%2Follama.com%2Fauth%2Fcallback&authorization_session_id=01KSP4GBHRNEDTA58YR6XMZ3PD)


## 申请的 API key 填写对应的 Add a provider key 添加完API后然后在Configured providers点击Remove测试：绿色通过正常；红色API key不可用。
<img width="1420" height="880" alt="image" src="https://github.com/user-attachments/assets/83c01564-6091-4c12-afc2-c1f7e0fe40c9" />

## 接入HERMES
<img width="973" height="482" alt="6db92d48-b7f4-4097-bd7f-dddf67174b88" src="https://github.com/user-attachments/assets/77ff3d18-0ddb-4685-ad05-ad99c85a5af5" />
<img width="744" height="278" alt="d875ea62-29eb-4b9b-b47e-c7bc66d21a1e" src="https://github.com/user-attachments/assets/aed89d4e-d43f-434c-b5f2-ccab09d77365" />
<img width="979" height="481" alt="b4a30c5c-aece-4eb3-b6cc-ba7446312cf2" src="https://github.com/user-attachments/assets/48cd4b7d-8dbf-4f65-8a8f-5b79955e2c53" />
<img width="979" height="478" alt="da3d3797-e044-4dfe-a548-50f7d3f02522" src="https://github.com/user-attachments/assets/b2ec0f66-c2cd-4542-867e-2358e83322db" />
<img width="979" height="481" alt="2a811539-e2d0-46a6-b75e-0865cc966bc9" src="https://github.com/user-attachments/assets/d44705d9-e122-4019-a4e7-25c40b5205f3" />
<img width="946" height="455" alt="b43ede8d-9553-4b51-b44b-dd54b85fe63e" src="https://github.com/user-attachments/assets/a9c44434-6f31-4aa7-98c1-e5e3a521cde0" />










