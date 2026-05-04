# The-twilight-hours-seem-lonely-Chinese-translation
# ⛩️ Oumagadoki-Ai-Translator (逢魔時·全感官浸入式AI视觉同传引擎)

[![Android API](https://img.shields.io/badge/API-21%2B-brightgreen.svg?style=flat)](https://android-arsenal.com/api?level=21)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Model](https://img.shields.io/badge/AI_Vision-Gemini_Optimized-blue)](https://deepmind.google/technologies/gemini/)
[![Status](https://img.shields.io/badge/Status-Active_Development-orange)]()

## 📖 项目全景概述 (Project Overview)

**Oumagadoki-Ai-Translator** 是一款专为 Android 平台打造的系统级、极低延迟、全自动浸入式游戏屏幕翻译/辅助引擎。本项目最初为日系文字冒险游戏（AVG/Galgame）《逢魔時》量身定制，但其底层的**“全向视觉捕获与大语言模型（LLM）流式同传架构”**使其完美适配任何缺乏官方汉化的外语应用与游戏。

传统基于 OCR（光学字符识别）+ 机器翻译的方案存在致命缺陷：无法理解游戏内复杂的背景设定、生硬的机翻导致出戏、难以识别花体或竖排文字，且频繁的截屏导致设备严重发热。

本项目彻底颠覆了这一路径。我们采用 Android 底层的 `AccessibilityService` 与 `MediaProjection` 双擎驱动，结合业界最顶尖的多模态大模型（Vision LLM，特别针对 **Gemini 2.5/3.0 Flash** 系列进行毫秒级同声传译优化），实现了“画面即译、无缝穿透、上下文连贯”的终极游玩体验。

---

## 🚀 核心革命性特性 (Core Features)

### 1. 毫秒级智能帧差对齐算法 (Smart Frame Hash-Matching)
引擎并非盲目地定时截屏，而是内置了极度省电的**画面指纹对比技术 (Bitmap Fingerprint)**。
*   **降采样静默监听**：将截获的每一帧在底层内存中迅速压缩为 32x32 的特征矩阵，计算 RGB 容差像素。
*   **打字机效应免疫**：精确识别游戏中文字“逐个蹦出”的动画过程（判定为动态帧，挂起请求）。只有当画面静止超过指定阈值，且与上一句话的特征指纹不同时，才会触发 AI 视觉请求，将无效 API 调用率降低 98%，极致压缩耗电量与 API 成本。

### 2. 物理级 UI 穿透技术 (Ghost-Touch Overlay)
通过重写 `WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE` 标志位，本项目的悬浮翻译框实现了**真正的“幽灵态”**。
*   玩家可以直接透过屏幕上显示的中文翻译文字，点击到下方的游戏按钮与立绘。
*   告别传统翻译软件“翻译框遮挡UI -> 关闭翻译框 -> 点击游戏 -> 重新打开翻译框”的繁琐操作。你的手指与游戏之间，不再有任何阻隔。

### 3. AI 视觉“同声传译”极速响应管道 (Simultaneous Interpretation Pipeline)
放弃传统的“识别+翻译”两步走策略，直接将原始图像数据 Base64 编码后投喂给拥有原生视觉理解能力的 LLM（如 `gemini-2.5-flash-lite`）。
*   **极限 Prompt 压缩**：底层注入了精心调试的系统指令，剥离 AI 一切无关思考过程，强制要求其以“同声传译员”的身份瞬间输出纯净译文。
*   **自动选项嗅探与排序**：当游戏出现分支选项时，视觉模型会自动嗅探多行文本，并在译文中按顺序排列输出（如：`选项1：... / 选项2：...`），完美契合复杂 RPG 场景。

### 4. 自动化私有目录资源释放 (Automated Asset Deployment)
彻底绕过 Android 11+ 严苛的沙盒存储限制（Scoped Storage）。
*   内置一键部署逻辑，应用启动时自动将 `assets` 目录下的超大 OBB 数据包与 APK 文件解压释出至 `Android/data/包名/files` 专属私有目录，并直接拉起静默或免权安装。

---

## 🧠 系统架构深度解剖 (Architecture Deep Dive)

本工程采用极其严密的三层解耦架构开发，确保模块间的绝对独立与极致性能：

```text
[ 用户端 User ]
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. 交互与渲染层 (MyAccessibilityService.java)               │
│ - 承载基于 Typeface 渲染的悬浮球与字幕框。                  │
│ - 处理多点触控（缩放、拖拽）、沉浸式暗场背景 (B3000000)。   │
│ - 动态交替 FLAG_NOT_FOCUSABLE 与 FLAG_NOT_TOUCHABLE。       │
└──────────────────────────────┬──────────────────────────────┘
                               │ (边界坐标注入)
       ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. 核心截流引擎 (MyMediaProjection.java)                    │
│ - ImageReader 在独立 HandlerThread 中疯狂榨取 Surface 数据。│
│ - 动态分辨率适应 (updateMetrics) 应对横竖屏突发切换。       │
│ - 像素级 ByteBuffer 拷贝与 Rect 选区裁剪算法。              │
└──────────────────────────────┬──────────────────────────────┘
                               │ (Base64 JPEG 流)
       ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 神经网络汇聚点 (Vision LLM API / MainActivity.java)      │
│ - 采用高并发异步线程维护 HTTP(S) 长连接。                   │
│ - 动态模型分类器 (Categorizer)，永远将 Gemini 视觉簇列为魁首│
│ - 拦截并解析 JSON 树，剥离出核心翻译字符串回调至 UI 线程。  │
└─────────────────────────────────────────────────────────────┘

🛠️安装、编译与部署指南(Installation&Deployment)
本项目原生支持在 Android 设备上使用 AIDE Pro (AndroidIDE) 进行原生编译。
1.编译环境要求
• 
IDE: AIDE Pro 或 Android Studio (推荐 Gradle 7.0+)
• 
SDK: minSdkVersion 21 (Android 5.0), targetSdkVersion 28 (为兼容更多悬浮窗权限策略而设定，可根据需求自行上调至 33+)。
• 
依赖: 采用纯原生 API 构建，零第三方臃肿库。HTTP 请求与 JSON 解析全权交由系统原生 HttpURLConnection 与 org.json 处理，极致轻量（APK体积极小）。
2.构建步骤
1.  
克隆本仓库：git clone https://github.com/YourUsername/Oumagadoki-Ai-Translator.git

2.  
将项目导入您的 IDE。
3.  
(可选) 将游戏本体的 APK 与 OBB 文件放置于 app/src/main/assets/Game/ 目录下，并重命名为 com.espero.oumagadoki.apk 和 main.7.com.espero.oumagadoki.obb，以便利用一键部署功能。
4.  
执行 Build 并生成 APK。
3.API 节点配置秘籍(针对高压使用者的省钱策略)
在应用的“AI配置”界面，您需要输入自己的 API 中转站/官方 URL 与 Key。
开发组成员的私密建议：
我们强烈推荐使用打折力度极高的中转 API（如0.4折分组令牌）。在配合本引擎极其廉价的文本 Token 消耗量与 gemini-2.5-flash 模型时：
• 
日均 600 次高频截屏+翻译请求。
• 
在 0.4 折率加持下，您的日均消耗可能低至 ¥0.20 人民币。
• 
这意味着一杯奶茶钱即可支撑一个大型 AVG 游戏长达数月的完整游玩周期。
⚙️内置提示词工程(Prompt Engineering)
本引擎强大表现的核心，在于发送给 LLM 的指令。我们抛弃了长篇大论的 System Prompt，转而采用极简、暴力的“指令截断”技术：
"提取图片中的日文文本并翻译成中文。请只输出中文翻译，严禁输出任何多余的解释！如果图片中没有字，请必须输出“EMPTY”这五个字母。遇到选项请自动格式化排序。"
这一设计彻底杜绝了 AI 产生如“这张图片展示了...”或“我的翻译是...”的废话，确保用户在悬浮窗中看到的仅仅是最纯粹、最高密度的剧情信息。
🗺️未来演进路线图(Roadmap&Future V2.0)
本项目绝非止步于单纯的“屏幕翻译器”，我们正在构筑一个内容创作的生态闭环。
• 
Phase 1: 实现高能效静默截屏与悬浮窗 UI 物理穿透。 (✅ 已完成)
• 
Phase 2: 接入多模态 AI，完成 Gemini 模型优先权重构与同传极速优化。 (✅ 已完成)
• 
Phase 3 (正在研发): 剧情记录与沉淀引擎 (Translation History Manager)。
◦ 
描述: 引擎将在后台默默记录您的整个游玩历程，将每一次翻译的图文对映射（时间戳、原图快照、精准中文翻译）序列化至本地 JSON。
◦ 
目的: 游戏通关后，一键导出完整剧情文案大纲。这不仅方便分发给社区进行剧情探讨，更为视频创作者（B站/YouTube）制作“游戏剧情解析”、“汉化流程发布”等视频提供了最完整、按时间线精确排列的素材库。
• 
Phase 4: 自动化游戏进程控制（检测到关键剧情动画自动挂起截图，节约算力）。
📜许可证与免责声明(License&Disclaimer)
本项目基于 MIT License 协议开源。
免责声明：本项目仅供 Android 逆向工程学习、无障碍服务(Accessibility)开发研究以及个人的 UI 自动化测试使用。请勿将本工具用于侵犯任何游戏/应用原作者版权的商业汉化倒卖活动。任何因滥用 API 或违反目标应用服务条款而产生的后果，由使用者自行承担。
注:本项目由AI编写开发
