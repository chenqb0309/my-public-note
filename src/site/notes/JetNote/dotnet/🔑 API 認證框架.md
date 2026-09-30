---
{"dg-publish":true,"permalink":"/jet-note/dotnet/api/","title":"🔑 API 認證框架","tags":[".Net","Development"],"dg-note-properties":{"title":"🔑 API 認證框架","tags":[".Net","Development"],"created":"2025-09-30"}}
---


# 🔑 API 認證框架

## 1. 核心概念

* **認證 (Authentication)** → 你是誰？
* **授權 (Authorization)** → 你能做什麼？
* **Pipeline 流程**：先認證 → 再授權 → 最後執行 API。

---

## 2. 憑證形式

1. **API Key**

   * 簡單字串，放 Header / Query。
   * 缺乏過期、權限控管。
2. **JWT (JSON Web Token)**

   * `header.payload.signature`
   * 攜帶 Claims，支援過期、簽名驗證。
   * 缺點：不易撤銷。
3. **OAuth2 Token**

   * Access + Refresh Token。
   * 標準協定，安全性高。

---

## 3. 憑證生命週期

1. **簽發 (Issue)** → 用 SigningKey 產生 token。
2. **存放 (Storage)** → LocalStorage / Cookie / Session。
3. **傳遞 (Transmission)** →

   ```http
   Authorization: Bearer <token>
   ```
4. **驗證 (Validation)** → 簽名、過期、Issuer、Audience。

---

## 4. Pipeline 驗證流程

```csharp
app.UseAuthentication();  // 驗證 token
app.UseAuthorization();   // 檢查權限
app.MapControllers();     // 執行 API
```

* 無 token → 401 Unauthorized
* 權限不足 → 403 Forbidden

---

## 5. Debug Checklist (401 排查)

1. Header 有 token 嗎？
2. "Bearer" 大小寫正確嗎？
3. Token 是否過期？(`exp`)
4. Issuer / Audience 是否匹配？
5. SigningKey 是否一致？
6. Middleware 順序正確嗎？

---

## 6. 補充知識

* **SigningKey 不可外流** → 前端僅拿到 token，不拿金鑰。
* **短期 token + Refresh Token** → 減少風險。
* **Token 撤銷** → 需黑名單或縮短過期時間。
* **環境一致性** → Dev / Local / Prod SigningKey 與 Issuer 必須對齊。

---

## 7. 方法論比喻

* **憑證 = 門票**
* **金鑰 = 印章**（只有主辦方有）
* **Pipeline = 驗票口**
* **Token Payload = 票面資訊（姓名、有效期）**
