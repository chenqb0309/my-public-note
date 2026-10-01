---
{"dg-publish":true,"permalink":"/jet-note/auth/auth/","title":"授權架構筆記（部門 × 能力 × 頁面）","tags":["Auth"],"dg-note-properties":{"title":"授權架構筆記（部門 × 能力 × 頁面）","tags":["Auth"],"created":"2025-12-09"}}
---


# 授權架構筆記（部門 × 能力 × 頁面）

## 一、授權的三層模型

1. **部門（Department）**

   * 代表使用者的身分，如 Logistics、Production。
   * 不直接控制頁面存取。

2. **能力（Permission / Claims）**

   * 真正決定能否進入頁面。
   * 例如：Inventory.View、Shipping.Print、WorkOrder.Edit。
   * 使用者登入後被發放這些能力。

3. **頁面（Page）**

   * 透過 Policy 要求特定能力。
   * `@attribute [Authorize(Policy = "能力名稱")]`

---

## 二、核心運作流程

1. **使用者登入** → 系統根據部門查出能力
2. **系統將能力寫入 Claims（鑰匙）**
3. **Policy 定義能力需求（鎖的規則）**
4. **頁面套用 Policy（要用哪把鑰匙開門）**

---

## 三、能力與頁面的對應

* 一個部門可以擁有多個能力。
* 一個能力可以授權多個頁面。
* 頁面不直接看部門，只看能力。

---

## 四、範例

### Logistics 員工登入後的 Claims

* permission = Inventory.View
* permission = Shipping.Print

### Policy 設定

* InventoryViewer → 需要 Inventory.View
* ShippingOperator → 需要 Shipping.Print

### 頁面授權

* 庫存頁 → [Authorize(Policy="InventoryViewer")]
* 標籤列印頁 → [Authorize(Policy="ShippingOperator")]

---

## 五、規劃授權時的原則

* 部門 ≠ 頁面控制
* 能力（Permission）才是授權中心
* Policy 是能力的檢查規則
* 頁面透過 Policy 管控存取

---

## 六、建議的後續任務

* 建立「部門 → 能力 → 頁面」完整矩陣表
* 規劃統一的 Permission 列舉或資料表
* 建立登入後的 Claims 發放邏輯
* 將 Policy 標準化便於維護

---

（可隨時補充更多頁面、部門與能力對應）

---

# 七、部門 × 能力 × 頁面矩陣（範例）

| 部門         | 能力 (Permission)  | 能力描述   | 對應頁面        |
| ---------- | ---------------- | ------ | ----------- |
| Logistics  | Inventory.View   | 可查看庫存  | 庫存查詢頁、庫存報表頁 |
| Logistics  | Shipping.Print   | 列印貨運標籤 | 標籤列印頁       |
| Production | WorkOrder.Edit   | 編輯工單   | 工單編輯頁       |
| Production | WorkOrder.Finish | 完工工單   | 工單完工頁       |

> 可依系統需求再擴充。

---

# 八、Permission Enum（建議方式）

```csharp
public static class Permissions
{
    public static class Inventory
    {
        public const string View = "Inventory.View";
    }

    public static class Shipping
    {
        public const string Print = "Shipping.Print";
    }

    public static class WorkOrder
    {
        public const string Edit = "WorkOrder.Edit";
        public const string Finish = "WorkOrder.Finish";
    }
}
```

優點：

* 統一管理所有權限字串
* 不容易拼錯
* 擴充性高

---

# 九、Policy 標準化（Startup/Program 設定示例）

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("InventoryViewer", p =>
        p.RequireClaim("permission", Permissions.Inventory.View));

    options.AddPolicy("ShippingOperator", p =>
        p.RequireClaim("permission", Permissions.Shipping.Print));

    options.AddPolicy("WorkOrderEditor", p =>
        p.RequireClaim("permission", Permissions.WorkOrder.Edit));

    options.AddPolicy("WorkOrderFinisher", p =>
        p.RequireClaim("permission", Permissions.WorkOrder.Finish));
});
```

建議規則：

* Policy 名稱用「功能角色」命名（如 Editor、Viewer、Operator）
* Policy 只做一件事：檢查 Permission

---

# 十、登入後 Claims 發放邏輯（範例）

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Account),
    new Claim("department", user.Department)
};

// 根據部門決定能力
if (user.Department == "Logistics")
{
    claims.Add(new Claim("permission", Permissions.Inventory.View));
    claims.Add(new Claim("permission", Permissions.Shipping.Print));
}
else if (user.Department == "Production")
{
    claims.Add(new Claim("permission", Permissions.WorkOrder.Edit));
    claims.Add(new Claim("permission", Permissions.WorkOrder.Finish));
}

var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(identity));
```

這段程式碼的作用：

* 登入成功後，把能力寫成 Claims
* 後續所有頁面就能用 Policy 檢查

---

# 十一、頁面使用範例（Blazor / Razor 頁面）

```csharp
@attribute [Authorize(Policy = "InventoryViewer")]
```

或多能力（AND）：

```csharp
@attribute [Authorize(Policy = "WorkOrderEditor")]
```

---

# 十二、Radzen Component 顯示/隱藏（依權限）

```csharp
@if (AuthorizationService.AuthorizeAsync(User, "InventoryViewer").Result.Succeeded)
{
    <RadzenButton Text="庫存查詢" />
}
```

用途：

* 讓沒有權限的人 UI 上也看不到按鈕

---

# 十三、完整授權架構總結

1. **部門決定能力（Permission）**
2. **能力被轉成 Claims**
3. **Policy 檢查 Claims**
4. **頁面使用 Policy 控制進入**
5. **UI 也可依 Policy 動態顯示**

---