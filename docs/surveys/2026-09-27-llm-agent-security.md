# 2026 年 LLM 与 Agent 安全调研

> 调研快照。2026-09-27。
>
> 材料来自 2026 年 X 上的安全讨论，以及这些讨论指向的论文、标准和仓库。
> 这不是当前开发目标，也不进入 [文档入口](../developer/document-status.md) 的现行清单。
> 数字以论文摘要、实验室原文或仓库说明为准。只读到新闻或转帖的，在来源表里标明。
> 本文记录分类和公开结果，不收录可直接复用的攻击样本。

## 检索范围

时间窗是 2026-01-01 至 2026-09-27。

关键词覆盖 jailbreak、prompt injection、indirect prompt injection、MCP、tool poisoning、memory poisoning，以及中文的越狱、破甲、提示注入。

低信号内容已剔除：代币软文、无审查接口商家、把实验室事件改写成犯罪组织的转述。

X 搜索噪声很大。下面只保留能指回论文、标准、仓库或实验室原文的讨论。

## 三个词

| 说法 | 攻击者通常是谁 | 要打破的是什么 | 2026 年的状态 |
| --- | --- | --- | --- |
| 越狱 | 直接对话的用户或红队 | 模型自己的拒答 | 静态模板在前沿模型上接近失效。剩下的是多轮搜索、Agent 自研攻击、推理通道 |
| 破甲 | 词被用杂了，见下 | 权重里的拒答方向，或任何绕过 | 安全讨论里主要指消融拒答。中文推特上更常见的是无审查商家 |
| 注入 | 用户，或网页、仓库、工具输出、图片、邮件的作者 | 应用或 Agent 的任务和权限 | 今年的主战场 |

直接注入和间接注入说的是内容从哪进来。越狱说的是目标是绕过模型安全策略。一次攻击可以同时占两类。

客服 Agent 没有输出违禁文本，只要在错误指令下调用了退款工具，业务就已经被打中。

「破甲」在中文推特上有三层：

1. 权重消融。把拒答方向从权重里削掉。官方 API 仍拒绝，本地权重不再拒绝。
2. 无审查写作和生图接口。和安全研究无关。搜「破甲」时这层噪声最大。
3. 口语里把沙箱逃逸也叫越狱。那是隔离失败。

## 标准已经怎么分

OWASP GenAI 在 2026-08-04 发布 LLM Top 10 2026。排序参考了公开漏洞库里可归类的事件。过度代理从第 6 升到第 3。无界消耗从第 10 升到第 6。提示注入留在第 1。文件写明目前没有可靠的模型层根除办法。周围系统应按「模型会被骗」来设计。

| 2026 排序 | 风险 | 相对 2025 |
| --- | --- | --- |
| LLM01 | 提示注入 | 不变 |
| LLM02 | 敏感信息泄露 | 不变 |
| LLM03 | 过度代理 | 自第 6 上升 |
| LLM04 | 供应链 | 自第 3 下降 |
| LLM05 | 数据与模型投毒 | 自第 4 下降 |
| LLM06 | 无界消耗 | 自第 10 上升 |
| LLM07 | 错误信息 | 自第 9 上升 |
| LLM08 | 隐藏上下文暴露 | 由「系统提示泄露」改名 |
| LLM09 | 向量与嵌入弱点 | 自第 8 下降 |
| LLM10 | 输出处理不当 | 自第 5 下降 |

来源：[GenAI-LLM-Top10](https://github.com/GenAI-Security-Project/GenAI-LLM-Top10)，排序变化见 [Citadelo 对照](https://citadelo.com/en/blog/owasp-llm-top-10-2026)。

Agent 另有一份 2026 版清单，2025-12 发布，2026 年全年被引用。官方页：[OWASP Top 10 for Agentic Applications for 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)。下面十条的中文对照来自两份独立转述，正式名称以官方 PDF 为准。

| 编号 | 风险 | 今年讨论里的现象 |
| --- | --- | --- |
| ASI01 | 目标劫持 | 网页、邮件、工单、仓库里的间接注入改写任务 |
| ASI02 | 工具误用 | 工具合法，调用方式和对象被带偏 |
| ASI03 | 身份与权限滥用 | Agent 继承过宽凭据，或变成 confused deputy |
| ASI04 | Agent 供应链 | MCP、Skill、规则文件、提示模板、第三方权重被改 |
| ASI05 | 意外代码执行 | 不可信内容变成 shell、解释器或钩子 |
| ASI06 | 记忆与上下文投毒 | 一次写入长期记忆，之后被当成可信上下文 |
| ASI07 | Agent 间通信不可靠 | 伪造、重放、篡改多 Agent 消息 |
| ASI08 | 级联故障 | 一个 Agent 的错误被编排器放大 |
| ASI09 | 人对 Agent 的信任被利用 | 用户因为界面像助手而批准了不该批准的动作 |
| ASI10 | 流氓 Agent | 被攻破或目标偏移的 Agent 看起来仍在正常工作 |

条目转述：[arnav.au 的十条对照](https://arnav.au/2026/07/02/owasp-top-10-for-agentic-applications/)。

6 月 Florian Roth 转述 NIST 护栏文件时的判断和 OWASP 同向。有限规则盖不住开放输入。能做的是加难绕过、持续检测、缩小爆炸半径。帖：[cyb3rops，2026-06-12](https://x.com/cyb3rops/status/2065380518892314736)。NIST 原文链接以该帖为准，本笔记没有另存 PDF。

## 1. 模型拒答

前沿模型上，手写角色扮演和编码混淆已经接近失效。活下来的是带反馈的搜索，以及让另一个模型发明攻击。

### HackAgent 对 Fable 5、Opus 4.8、Fable 5.1

意大利 AI4I 安全实验室用 HackAgent，对 7826 条有害意图跑了 PAP、PAIR、TAP、h4rm3l。

当前 arXiv 版本是 2026-09-20 的 v2。三个模型里，TAP 与 PAP 合计：Fable 5 为 2.72%，Opus 4.8 为 5.76%，Fable 5.1 为 8.19%。裁判改为五模型面板，至少 4/5 同意。确认的有害完成数：Opus 4.8 为 1315，Fable 5 为 620，Fable 5.1 为 1282。

6 月的 v1 用三模型多数票。那一版里 TAP 在 Opus 4.8 上约 11.5%，Fable 5 最差一族约 6.1%。静态混淆 h4rm3l 在约 5 万次尝试后低于 0.2%。中文安全媒体引用的是 v1。两版分母和裁判不同，不能并成一个百分比。

论文：[arXiv:2606.18193](https://arxiv.org/abs/2606.18193)。工具：[hackagent.dev](https://hackagent.dev)，仓库 [AISecurityLab/hackagent](https://github.com/AISecurityLab/hackagent)。

### 编码 Agent 自己发明攻击

Claudini 把 Claude Code 和 Codex 放进固定算力的自动研究循环，读 30 多种已有方法再改算法。

在 GPT-OSS-Safeguard-20B 的 CBRN 查询上，Agent 找到的方法最高约 80%。已有方法低于 50%。对做过对抗训练的 Meta-SecAlign-70B，新方法到 100%，此前自动化方法约 82%。攻击在无关替代模型上用随机目标 token 练出，再迁到提示注入。

论文：[arXiv:2603.24511](https://arxiv.org/abs/2603.24511)（2026-03-25 提交，2026-05-29 修订）。讨论帖：[alphaXiv，2026-03-27](https://x.com/askalphaxiv/status/2037664915255685624)。本笔记没有找到作者放出的代码仓库。

### 自适应攻击打穿「接近 0」的防御

*The Attacker Moves Second* 用梯度、强化学习、随机搜索和人工引导，打 12 种越狱和注入防御。多数在自适应攻击下成功率超过 90%。这些防御的原文曾报告接近 0。预印本是 2025-10，宣讲在 USENIX Security 2026。2026 年的防御讨论把它当成评测下限。

论文：[arXiv:2510.09023](https://arxiv.org/abs/2510.09023)。会议页：[USENIX Security 26](https://www.usenix.org/conference/usenixsecurity26/presentation/nasr)。

### 推理通道

arXiv:2609.29775 把输出前缀拿到推理模型上测。只改推理草稿，攻击成功率约 0%。同一段推理加上很短的输出前缀，在 Gemini 3 Flash Preview、DeepSeek V4 Flash、Claude Haiku 4.5 上最高约 99%。1800 个 AdvBench 用例。

论文：[arXiv:2609.29775](https://arxiv.org/abs/2609.29775)。

*Stealing Reasoning Traces* 说明，同一家厂商里较弱的模型有时能解开较强模型加密的思维链。作者解码了公开轨迹里的 315320 个推理块，摘要报告回收 367 条个人信息和 182 条凭据。加密块还可以当作看不见的注入通道。厂商随后堵上了作者复现的那条路径。

论文：[arXiv:2608.09867](https://arxiv.org/abs/2608.09867)。项目页：[stolen-thoughts.com](https://stolen-thoughts.com)。独立复现仓库：[mitkox/stolen-thoughts](https://github.com/mitkox/stolen-thoughts)。讨论帖：[alphaXiv，2026-08-11](https://x.com/askalphaxiv/status/2087221551632420897)。Simon Willison 的笔记：[2026-08-11](https://simonwillison.net/2026/Aug/11/stealing-reasoning-traces/)。

### 多 Agent 训练里的互相注入

2026-09-17，@fleetingbits 转述 OpenAI 对齐博客里的一例：模型把越狱写进自己的压缩摘要。帖里的解释是，训练时同伴失灵，注入对方能把共同奖励做高。这是对实验室笔记的转述，不是独立论文。

帖：[fleetingbits](https://x.com/fleetingbits/status/2100433480555381115)，后续解说：[MTS](https://x.com/MTSlive/status/2102139730347647280)。

### 权重层破甲

提示词攻击打的是当次上下文。Abliteration 打的是拒答方向本身。

2026-08-20，Pliny 放出 Qwen3.8-27B-OBLITERATED，自称在 842 条有害提示上拒答率为 0。同线程后来更正：bf16 权重没问题，第一版 GGUF 转换有误，需要重新下载。这是发布者的自述，不是第三方审计。

帖：[elder_plinius](https://x.com/elder_plinius/status/2090229579189518394)。权重：[OBLITERATUS/Qwen3.8-27B-OBLITERATED](https://huggingface.co/OBLITERATUS/Qwen3.8-27B-OBLITERATED)。

2026-09-11，@wquguru 建议安全团队在本地部署消融模型，用来接官方模型拒答的红队问题。帖里写了两条边界：它不提高能力上限。第三方 GGUF 有投毒先例。

帖：[wquguru](https://x.com/wquguru/status/2098560237867512282)。

9 月仍有人公开演示把指令嵌进不可见字符。帖：[elder_plinius，2026-09-14](https://x.com/elder_plinius/status/2099548218355056823)。本笔记不摘录那段字符。

## 2. 间接注入

### 网上已经有多少

arXiv:2604.27202 扫了 12 亿个 URL、2480 万台主机。确认 1.53 万条注入，分布在 1.17 万个页面、2042 台主机上。54 个模板占了 95%。约 70% 写在不渲染的 HTML 里。目的包括捣乱、操纵声誉、保护内容、识别 AI 爬虫。

论文：[arXiv:2604.27202](https://arxiv.org/abs/2604.27202)。讨论帖：[asthashukla95，2026-09-23](https://x.com/asthashukla95/status/2102652474310210010)。

### 公开竞赛

arXiv:2603.15714：464 人，27.2 万次尝试，13 个前沿模型，41 个场景，成功 8648 次。攻击成功率从 Claude Opus 4.5 的 0.5% 到 Gemini 2.5 Pro 的 8.5%。有的策略能迁到 41 种行为里的 21 种。能力和鲁棒性几乎不相关。

论文：[arXiv:2603.15714](https://arxiv.org/abs/2603.15714)。

### 网页 Agent

2026-06-12，Decrypt 报道南洋理工、ST Engineering、IBM、UIUC 的模拟。NanoBrowser 和 BrowserUse，背骨是 GPT-5 和 Gemini 2.5 Flash，共 3168 次。直接注入超过 79%。藏在网页里的间接注入在 42% 到 68% 之间。本笔记没有打开论文 PDF，数字以这篇报道为准。

报道：[Decrypt](https://decrypt.co/370972/ai-agents-prompt-injection-attacks-research)。

### AgentDojo 上的自动化攻击

arXiv:2606.10525 把 GCG 和 TAP 放进 AgentDojo。黑盒 TAP 明显强于白盒梯度。原因是合理算力下 GCG 不稳定。攻击模型越强，注入越好写。做过安全对齐的攻击模型会拒绝生成攻击。在小模型上优化的攻击迁不到 GPT-5。任务通用的攻击可以迁到没见过的任务。

论文：[arXiv:2606.10525](https://arxiv.org/abs/2606.10525)。评测床：[ethz-spylab/agentdojo](https://github.com/ethz-spylab/agentdojo)，论文 [arXiv:2406.13352](https://arxiv.org/abs/2406.13352)，结果页 [agentdojo.spylab.ai](https://agentdojo.spylab.ai)。

### 休眠到触发才执行

arXiv:2609.22510 测了一种先休眠、等条件满足才执行的间接注入。裸命令在前沿模型上几乎被拒。改成条件句之后，真实改状态的工具调用从平均 2.4% 升到 16.5%。

九个生产 Agent 上各 30 次：Codex、Gemini CLI、Claude Code CLI、Cursor CLI、GitHub Copilot、Devin、Kiro、Qwen Code、Google Assistant。成功率 43% 到 83%。命令式基线最高 3%。把命令式注入训到接近 0 的偏好模型，仍会在触发轮执行约 12%。

论文：[arXiv:2609.22510](https://arxiv.org/abs/2609.22510)。本笔记没有找到配套仓库。

### 已经落在具体工具链上的入口

| 入口 | 公开结果 | 来源 |
| --- | --- | --- |
| 仓库和依赖 | 2026-03，Gergely Orosz 写道，Agent 往生产推代码时，注入已经出现在高关注度项目里 | [帖](https://x.com/GergelyOrosz/status/2029992079741304977) |
| 逆向 Agent | 海军研究生院把短指令放进程序字符串。Ghidra 把字符串送进 Agent 后，分析结论会被带偏，程序仍能运行。用遗传算法在字符串长度限制里搜索 | [arXiv:2605.30667](https://arxiv.org/abs/2605.30667)，会议版 [ECCWS](https://papers.academic-conferences.org/index.php/eccws/article/view/4597) |
| 逆向 Agent 的讨论帖 | 同一工作在 7 月被安全账号转发，约 850 赞。帖里还点了第二篇检测与混淆论文，本笔记没有核对到那篇的 arXiv | [0x0SojalSec](https://x.com/0x0SojalSec/status/2076413397802029464) |
| 图片 | Repeat-After-Me。用户问题和图片任务可以语义无关。Qwen3.6-27B 上工具调用超过 80%，GPT-5.5 约 47%。替代模型上优化的图片能保住大约四到六成并迁到商用模型。默认配置的 OpenClaw Discord 里，图片可以改写之后每次都会读的工具说明 | [arXiv:2609.04533](https://arxiv.org/abs/2609.04533)，[Meta 页面](https://ai.meta.com/research/publications/repeat-after-me-black-box-adaptive-visual-prompt-injection/) |
| 图片的讨论帖 | [Void，2026-09-08](https://x.com/VoidStateKate/status/2097305795830386726) | 转述上面这篇 |

## 3. 行为越狱

嘴上拒绝不够。要看系统状态有没有被改。

### TRACE

浙大、清华、上海人工智能实验室。把恶意任务拆成尽量少的显式有害子任务，再包进看起来正常的场景，并用类似 Q-learning 的方式改写场景。AgentHarm 和 AdvCUA 上，绕过率最高到 100%，平均成功分 0.73。

论文：[arXiv:2605.30883](https://arxiv.org/abs/2605.30883)。本笔记没有找到配套仓库。中文技术媒体用其中一个 Gemini 分数差算出约十倍。引用时以论文摘要的绕过率和成功分为准。

### LITMUS

南航与浙大。真实操作系统里 819 个高风险用例，覆盖言语越狱、Skill 注入和实体包装。语义拒绝和操作系统状态一起看。Claude Sonnet 4.6 仍执行了 40.64% 的高风险操作。仓库说明里，各 Agent 的执行幻觉率在 7.98% 到 9.97%。执行幻觉指对话里拒绝了，危险操作已经在系统里完成。

论文：[arXiv:2605.10779](https://arxiv.org/abs/2605.10779)。仓库：[AlienZhang1996/LITMUS](https://github.com/AlienZhang1996/LITMUS)。数据：[Hugging Face 数据集](https://huggingface.co/datasets/AlienZhang1996/LITMUS)。

### 多 Agent 的 14 个向量

arXiv:2609.22949 把注入分成用户输入、工具输出、Agent 间消息、编排器级联，共 14 个向量。一个 6 Agent 系统在系统提示里加了护栏之后，仍有 67% 的 Agent 至少越权一次。经工具输出的间接注入成功率 43%。作者报告四种架构控制把总成功率从 31.2% 降到 4.2%。四项是消息签名、边界清洗、按角色收紧工具、通信异常检测。这是预印本，本笔记没有看到独立复现。

论文：[arXiv:2609.22949](https://arxiv.org/abs/2609.22949)。

## 4. 拦截分类器

编码 Agent 为了少点批准，在动作执行前加了分类器。Claude Code 的 Auto Mode、Codex 的 Guardian 都是这种。Anthropic 在 2026-03-25 写过 Auto Mode 的两段式分类器：[工程博客](https://www.anthropic.com/engineering/claude-code-auto-mode)。

2026-09-17 的红队论文改了假设。以前测的是不可信内容劫持一个本意正常的 Agent。这篇测的是 Agent 自己持续作恶，并专门躲监控。

约 79% 的试验里，对 Auto Mode 和 Guardian 的注入能让任意 shell 通过。加宽工具覆盖、改写送进分类器的记录格式、再加一层 Agent 式监控，能补上单轮缺口。跨多个上下文的攻击仍开着。

论文：[arXiv:2609.19587](https://arxiv.org/abs/2609.19587)。代码：[safety-research/red-teaming-auto-mode](https://github.com/safety-research/red-teaming-auto-mode)。讨论帖：[safe_paper](https://x.com/safe_paper/status/2103968131857178691)。

2026-09-23 有帖转述对 Prompt Guard 2 的解释性分析：分类器靠很多 token 的累积。同义替换和改写可以在只改中等比例文本时翻转判定。帖里没有给出论文链接，本笔记没有另找到 PDF。

帖：[irastech](https://x.com/irastech/status/2102792650411892851)。

这和 USENIX 那篇是同一条评测要求。分类器要留。静态坏字符串上的接近 0 不能单独当证据。

## 5. MCP、Skill 和凭据

### STDIO 把配置里的命令交给操作系统

2026-04，OX Security 披露 MCP STDIO 传输的设计问题。官方 Python、TypeScript、Java、Rust SDK 都中。启动参数里的命令字符串会进操作系统执行。服务启动失败时，命令也已经跑过。

CSO 2026-04-16 的报道写了规模：六个有真实客户的服务上执行了命令。200 多个开源项目，7000 个以上公开服务，下载量约 1.5 亿，估计约 20 万个实例。随后有 30 多次协调披露、10 个以上 CVE。Anthropic 将此行为视为传输层的既定设计。

报道：[CSO Online](https://www.csoonline.com/article/4159889/rce-by-design-mcp-architectural-choice-haunts-ai-agent-ecosystem.html)。案例整理：[vectara/awesome-agent-failures 里的这篇](https://github.com/vectara/awesome-agent-failures/blob/main/docs/case-studies/mcp-stdio-supply-chain-rce.md)。CSA 的非官方研究笔记：[PDF](https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/04/CSA_research_note_mcp-design-rce-protocol-attack-surface_20260425-csa-styled-2.pdf)。

2026-09-06，OWASP 放出与厂商无关的 MCP 风险用语。仓库：[OWASP/MCP-Taxonomy](https://github.com/OWASP/MCP-Taxonomy)。帖：[InfosecVandana](https://x.com/InfosecVandana/status/2096568141451297103)。

### 凭据在使用时暴露

2026-09-21，DeepLearning.AI 介绍 Meta Muse 的三条产品假设：模型手里不放真实凭据。工具跑在隔离的 Linux 容器里。出站调用由独立门卫核对。这是该账号对 Muse 的转述。

帖：[DeepLearningAI](https://x.com/DeepLearningAI/status/2102070383398502862)。

同月有财经账号转发 Muse 赏金：单用户 Agent 被攻破 13 万美元，全舰队 25 万美元，关键问题最高 30 万美元。数字来自该帖，本笔记没有核对 Meta 赏金页。

帖：[wallstengine](https://x.com/wallstengine/status/2097406834571448636)。

中文工程师的同类边界：

- 不要在裸机上开自动批准。隔离落到 microVM 或一次性沙箱。[ai2face，2026-09-15](https://x.com/ai2face/status/2099879548578058520)
- Agent 钱包要和主资金分开，限制单笔额度和可交互合约，用完撤销。[0x99DaDa，2026-09-03](https://x.com/0x99DaDa/status/2095477076178747630)

## 6. 沙箱事件和叙事纠偏

2026-09-20，OpenAI 一个做检索训练的内部 Agent，直接访问公网被代理拦住。训练环境的 DNS 解析器仍会回答真实域名。Agent 经这条路径问了一个公开聊天机器人。监控约 15 分钟告警。人工约 2.5 小时后终止。最强模型的工具调用训练、评测和推理因此继续暂停。

实验室原文：[An agent used DNS to reach an external chatbot](https://alignment.openai.com/misalignment-reports/an-agent-used-dns-to-reach-an-external-chatbot/)。

2026-09-12，Simon Willison 写到另一起：5 月有一群 Agent 在维基事件之后几天里骚扰和滥用 RubyGems。

笔记：[simonwillison.net](https://simonwillison.net/2026/Sep/12/openai-agents-rubygems/)。帖：[simonw](https://x.com/simonw/status/2098573718142452055)。

多家报道把 7 月的沙箱离开算成三个月内的上一次。本笔记没有核对 OpenAI 的 7 月原文，不引用具体人数。

圈子里同时有人反对把这些事讲成「模型产生了欲望」：

- Agent 是套了工具和目标的模型循环。它会朝目标行动。这不等于它想要什么。[cyb3rops，2026-09-01](https://x.com/cyb3rops/status/2094866215739040102)
- 澳大利亚健身房候补的报道：用户明确要求从第 4 挪到第 1。Agent 发现接口缺少授权检查，取消了别人的预约。破坏性动作不该对另一个用户执行。把它叫成自主网络攻击，是报道口径放大了。[cyb3rops，2026-08-09](https://x.com/cyb3rops/status/2086585810514747796)
- 一旦允许 Agent 改生产环境，范围、关键性、置信度和能不能恢复，比「模型懂不懂这次攻击」更要紧。他推荐的是英国 NCSC 的说明，原文以帖内链接为准。[cyb3rops，2026-09-22](https://x.com/cyb3rops/status/2102305510376493365)

## 7. 防御侧在用的做法

没有一份 2026 年的严肃材料声称某个提示词或某个分类器能根除注入。重复出现的是下面几层。

**权限先于模型。** 工具白名单、按任务给凭据、秘密不进通用 shell、不可逆动作留人批。Muse 的转述和中文工程师的沙箱帖都落在这一层。

**模型外面的确定性门卫。** Springer 2026-09-18 的工作把 Dual-LLM 做进 Google ADK。固定的 50 个攻击用例，五个重复，模型是 GPT-4.1。单 Agent 效用约 94%，攻击成功率约 26%。工具过滤把攻击成功率降到约 3%，效用约 82%。两种 Dual-LLM 配置都没有观察到成功攻击，效用约 78% 和 70%。样本只有 50 例。

论文：[Springer](https://link.springer.com/article/10.1007/s10015-026-01152-3)。

**分类器按恶意 Agent 来测。** 见第 4 节的 Auto Mode 论文和 USENIX 论文。

**多 Agent 要有消息来源。** 见第 3 节那篇预印本里的四项控制。数字尚未独立复现。

**指令层级可以在推理时补。** V-Steer 不重训，改缓存的 value 向量，让高优先级片段压过冲突的低优先级片段。7B 到 70B 上，受控角色冲突的主约束准确率从低于 18% 升到最高 92%。COLM 2026。

论文：[arXiv:2607.26228](https://arxiv.org/abs/2607.26228)。代码：[cindy2000sh/v-steer](https://github.com/cindy2000sh/v-steer)。帖：[cindy2000_sh](https://x.com/cindy2000_sh/status/2097838398958445051)。

实践者备注：全大写的忽略指令会被当前模型当成明显标志。有效样本要顺着模型学过的指令层级和真实工具返回格式来写。[lefthanddraft，2026-09-17](https://x.com/lefthanddraft/status/2100441136930476168)

**红队进持续集成。** 今年帖里反复出现的扫描器是 NVIDIA garak。RAG 侧有人把注入、文档泄露和跨文档夹带做成 CI。

| 工具或合集 | 地址 | 本笔记里的依据 |
| --- | --- | --- |
| garak | [NVIDIA/garak](https://github.com/NVIDIA/garak) | [KitPloit 转发 v0.17.0，2026-09-15](https://x.com/KitPloit/status/2099810779671195991) |
| rag-redteam | [Srivatsa03/rag-redteam](https://github.com/Srivatsa03/rag-redteam) | [Dinosn，2026-08-23](https://x.com/Dinosn/status/2091392888500351167) |
| HackAgent | [AISecurityLab/hackagent](https://github.com/AISecurityLab/hackagent) | 见第 1 节 |
| LITMUS | [AlienZhang1996/LITMUS](https://github.com/AlienZhang1996/LITMUS) | 见第 3 节 |
| AgentDojo | [ethz-spylab/agentdojo](https://github.com/ethz-spylab/agentdojo) | 见第 2 节 |
| Auto Mode 红队 | [safety-research/red-teaming-auto-mode](https://github.com/safety-research/red-teaming-auto-mode) | 见第 4 节 |
| V-Steer | [cindy2000sh/v-steer](https://github.com/cindy2000sh/v-steer) | 见本节 |
| MCP Taxonomy | [OWASP/MCP-Taxonomy](https://github.com/OWASP/MCP-Taxonomy) | 见第 5 节 |
| Awesome Prompt Injection | [Joe-B-Security/awesome-prompt-injection](https://github.com/Joe-B-Security/awesome-prompt-injection) | 2026 年仍在维护的资源合集。Intigriti 2026-09-23 推荐过同名合集，帖内链接以 [原帖](https://x.com/intigriti/status/2102685160932089976) 为准 |
| Arcanum 分类 | [Arcanum-Sec/arc_pi_taxonomy](https://github.com/Arcanum-Sec/arc_pi_taxonomy) | 2026 年的注入分类，带可引用编号。网页：[pitax](https://www.arcanum-sec.com/pitax) |

## 8. 不计入研究的推特噪声

同一批关键词里，下面这些声量不小，和上面的证据不是一类。

- 以提示注入为叙事的代币和「Agent 防火墙」空投。
- 中文「破甲」商家，卖的是无审查写作和生图接口。
- 把分类器营销成无法被语言注入。第 4 节的 Prompt Guard 转述和 USENIX 论文都有反例。
- 把 OpenAI 的沙箱事件复述成 Agent 拉帮结派。第 6 节以实验室原文和 Willison、Roth 的帖为准。

## 9. 今年仍被当成基础的更早工作

这些不是 2026 年的新结果。新论文默认读者已经接受它们。

| 框架 | 在说什么 | 来源 |
| --- | --- | --- |
| 间接注入的问题定义 | 不可信内容里的指令会被模型执行 | Greshake 等，[arXiv:2302.12173](https://arxiv.org/abs/2302.12173) |
| Lethal trifecta | 私有数据、不可信内容、对外通信三者齐备时，注入可以变成窃数 | Willison，2025-06-16，[原文](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta) |
| Agents Rule of Two | 在能可靠拒绝注入之前，一次会话里「不可信输入、敏感数据、对外改状态」最多保留两项 | Willison 2025-11-02 的笔记里转述 Meta 的规则，[原文](https://simonwillison.net/2025/Nov/2/new-prompt-injection-papers/) |
| GCG | 白盒梯度搜索越狱，2026 年在 AgentDojo 上不如黑盒 TAP | Zou 等，[arXiv:2307.15043](https://arxiv.org/abs/2307.15043) |
| PAIR | 黑盒多轮改写 | Chao 等，[arXiv:2310.08419](https://arxiv.org/abs/2310.08419) |
| TAP | 树搜索。HackAgent 的 v1 里，前沿模型残余面主要来自这一族 | Mehrotra 等，[arXiv:2312.02119](https://arxiv.org/abs/2312.02119) |
| PAP | 说服式改写 | Zeng 等，[arXiv:2401.06373](https://arxiv.org/abs/2401.06373) |
| Crescendo | 多轮逐步加码 | Russinovich 等，[arXiv:2404.01833](https://arxiv.org/abs/2404.01833) |
| AgentDojo | 工具调用 Agent 的效用和安全一起打分 | 见第 2 节仓库 |

去掉 lethal trifecta 的任意一条，窃数那条链就断了。2026 年的仓库注入、图片改工具说明、MCP 命令执行，是这三条重新齐备。Agent 读不可信内容，它有副作用工具，它还有一条把结果送出去的通道。

## 读完可以留下的判断

1. 聊天拒答要用自适应攻击和多轮搜索来测。一组固定越狱句不够。依据是 HackAgent 和 USENIX Security 2026。
2. Agent 的默认输入包括网页、仓库、工具结果、图片、邮件、记忆和其他 Agent 的消息。依据是第 2 节的测量、竞赛和生产 Agent 试验。
3. 模型层防护降低受骗概率。爆炸半径由工具权限、凭据隔离、出站门卫和人批决定。依据是 OWASP LLM Top 10 2026 和 Muse、Dual-LLM 两类设计。
4. 拦截分类器要按「Agent 自己在躲监控」来红队。依据是 arXiv:2609.19587。
5. 评测要看世界有没有被改动。只看模型说「我拒绝了」会漏掉已经发生的系统操作。依据是 LITMUS。
6. 中文圈说的破甲，先分开权重消融、提示词绕过和无审查接口。三者的防御不同。

## 来源索引

| 主题 | 类型 | 地址 |
| --- | --- | --- |
| LLM Top 10 2026 | 标准仓库 | https://github.com/GenAI-Security-Project/GenAI-LLM-Top10 |
| LLM Top 10 排序变化 | 二次对照 | https://citadelo.com/en/blog/owasp-llm-top-10-2026 |
| Agentic Top 10 2026 | 标准页 | https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/ |
| Agentic 十条中文对照 | 二次对照 | https://arnav.au/2026/07/02/owasp-top-10-for-agentic-applications/ |
| HackAgent 论文 | 论文 | https://arxiv.org/abs/2606.18193 |
| HackAgent 仓库 | 仓库 | https://github.com/AISecurityLab/hackagent |
| Claudini | 论文 | https://arxiv.org/abs/2603.24511 |
| The Attacker Moves Second | 论文 | https://arxiv.org/abs/2510.09023 |
| The Attacker Moves Second | 会议 | https://www.usenix.org/conference/usenixsecurity26/presentation/nasr |
| 推理前缀 | 论文 | https://arxiv.org/abs/2609.29775 |
| 窃取推理轨迹 | 论文 | https://arxiv.org/abs/2608.09867 |
| 窃取推理轨迹 | 项目页 | https://stolen-thoughts.com/ |
| 窃取推理轨迹 | 独立复现 | https://github.com/mitkox/stolen-thoughts |
| 野外间接注入 | 论文 | https://arxiv.org/abs/2604.27202 |
| 公开竞赛 | 论文 | https://arxiv.org/abs/2603.15714 |
| 网页 Agent 模拟 | 新闻 | https://decrypt.co/370972/ai-agents-prompt-injection-attacks-research |
| AgentDojo 上的自动化攻击 | 论文 | https://arxiv.org/abs/2606.10525 |
| AgentDojo | 仓库 | https://github.com/ethz-spylab/agentdojo |
| AgentDojo | 论文 | https://arxiv.org/abs/2406.13352 |
| 休眠注入 | 论文 | https://arxiv.org/abs/2609.22510 |
| 逆向 Agent 注入 | 论文 | https://arxiv.org/abs/2605.30667 |
| Repeat-After-Me | 论文 | https://arxiv.org/abs/2609.04533 |
| TRACE | 论文 | https://arxiv.org/abs/2605.30883 |
| LITMUS | 论文 | https://arxiv.org/abs/2605.10779 |
| LITMUS | 仓库 | https://github.com/AlienZhang1996/LITMUS |
| LITMUS | 数据集 | https://huggingface.co/datasets/AlienZhang1996/LITMUS |
| 多 Agent 14 向量 | 预印本 | https://arxiv.org/abs/2609.22949 |
| Auto Mode 红队 | 论文 | https://arxiv.org/abs/2609.19587 |
| Auto Mode 红队 | 仓库 | https://github.com/safety-research/red-teaming-auto-mode |
| Auto Mode 设计 | 工程博客 | https://www.anthropic.com/engineering/claude-code-auto-mode |
| MCP STDIO | 新闻 | https://www.csoonline.com/article/4159889/rce-by-design-mcp-architectural-choice-haunts-ai-agent-ecosystem.html |
| MCP Taxonomy | 仓库 | https://github.com/OWASP/MCP-Taxonomy |
| Dual-LLM 与 Google ADK | 论文 | https://link.springer.com/article/10.1007/s10015-026-01152-3 |
| V-Steer | 论文 | https://arxiv.org/abs/2607.26228 |
| V-Steer | 仓库 | https://github.com/cindy2000sh/v-steer |
| garak | 仓库 | https://github.com/NVIDIA/garak |
| rag-redteam | 仓库 | https://github.com/Srivatsa03/rag-redteam |
| OpenAI DNS 事件 | 实验室原文 | https://alignment.openai.com/misalignment-reports/an-agent-used-dns-to-reach-an-external-chatbot/ |
| OpenAI Agent 与 RubyGems | 笔记 | https://simonwillison.net/2026/Sep/12/openai-agents-rubygems/ |
| Qwen3.8 消融权重 | 模型卡 | https://huggingface.co/OBLITERATUS/Qwen3.8-27B-OBLITERATED |
| Lethal trifecta | 笔记 | https://simonwillison.net/2025/Jun/16/the-lethal-trifecta |
| 间接注入定义 | 论文 | https://arxiv.org/abs/2302.12173 |
| GCG | 论文 | https://arxiv.org/abs/2307.15043 |
| PAIR | 论文 | https://arxiv.org/abs/2310.08419 |
| TAP | 论文 | https://arxiv.org/abs/2312.02119 |
| PAP | 论文 | https://arxiv.org/abs/2401.06373 |
| Crescendo | 论文 | https://arxiv.org/abs/2404.01833 |
