---
{"dg-publish":true,"permalink":"/jet-note/auth/io-t/","title":"內網 / IoT 認證與加密筆記","tags":["Auth"],"dg-note-properties":{"title":"內網 / IoT 認證與加密筆記","tags":["Auth"],"created":"2026-03-19"}}
---


# 內網 / IoT 認證與加密筆記

### 1️⃣ HTTPS / TLS

* HTTPS = HTTP + TLS
* TLS 負責加密通訊，保護資料不被截取
* 建立加密通道流程:

  1. Client 產生 randomA
  2. Server 產生 randomB
  3. 雙方交換資訊並計算 sessionKey = f(randomA, randomB)
  4. 由於交換過程經過簽名或私鑰加密，駭客無法計算 sessionKey
* 內網 / MES 系統通常使用自簽憑證或公司內部 CA，避免購買公眾 CA

### 2️⃣ JWT 認證

* JSON Web Token，用於身份驗證 + 授權
* 結構: header.payload.signature (Base64)
* Server 端生成示例:

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.IdentityModel.Tokens;
using System.Text;

public string GenerateJwtToken(string userId)
{
    var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("your-256-bit-secret"));
    var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, userId),
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
    };

    var token = new JwtSecurityToken(
        issuer: "myserver",
        audience: "myclient",
        claims: claims,
        expires: DateTime.Now.AddHours(1),
        signingCredentials: credentials
    );

    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

* Client 呼叫 API:

```csharp
var client = new HttpClient();
client.DefaultRequestHeaders.Authorization =
    new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
var response = await client.GetAsync("https://myserver/api/data");
```

* JWT 特點: 可攜帶權限資訊、可驗證簽名、可設定過期

### 3️⃣ API Key 認證

* 本質: 在 HTTP Header 插入固定密碼
* Server 驗證流程:

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.TryGetValue("X-Api-Key", out var extractedApiKey))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("API Key missing");
        return;
    }

    if (extractedApiKey != "abc123")
    {
        context.Response.StatusCode = 403;
        await context.Response.WriteAsync("Unauthorized client");
        return;
    }

    await next();
});
```

* Client 使用示例:

```csharp
var client = new HttpClient();
client.DefaultRequestHeaders.Add("X-Api-Key", "abc123");
var response = await client.GetAsync("https://myserver/api/data");
```

* API Key 適用於內網 / IoT / 系統對系統驗證，簡單、固定、依靠 TLS 保護傳輸

### 4️⃣ Authorize 概念

* `Authorize` 是判斷請求是否允許的流程
* 核心是「身份驗證 + 授權」
* JWT / API Key / Cookie 都可用，Middleware 負責解析和驗證
* Bearer token = 指明 token 類型，JWT 會經過簽名驗證並解析 payload

### 5️⃣ 核心心智模型

```text
Client -> Header 帶 Token / API Key -> Server Middleware
    |- API Key: 比對固定密碼
    |- JWT: 驗證簽名 + 解析 payload
-> Authorize 判斷權限 -> API 執行
```

### ✅ 總結

* HTTPS/TLS: 保護傳輸安全，內網可用自簽或內部 CA
* API Key: 簡單身份憑證，適合 IoT / 系統對系統，需 TLS 保護
* JWT/Bearer: 規範化 token，可攜帶資訊、驗證簽名、可過期，適合使用者身份管理
* Authorize: Middleware 自動解析 Header 並決定是否授權，使用者只需提供 token/key，框架負責解碼驗證

### ✅系統安全 =>

1. TLS
   保護資料傳輸

2. Authentication
   確認是誰
   (JWT / APIKey)

3. Authorization
   確認能做什麼
   (Role / Policy)
