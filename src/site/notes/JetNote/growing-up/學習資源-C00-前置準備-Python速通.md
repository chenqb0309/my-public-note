---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c00-python/","dg-note-properties":{}}
---


# 前置準備：Python 速通（C# 開發者專用）

> 課程假設你已會 Python，但你不會。不過你有 C# 基礎，這份對照表幫你最快跨過語法門檻。

---

## 目標

不是學 Python，是**用你已有的 C# 知識直接翻譯到 Python**。預估時間：**1-2 週**達到能順暢操作 LangChain/PyTorch 的程度。

---

## C# → Python 快速對照

### 核心語法

| C# | Python | 注意 |
|----|--------|------|
| `var x = 10;` | `x = 10` | 動態型別，無需宣告 |
| `string s = "hi";` | `s = "hi"` | 字串不可變，同 C# |
| `if (x > 0) { }` | `if x > 0:` | 冒號取代大括號，縮排決定區塊 |
| `for (int i=0; i<n; i++)` | `for i in range(n):` | range 是左閉右開 |
| `foreach (var x in list)` | `for x in list:` | 更簡潔 |
| `list.Where(x => x > 0)` | `[x for x in list if x > 0]` | List comprehension |
| `list.Select(x => x * 2)` | `[x * 2 for x in list]` | 同上 |
| `async Task<T> Foo()` | `async def foo():` | await 語法幾乎相同 |

### OOP 對照

| C# | Python |
|----|--------|
| `class Foo : IBar` | `class Foo(IBar):` |
| `interface IFoo { void Bar(); }` | `class IFoo(ABC): @abstractmethod def bar()` |
| `public string Name { get; set; }` | `self.name = name` （無 access modifier） |
| `new Foo()` | `Foo()` （無 new 關鍵字） |
| `this.name` | `self.name` （self 需明確寫在參數） |

### 關鍵差異（C# 開發者容易踩的坑）

1. **縮排是語法** — 不像 C# 只是排版，Python 的縮排錯誤就是 compile error
2. **沒有 switch** — 用 `if/elif/else` 或 match（3.10+）
3. **沒有 private/protected** — 底線開頭為慣例：`_private`、`__really_private`
4. **list 不是陣列** — Python list 可混合型別，類似 `ArrayList`
5. **dict 取代 Dictionary/Hashtable** — `{"key": value}`
6. **GIL 限制多執行緒** — 多執行緒不并行，要用 `multiprocessing` 或 `asyncio`

---

## 必裝工具鏈

```bash
# Python 3.12+
python3 --version

# 套件管理（你已經有 uv）
uv --version

# 虛擬環境（必做，不然會汙染全域）
python3 -m venv .venv
source .venv/bin/activate  # WSL/Linux

# 常用套件
uv pip install jupyter  # 互動式開發，強烈推薦
uv pip install requests httpx pydantic pytest
```

---

## 優先學習的 Python 套件（按順序）

| 套件         | 用途                          | 預計時間 |
| ---------- | --------------------------- | ---- |
| `pydantic` | DTO 驗證，類似 C# DataAnnotation | 1 小時 |
| `httpx`    | HTTP 呼叫，類似 HttpClient       | 1 小時 |
| `asyncio`  | 非同步，你有 C# async 基礎          | 2 小時 |
| `pytest`   | 單元測試，類似 xUnit               | 1 小時 |

---

## 實戰練習

1. 用 pydantic 定義一個 `VehicleArrival` DTO（跟你 C# InboundContracts 一樣的結構）
2. 用 httpx 呼叫一個 OpenAI API
3. 用 asyncio + httpx 同時發送 5 個 API 請求

> 不需要學：class inheritance deep dive、multi-threading、C-extensions

---

## 參考資源

- **Python 官方速通**（3 小時）：https://docs.python.org/3/tutorial/
- **Real Python 爬蟲/API 實戰**：https://realpython.com/
- **C# 開發者學 Python 筆記**：搜尋 "Python for C# developers"

---

## 影片教學

| 資源 | 連結 | 時長 | 說明 |
|------|------|------|------|
| Bilibili：[3小時超快速入門Python - 林粒粒呀](https://www.bilibili.com/video/BV1Jgf6YvE8e/) | 3h | 動畫教學，707萬播放。C#背景可直接從這開始 |

> 這一門只是「速通」，你的主戰場在尚硅谷 LangChain 那門課。

---

## C# async/await → Python asyncio 對照

> 你寫 MES 時用 C# async/await，在 Python 寫 LLM 應用也會碰到。對照表在此。

| 概念 | C# | Python |
|------|----|--------|
| 非同步方法宣告 | `async Task<T> FooAsync()` | `async def foo():` |
| 等待結果 | `var result = await foo;` | `result = await foo` |
| 建立背景工作 | `Task.Run(() => DoWork())` | `asyncio.create_task(foo())` |
| 等待多個工作 | `await Task.WhenAll(t1, t2)` | `await asyncio.gather(t1, t2)` |
| 暫停 | `await Task.Delay(ms)` | `await asyncio.sleep(秒)` |

### 重要差異

```python
# Python：ainvoke 只是建立 coroutine，還沒開始跑
coro = llm.ainvoke(prompt)    # ← 還沒執行！
result = await coro            # ← 才開始跑

# 要立即在背景跑 → 用 create_task
task = asyncio.create_task(llm.ainvoke(prompt))  # ← 立即開始
```

> 底層實作不同（C# 多 thread / Python 單 thread event loop），但**應用層寫法幾乎一樣**——寫應用時不需要關心這個差異。
