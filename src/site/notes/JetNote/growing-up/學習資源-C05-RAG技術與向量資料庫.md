---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c05-rag/","dg-note-properties":{}}
---


# 學習資源：AI 應用技術 — RAG

> 課程 Area 5。結合原 05-RAG 與 09-向量資料庫，完整對應課程結構。

---

## 理論基礎

### Embeddings 與向量化
- Embedding 定義與工作方式：文字 → 向量座標
- 餘弦相似度（Cosine Similarity）
- MTEB 榜單（選擇 Embedding 模型的依據）
- 「俄羅斯套娃」模型（Matryoshka Embedding）：同一模型輸出不同維度

### 向量資料庫 vs 傳統資料庫
| 比較項 | 向量 DB | 傳統 DB |
|--------|---------|---------|
| 查詢方式 | 相似度搜索 | 精確比對 |
| 索引類型 | HNSW / IVFFlat | B-Tree |
| 適合場景 | 語義搜索、推薦 | 交易、精確查詢 |

---

## 完整 RAG Pipeline

```
使用者問題
  → Query 改寫（LLM 重寫為更適合檢索的形式）
  → 檢索（多路召回：向量 + 關鍵詞 BM25）
  → 重排序（Cross-encoder Reranker）
  → 生成（LLM + 檢索結果 → 回答）
```

### 實作順序建議

| 步驟 | 工具 | 時程 |
|------|------|------|
| 1. 基礎 RAG | LangChain + Chroma | 2-3 天 |
| 2. 加入 Hybrid Search | Elasticsearch/Bm25 | +1 天 |
| 3. 加入 Reranker | Cohere/BGE Reranker | +1 天 |
| 4. 多模態 RAG | PDF/圖片處理 | +2 天 |
| 5. 評估與調優 | RAGAS / TruLens | +2 天 |

---

## 實戰資源

### 課程冠軍方案拆解
課程包含一個企業 RAG 大賽冠軍方案的完整程式碼。

**架構亮點**：
- 多路由檢索：不同類型的問題走不同的檢索策略
- 動態知識庫：根據上下文決定檢索範圍
- Docling 文檔解析：PDF/Word/HTML 的結構化提取

---

## 向量資料庫選擇

### 開發用（最簡單）
- **Chroma**：`pip install chromadb`，零設定
  https://docs.trychroma.com/

### 本地生產
- **pgvector**：PostgreSQL 擴充，你有 SQL 背景最好上手
  https://github.com/pgvector/pgvector
- **Qdrant**：Rust 實作，效能佳
  https://qdrant.tech/documentation/

### 雲端
- **Pinecone**：SaaS，免運維
- **Milvus**：分散式，適合大規模

---

## RAG 取代方案（課程也有提到）

課程特別開了一節「可以取代 RAG 的技術」，不是你學了 RAG 就結束：

| 技術 | 概念 | 適合場景 |
|------|------|---------|
| Long-Context LLM | 直接把知識塞進 context | 少量文件、不要求效率 |
| Agentic Search | Agent 決定何時檢索 | 動態需求、多步驟任務 |
| LLM Wiki | 編譯式知識庫 | 高品質、結構化知識 |
