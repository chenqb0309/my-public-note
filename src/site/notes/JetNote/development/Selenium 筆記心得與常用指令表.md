---
{"dg-publish":true,"permalink":"/jet-note/development/selenium/","title":"Selenium 筆記心得與常用指令表","tags":["Selenium","Testing","Automation"],"dg-note-properties":{"title":"Selenium 筆記心得與常用指令表","tags":["Selenium","Testing","Automation"],"created":"2025-07-03"}}
---


# Selenium 筆記心得與常用指令表

## 📌 Selenium 簡介

Selenium 是一套開源自動化工具，用於模擬使用者操作瀏覽器。可應用於自動化測試、資料擷取、流程操作等場景。支援多種語言（如 C#、Java、Python）與瀏覽器（如 Chrome、Edge、Firefox）。

---

## ⚙ 主要功能

* 模擬瀏覽器中各種使用者操作：點擊、輸入、捲動、選單選取、上傳檔案。
* 可搭配等待機制處理非同步載入。
* 支援 Headless 模式以減少資源消耗。
* 可整合至 CI/CD 流程中。

---

## 🔎 元素定位方式

* `By.Id` / `By.Name`：首選，穩定、易讀。
* `By.ClassName` / `By.TagName`：用於簡單結構。
* `By.CssSelector`：語法簡潔，適用層級少的定位。
* `By.XPath`：適合複雜或需條件篩選的元素定位。
* `By.LinkText` / `By.PartialLinkText`：定位純文字連結。

範例：

```csharp
driver.FindElement(By.Id("submitBtn")).Click();
driver.FindElement(By.CssSelector("div.container input[type='text']")).SendKeys("測試");
driver.FindElement(By.XPath("//li[@role='option' and contains(@aria-label,'公司名稱')]")).Click();
```

---

## 🖱 操作與互動技巧

### 點擊與輸入

```csharp
element.Click();
element.SendKeys("輸入內容");
element.Clear();
```

### 捲動至元素

```csharp
((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].scrollIntoView(true);", element);
```

### JavaScript 點擊

```csharp
((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].click();", element);
```

### 滑鼠模擬

```csharp
new Actions(driver).MoveToElement(element).Click().Perform();
new Actions(driver).DragAndDrop(source, target).Perform();
```

---

## ⏳ 等待機制

```csharp
WebDriverWait wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
IWebElement element = wait.Until(d => d.FindElement(By.Id("targetId")));
```

---

## ⚠ 常見挑戰

* 元素屬性或位置變動，導致定位失效。
* 下拉選單、動態彈窗需搭配捲動、等待、滑鼠模擬處理。
* 頁面改版需調整定位與操作邏輯。

---

## 🛡 最佳實務

```csharp
public void SafeClick(IWebElement element)
{
    try { element.Click(); }
    catch {
        ((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].scrollIntoView(true);", element);
        ((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].click();", element);
    }
}

public void SendText(IWebElement element, string text)
{
    element.Clear();
    element.SendKeys(text);
}
```

---

## 🗂 常用指令表 (C# 範例)

| 功能        | 指令範例                                                                                                                                           |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 尋找元素 (單一) | `driver.FindElement(By.Id("id"))`<br>`driver.FindElement(By.Name("name"))`<br>`driver.FindElement(By.XPath("//div[@class='class']"))`          |
| 尋找元素 (多個) | `driver.FindElements(By.ClassName("className"))`                                                                                               |
| 點擊        | `element.Click();`                                                                                                                             |
| 輸入文字      | `element.SendKeys("輸入內容");`                                                                                                                    |
| 清除輸入欄位    | `element.Clear();`                                                                                                                             |
| 取得文字      | `string text = element.Text;`                                                                                                                  |
| 取得屬性值     | `string attr = element.GetAttribute("value");`                                                                                                 |
| 等待元素出現    | `WebDriverWait wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));`<br>`IWebElement el = wait.Until(d => d.FindElement(By.Id("id")));` |
| 捲動到元素     | `((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].scrollIntoView(true);", element);`                                                  |
| JS 點擊     | `((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].click();", element);`                                                               |
| 滑鼠移動      | `new Actions(driver).MoveToElement(element).Perform();`                                                                                        |
| 拖曳元素      | `new Actions(driver).DragAndDrop(source, target).Perform();`                                                                                   |
| 取得當前網址    | `string url = driver.Url;`                                                                                                                     |
| 切換 iframe | `driver.SwitchTo().Frame("iframeNameOrId");`                                                                                                   |
| 返回主頁框架    | `driver.SwitchTo().DefaultContent();`                                                                                                          |
| 切換視窗      | `driver.SwitchTo().Window(driver.WindowHandles[1]);`                                                                                           |
| 關閉視窗      | `driver.Close();`                                                                                                                              |
| 結束瀏覽器     | `driver.Quit();`                                                                                                                               |

---

## 💡 總結

Selenium 是功能完整的瀏覽器自動化工具。元素定位、等待條件、互動技巧需設計妥當，以提升穩定性與維護性。遇到大量資料處理或高效能需求時，應優先評估 API 或替代方案（如 Playwright）。
