---
{"dg-publish":true,"permalink":"/jet-note/growing-up/knowledge-graph/","dg-note-properties":{}}
---


一、AI大模型基础

理论知识

从提示工程到 RAG
-Prompt Engineering 与 Context Engineering：提示工程基础、高级技巧、上下文工程核心、长上下文管理

-RAG 与私有知识库：核心价值、构建三部曲、向量数据库、工作流

Agent：从可控性到自主反思
-控制幻觉、提升可控性：幻觉成因、RAG/Tool-use 校验、Function Calling + JSON 约束、Human-in-the-Loop

-思维链构建自主反思 Agent：CoT 原理、Self-Reflection、ReAct、自我修正

多模态前沿：从 Agent 构建到视频 AIGC
-多模态 Agent 构建要点：MLLM 能力基座（GPT5/Gemini）、视觉感知封装为工具、统一表征与分发、VQA

-视频检索及生成方案：视频检索、视频理解、视频生成（Sora/Kling 扩散模型）、时序一致性等挑战

实操基础

AI 大模型基本原理及 API 使用
-基本原理：分析式 vs 生成式 AI、从 GPT-1 到 GPT-5、LLM 训练、Temperature 与 Top P、AI Chat 超能力

-大模型 API 使用：系统/用户提示词、Token 限制、CASE（情感分析/天气 Function/表格提取/运维处置，均用 Qwen+DashScope）

AI 编程 - 从入门到精通
-Cursor 编程：Cursor Rules、主要功能

-Cursor 实战：CASE（多张 Excel 报表处理、疫情实时监控大屏）

-Trae 与 CodeBuddy：Trae 使用、CodeBuddy 使用、多 Excel 报表处理

二、AI框架及工具平台

开发框架理论知识 + 案例详解 + 实操

LangChain：多任务应用开发
-LangChain 多任务应用开发

Models, Prompts, Memory, Indexes, Chains, Agents

LangChain 中的 tools（serpapi, llm-math）

LangChain 中的 Memory

LCEL 构建任务链

-LangChain 开发实操详解

CASE：动手搭建本地知识智能客服（理解 ReAct）

CASE：工具链组合设计（LangChain Agent）

CASE：搭建故障诊断 Agent（LangChain Agent）

CASE：工具链组合设计（LCEL）

AI框架设计与选型
-自研框架设计思路

核心组件抽象：模块化、插件化与可扩展的框架结构

数据流与控制流：Agent、Tools、Models 的标准化交互管线

状态管理与性能考量：异步处理、缓存机制与日志监控

-优秀开源开发框架详解

LlamaIndex 深度解析

AutoGen 多智能体框架

框架选型对比：LangChain vs. LlamaIndex vs. AutoGen

HuggingFace 生态实战：从模型应用到高效微调
-HuggingFace 模型库的使用

核心组件（Transformers, Datasets, Tokenizers）

Pipelines API：零代码/少代码调用模型

Model Hub 与 Dataset Hub：搜索、下载与版本控制

-使用 HuggingFace 做模型微调

Trainer API：标准化训练与评估接口

PEFT 高效微调：LoRA 与 QLoRA 原理与代码实战

TRLx：使用强化学习（RLHF/PPO）对齐语言模型

神经网络基础与 TensorFlow 实战
-神经网络基础

神经网络结构、激活函数、损失函数

反向传播、梯度下降、优化方法（SGD、Adam）

使用 numpy 搭建神经网络

-TensorFlow 实战

计算图与会话管理

分布式训练与模型并行

TensorFlow Serving 部署与推理

使用 Keras 构建简单神经网络 / 二手车价格预测

PyTorch 与视觉检测
-PyTorch 核心概念：张量与自动求导、动态图与静态图

-PyTorch 分布式训练：多 GPU 训练、PyTorch Lightning

-图像识别技术与缺陷检测：传统模型、视觉检测方法、缺陷检测方案、卷积网络可视化

-视觉检测模型 YOLO

从 YOLOv1 到 YOLOv12

ultralytics：基于 PyTorch 的视觉检测工具

Project：钢铁表面缺陷检测

三、AI研发工程师工作新范式

AI Coding 带来的范式变革

大厂优秀工程师使用 AI Coding 的最新方法与经验
-用确定性驾驭概率性：告别"感觉式编程"、"Spec Coding"三铁律、渐进式复杂度管理

-构建你的 AI 流水线：L3 AI Coding、多智能体协作模式、自动化 CI/CD 实战

-构建团队的"知识飞轮"：Rules+Spec+Skills 三位一体、知识飞轮、解决"代码考古"难题

-构建"审查-优化"闭环：AI 自我审查与优化、人的最终决策权、从"超级个体"到"超级团队"

大型软件项目的 AI 开发与 AI 重构
-从"代码实现者"到"系统架构师"：演进式设计、上下文工程、AI 作为"活文档"

-AI 重构战略：战略框架、“暴力拆解"与"精准缝合”、渐进式现代化（桑树模式）

-构建"生成-验证-修正"质量飞轮：自我反思与对抗性测试、人的最终决策权、工程文化

AI Coding 中的团队重新分工与新协作模式
-从"职能专家"到"AI 指挥官"：Spec 架构师、AI 训练师、质量验证师

-流程重构：Spec 与 Issue 协同、团队"生产性资本"、人机协同"精准制导"

-组织进化：分形化团队架构、新角色职责与技能

-文化重塑：建立 AI 辅助的团队学习文化

四、AI应用技术

RAG 理论知识 + 案例详解 + 实操

Embeddings 和向量数据库
-Embeddings 模型及向量化：什么是 Embedding、Word Embedding、余弦相似度、模型选择、MTEB 榜单、向量维度影响、“俄罗斯套娃”

-向量数据库和向量检索：FAISS / Elasticsearch / Milvus / Pinecone 特点、与传统数据库区别、数据导入、性能比较

RAG 技术与应用
-RAG 原理及流程：三种开发范式、RAG 如何增强、核心原理与流程、NativeRAG、CASE（DeepSeek + Faiss 本地知识库）

-Query 改写与知识库处理：Query 改写/联网搜索、知识库处理、问题生成、对话沉淀、健康度检查、版本管理

RAG 多模态数据处理
-图文数据：PDF 解析、Word 解析、网页解析、Qwen-Agent 中的 RAG

-多媒体数据：图像多重表征、视频处理流程、ASR、GraphRAG（全局/局部搜索）

RAG 调优
-混合检索：为什么需要、BM25 + 向量检索、多路召回、重排序（Rerank）

-调试与评估：数据准备、检索阶段、生成阶段、效果量化、持续优化

部分场景可取代 RAG 的技术
-Long-Context LLM：上下文即知识、全量知识注入、成本与性能权衡

-Agentic Search：从 Just-In-Time 到 Just-In-Case、思考-行动-观察循环、ReAct 与 Plan-and-Solve

-LLM Wiki：RAG 致命缺陷、LLM Wiki 范式、自进化机制

Agent 理论知识 + 案例详解 + 实操

Function Calling 与 MCP
-Function Calling 工具调用：原理、与 MCP 区别、Qwen3 天气调用、Qwen-Agent、数据库查询

-MCP 与 A2A 应用：MCP 核心概念（Host/Client/Server）、使用场景、CASE（旅游攻略/网页抓取/Bing 搜索/搭建 MCP 服务）、A2A 与 MCP 关系

Agent 的自主规划与工具开发
-思考规划反思能力：设计范式（反应式/深思熟虑式/混合式）、CoT 与 ReAct

-自主编写代码开发工具：Tool Use 基础、code_interpreter、Text-to-SQL Copilot

Agent 能力优化与效果评估
-用使用数据提升能力：显式/隐式反馈、用反馈做 RAG 或微调

-效果评估：大海捞针、多跳推理（Multi-Hop）、业务指标

Harness Engineering
-四层架构：记忆层、执行层、编排层、反馈层

-核心 memory 机制（OpenClaw / Hermes / Claude Code）：SQLite 历史、每日 memory/YYYY-MM-DD.md、四层颗粒度

-环境与工具：工具集设计原则、执行环境构建

-沙箱、文件系统、权限：Git Worktree 隔离、四层防护、权限精细化

模型训练与微调 理论知识 + 案例详解 + 实操

LLM 微调原理
-微调原理：高效微调方法、LoRA 数学原理、核心假设、矩阵分解、SVD

-数据处理与显存评估：数据准备、质量与数量、显存计算、LoRA 显存优化、微调后评估

高质量微调数据工程与评估
-数据收集、清洗、标准：收集策略、清洗流程、标注规范、SFT vs RLHF 数据差异

-数据质量与结果关系：Garbage In Garbage Out、自动化指标、Benchmark、人工评估

LLM 模型蒸馏与微调实操
-模型蒸馏：核心思想、目的与价值、经典方法、LLM 时代挑战

-微调与蒸馏实操：unsloth 框架、教师-学生模型选择、任务蒸馏、效果评估

视觉与多模态模型
-多模态与视觉识别：CV 三大任务、VLM 行业应用、视频理解 SOTA、MinerU

-训练 YOLO 目标检测模型：核心原理、数据准备与标注、训练自定义模型、性能评估

五、项目实战

企业知识库（企业 RAG 大赛冠军项目）
-企业 RAG 大赛：搭建 RAG 知识库

RAG 冠军方案（多路由 + 动态知识库）、比赛任务说明、基础 RAG 流程

解析模块、Docling 优化、表格序列化

内容提取（ingestion）、检索（Retrieval）、LLM 重排序、父页面检索、整合检索器

增强（Augmentation）、生成（Generation）

思维链、结构化输出、指令细化（Instruction Refinement）

提示词创建、Prompt.py 实现、RAG 系统调参

-搭建自己的 RAG 系统

选择 LLM 和 Embedding 模型、MinerU 使用

更新中文知识库、问题清单、开放式问题 Prompt 设置

搭建前端页面（如 streamlit）

OpenManus 开发实战
-深度解析 OpenManus 框架：项目导论、手稿生成生命周期、核心模块（Orchestrator / Agents / Memory / Tools）、提示词工程

-构建自己的 AI 写作助手：本地运行、Agents 角色定制、集成企业知识库 RAG、中文写作流

AI质检
AI 在工业质检的价值

技术选型对决：YOLO vs. Qwen-VL

数据集分析（EDA）、环境准备、数据工程

YOLO 训练与调优、模型评估与缺陷分析

Qwen-VL 多模态探索性测试、"零样本"缺陷检测

结果分析与 YOLO vs Qwen-VL 终极对决

搭建 Hermes Agent 中的长期记忆和自进化能力
-心跳唤醒，自主收集记忆：心跳机制代码、自适应频率调整、记忆价值评估算法

-整理成 4 层颗粒度记忆：Tier2→Tier1→Tier0 逐级摘要、Tier2→Tier3 相关性扩展

-Embedding 保存到 LanceDB：存储格式设计、LanceDB 部署、混合检索策略

-用户新任务唤醒记忆：意图识别与记忆路由、上下文动态构建、实战演示

实现 Hermes 中的多 Agent 协作、主 Agent 调度
-主 Agent 核心能力：意图理解、任务拆解、智能分发、最小闭环上下文

-子 Agent 分工、注册与执行机制：标准结构、注册机制、执行流程

-Agent 间通信 + 任务流闭环：通信机制、任务依赖、结果聚合、异常处理

-全局闭环系统：状态持久化设计、自进化前置底座

短剧视频逐帧换脸的显卡资源分配及排队系统
-换脸流程：视频拆帧→逐帧换脸→脸部检测对齐→换脸→帧组装、任务管理

-换脸任务及算力分配：RocketMQ 解耦、队列长度控制、生命周期与超时重试、原始视频缓存

-脸部预处理：常见问题、预处理方式（人脸识别检测、闭嘴、光头化）

LLM Wiki
结构化、统一口径、双向链接、去重的 Wiki 文档

三层架构：Raw（原始数据湖）→ Wiki（编译输出层）→ Schema（控制协议）

增量更新、基于 Git 的版本控制、从 RAG 到"编译器模式"

知识库索引体系：总目录/主题索引/标签索引、关键词检索 + 目录路由

在华为昇腾显卡上部署 DeepSeek V4 并连通本地 Claude Code
-DeepSeek V4 深度解析与性能基准：V4 架构、版本选型、能力边界测试

-华为昇腾（Ascend）生态与 CANN 架构：国产算力底座、CANN 软件栈、环境准备

-在昇腾 NPU 上部署 DeepSeek V4：模型适配与量化、推理服务化、API 兼容性配置

-Claude Code 本地化集成与 Agent 工作流打通：本地环境、无缝连接、Agent 能力实测

六、模型部署及高并发

理论知识 + 案例详解 + 实操

企业级 AI 部署：从硬件选型到框架选择
-硬件选型、规划与优化

GPU 选型：H100 / A100 / L40S / 4090 对比

CPU 与内存配比：对 Tokenization、请求调度、并发的影响

网络基础设施：RoCE (RDMA) vs. TCP/IP

服务器与集群规划

-部署框架特点

Ollama：易用性、本地化部署

vLLM：高吞吐推理（PagedAttention）

SGLang：复杂控制流高性能、与 vLLM 差异

选型对比

AI 服务核心：高并发原理与性能监控调优
-高并发原理：KV Cache 瓶颈、PagedAttention 核心思想、vLLM 实现、Continuous Batching、动态插入/退出

-性能监测及调优：关键指标、监控工具栈、vLLM 调优、瓶颈定位

SGLang 深度优化：Radix 缓存与复杂任务极致吞吐
-缓存优化及 Radix Tree：原理、RadixAttention、运行时高级抽象（Frontend/Backend）、SGLang IR

-复杂任务极致延迟及吞吐：Token Healing、复杂控制流优化、多租户与智能调度、与 vLLM 差异

附：专项面试辅导课

RAG 相关面试题

大模型应用开发的三种主要模式？

文档分块有哪些策略？为什么选这个策略？

画一下 RAG 系统架构图并解释关键步骤

用了哪个 Embedding 模型？为什么？

RAG 效果差时从哪几方面调试？

用户问题模糊或依赖上一轮对话时怎么优化？

只用向量检索吗？缺点？什么是混合检索？

召回 20 条文档，怎么确保喂给 LLM 的是最好的 3 条？

上线后怎么维护和迭代知识库？

如何评估一个 RAG 系统的好坏？

Agent 相关面试题

一分钟讲清楚 Agent 的定义

如何处理 Agent 的幻觉问题？

Agent 的"状态"如何管理？

如何平衡 Agent 的自主性与可控性？

介绍你最复杂的 Agent 项目

为什么用 LangGraph？如何处理 Text-to-SQL？

如何搭建 RAG Agent 实现本地知识库（如 PDF）问答？

如何定义自定义工具？多文件上下文长度限制怎么解决？

如何评估检索效果 / 收集用户使用数据？

Agent 设计哲学

开发框架相关面试题

LangChain 解决了什么问题？六大核心组件职责与交互？

如何用 LangChain 构建 RAG 系统？Memory 机制如何实现？

LangChain vs. LlamaIndex 核心定位差异？何时选谁？

如何看待 AutoGen 等多智能体框架？

从零设计 LLM 应用框架如何抽象组件？如何设计可插拔 Tool/Plugin 系统？

HuggingFace Pipelines API 优势与局限？Tokenizers 作用？

全参数微调缺点？什么是 PEFT？

PyTorch 动态图 vs 静态图？TF 2.x Keras vs TF 1.x Session/Graph？

LLM 时代 TensorFlow vs PyTorch 优劣与生态？

模型训练与微调相关面试题

预训练和微调的理解？全参数微调？为什么需要 PEFT？

详细讲 LoRA 原理；微调技术选型？

微调数据来源（开源/业务/合成）？高质量微调数据标准？

SFT 数据格式如何构建？如何评估微调后效果？

自动化评估（BLEU/ROUGE）vs 人工评估？知道哪些 Benchmark？

为什么用模型蒸馏？核心思想？如何评估蒸馏效果？

什么是多模态模型？CV 三大任务 vs 多模态 VQA 区别？

YOLO 数据集如何标注准备？工业质检 YOLO vs Qwen-VL 各自优缺点？
————————————————
版权声明：本文为CSDN博主「DeepSeek-R2」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/m0_74942241/article/details/161865551