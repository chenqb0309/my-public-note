---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c01-ai/","dg-note-properties":{}}
---


# 學習資源：AI 框架及工具平台

> 課程 Area 1，對應圖譜：自研框架設計、LangChain/LlamaIndex/AutoGen、HuggingFace 生態、PyTorch/TF 視覺

---

## 核心框架選型與對比

### LangChain — 業界最廣泛的生態
https://python.langchain.com/docs/introduction/

- **六大核心元件**：
  - **Models**：LLM 包裝層（OpenAI、Anthropic、本地模型）
  - **Prompts**：模板管理、Few-shot 組織、輸出解析
  - **Memory**：對話歷史、狀態保存
  - **Indexes**：文檔載入、分塊、向量化
  - **Chains**：多步驟任務串接（LCEL 語法）
  - **Agents**：Tool-use + 決策循環（ReAct）
- **必看**：LangChain 101 官方教學、LCEL（LangChain Expression Language）
- **課程 CASE 實戰**：
  - 本地知識智能客服（ReAct 循環）
  - 工具鏈組合設計（Agent + Tool）
  - 故障診斷 Agent（多步驟推理）
- 適合：快速原型、廣泛的生態整合

### LlamaIndex — 專注在資料索引與 RAG
https://docs.llamaindex.ai/en/stable/

- **核心概念**：Document → Node → Index → Retriever → QueryEngine
- 比 LangChain 更專注、抽象更乾淨
- 適合：知識庫、文件問答場景

### AutoGen — 微軟出品的多智能體框架
https://microsoft.github.io/autogen/stable/

- **核心概念**：Agent → GroupChat → Manager
- 適合：多 Agent 協作對話、角色分工

### 框架選型指南

| 需求 | 推薦 |
|------|------|
| 快速建立 RAG 原型 | LlamaIndex |
| 複雜 Agent + Tool-use | LangChain / LangGraph |
| 多 Agent 協作對話 | AutoGen |
| .NET 生態整合 | Semantic Kernel |

---

## HuggingFace 生態實戰

### 核心組件
- **Transformers**：統一的模型載入/推理 API
  https://huggingface.co/docs/transformers/index
- **Datasets**：標準化資料集處理
- **Tokenizers**：高效率分詞

### Pipelines API（零程式碼調用）
```python
from transformers import pipeline
classifier = pipeline("sentiment-analysis")
result = classifier("I love this product!")
```

### PEFT 高效微調
https://huggingface.co/docs/peft/index

- LoRA / QLoRA 原理與實作
- 搭配 transformers + bitsandbytes 可在一張顯卡上微調 7B 模型

---

## 神經網路基礎與 TensorFlow

> 課程有此節，但**對你來說只需理解概念**，不須深入

- 神經網路結構、激活函數、反向傳播
- TensorFlow / Keras 基本操作
- **重點**：理解「訓練」這件事是怎麼發生的 — 之後理解微調才有 sense

### 課程中的 TF 實戰
- Keras 二手車價格預測（回歸任務）
- TF Serving 部署（了解即可）

---

## PyTorch 與視覺檢測

### PyTorch 核心
- 張量與自動求導（你的 C# 泛型思維可平移）
- 動態圖 vs 靜態圖（PyTorch 動態 = 比較直覺）
- PyTorch Lightning（簡化訓練迴圈）

### YOLO 目標檢測
https://docs.ultralytics.com/

- YOLOv1 → v12 的演進
- **課程實戰**：鋼鐵表面缺陷檢測
- 對你的意義：工業 AI 場景（瑕疵檢測）若你們公司有類似需求就有直接價值

---

## 影片教學主力：尚硅谷 2026 LangChain 教程（120集）

這是你的主線課程：https://www.bilibili.com/video/BV1rv7A6oEeP/

### 集數對應進度表

| 集數範圍 | 主題 | 對應你目前的進度 |
|---------|------|----------------|
| 01-10 | 課程介紹、環境搭建、LangChain 概述 | 已看過 |
| **11-20** | **模型調用（OpenAI / DeepSeek / 本地模型）** | **你目前在 p12 - 調用DeepSeek** |
| 21-30 | PromptTemplate、Message 類型、輸出解析器 | 即將進入 |
| 31-40 | LCEL（LangChain Expression Language） | |
| 41-50 | Runnable 進階、Chain 組合 | |
| 51-60 | Tool 定義與調用、ToolStrategy | |
| 61-70 | Agent（ReAct 循環） | |
| 71-80 | Memory（對話記憶）、LangGraph 基礎 | |
| 81-90 | LangGraph 節點/邊/條件、鉤子函數 | |
| 91-100 | 長期記憶（PostgresStore）、多 Agent | |
| 101-110 | RAG 實戰（向量庫、檢索、生成） | |
| 111-120 | 綜合專案：智能客服知識庫 | |

> 進度：**12 / 120 集**，你目前還在模型調用階段，離 Agent/RAG 還有約 50 集

### 補充課程（選看）

| 資源 | 連結 | 說明 |
|------|------|------|
| 黑馬程序員 LangChain+LangGraph 實戰 | [前往](https://www.bilibili.com/video/BV178w1z7EHQ/) | 28 集濃縮版，全看完尚硅谷後可當複習 |
| 吳恩達 Agent 智能體教程 | [前往](https://www.bilibili.com/video/BV1DfrdByE2H/) | 官方中英字幕，附課件代碼 |

> 建議：先把尚硅谷 120 集穩穩跟完，黑馬跟吳恩達是後續補充用。
