# Angular + TypeScript Coding Style Guide

## 📋 目錄 (Table of Contents)

### 1. [型別規則 (Type Safety)](#1-型別規則-type-safety)

- [1.1 禁止隨意使用 `any`](#11-禁止隨意使用-any)
  - 推薦替代方式
  - 例外情況

### 2. [其他建議規則](#2-其他建議規則)

- [2.1 優先使用 optional 而非 `|undefined`](#21-優先使用-optional-而非-undefined)
- [2.2 避免 `null`，優先使用 `undefined`](#22-避免-null優先使用-undefined)
- [2.3 使用 `readonly` 保護不可變資料](#23-使用-readonly-保護不可變資料)
- [2.4 使用 `enum` 或 `as const` 限制常數值](#24-使用-enum-或-as-const-限制常數值)
- [2.5 盡量使用 `interface` 定義資料結構](#25-盡量使用-interface-定義資料結構)
- [2.6 Type assertions 與 object literals](#26-type-assertions-與-object-literals)
  - 避免的寫法 - 使用 type assertions
  - 推薦的寫法 - 使用 type annotations
  - 為什麼 type annotations 更安全
  - 實際應用範例
  - 例外情況
- [2.7 switch 語句規則](#27-switch-語句規則)
  - 必須包含 default 子句
  - 非空的 case 必須以終止語句結束
  - 空的 case 可以穿透
- [2.8 註解的使用原則](#28-註解的使用原則)
  - 註解語言規範
  - 何時需要寫註解
  - 不必要的註解
  - 註解品質原則
- [2.9 函數呼叫時的參數註解](#29-函數呼叫時的參數註解)
  - 何時使用參數名稱註解
  - 優先考慮重構
  - 參數註解格式規範
  - 實際應用範例
  - Angular 特定範例
- [2.10 Angular Service 與 Component 的 API](#210-angular-service-與-component-的-api)
- [2.11 依賴注入：Constructor Injection vs inject()](#211-依賴注入constructor-injection-vs-inject)
  - 推薦：使用 inject() 函數
  - 條件和可選注入
  - 在函數中使用 inject()
  - 舊版：Constructor Injection
  - 混合使用的最佳實踐
  - 注意事項

### 3. [Component 生命週期與最佳實踐](#3-component-生命週期與最佳實踐)

- [3.1 檔案與類別結構順序](#31-檔案與類別結構順序)
- [3.2 生命週期 Hook 使用原則](#32-生命週期-hook-使用原則)
  - 明確實作必要的介面
  - 常用生命週期 Hook 的適用場景
- [3.3 變更檢測策略](#33-變更檢測策略)
  - 強烈建議使用 OnPush
  - 使用 Signal 的現代做法
- [3.4 表單處理最佳實踐](#34-表單處理最佳實踐)
  - Reactive Forms 與型別安全
- [3.5 錯誤處理模式](#35-錯誤處理模式)
- [3.6 組件間通訊最佳實踐](#36-組件間通訊最佳實踐)
- [3.7 Angular Control Flow 語法 (Angular 17+)](#37-angular-control-flow-語法-angular-17)
  - 新的控制流語法概述
  - @if 條件渲染
  - @for 迴圈渲染
  - @switch 條件分支
  - 與傳統結構指令的比較
  - 最佳實踐建議

### 4. [RxJS 與訂閱管理 (Unsubscribing)](#4-rxjs-與訂閱管理-unsubscribing)

- [4.1 必須正確取消訂閱](#41-必須正確取消訂閱)
- [4.2 推薦做法](#42-推薦做法)
  - 使用 Signal (Angular 16+) - 最推薦
  - 使用 `async pipe`
  - 使用 `takeUntilDestroyed` (Angular 16+)
  - 使用 `takeUntil` (舊版 Angular)
- [4.3 禁止做法](#43-禁止做法)
- [4.4 Signal vs RxJS 選擇建議](#44-signal-vs-rxjs-選擇建議)

---

## 1. 型別規則 (Type Safety)

### 1.1 禁止隨意使用 `any`

- `any` 是所有型別的 super 和 subtype，會導致靜態型別系統失效。
- 使用 `any` 會讓程式失去型別檢查的保障，容易隱藏錯誤。

#### 推薦替代方式

- **更精確的型別**

  ```ts
  // 避免
  let data: any;

  // 推薦
  interface User {
    id: number;
    name: string;
  }
  let data: User;
  ```

- **使用 `unknown`（需要 runtime 判斷時）**

  ```ts
  function parse(input: string): unknown {
    return JSON.parse(input);
  }

  const result = parse('{"id":1}');
  if (typeof result === "object" && result !== null) {
    // type narrowing
  }
  ```

- **使用泛型 (Generics)**

  ```ts
  function getValue<T>(key: string): T {
    // ...
  }
  const name = getValue<string>("username");
  ```

#### 例外情況

在以下情況可例外使用 `any`：

- 整合第三方函式庫（缺乏型別定義）
- 處理動態結構、舊系統遺留代碼

必須：

- **明確標註** ESLint/TS ignore，並說明原因
  ```ts
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  function legacyHandler(payload: any) {
    // 來自舊系統，型別結構不固定
  }
  ```
- **Code Review 必須檢視使用合理性**

---

> ### **重點總結**
>
> - **嚴格避免使用 `any`**，優先選擇精確型別定義
> - 使用 `unknown` 處理動態型別，配合 type narrowing
> - 善用泛型 (Generics) 提供型別安全的彈性
> - 例外使用 `any` 時必須標註原因並經過 Code Review
> - 建立完整的型別系統是 TypeScript 開發的基礎

---

## 2. 其他建議規則

### 2.1 優先使用 optional 而非 `|undefined`

TypeScript 支援可選參數和字段的特殊語法，使用 `?:`：

```ts
interface CoffeeOrder {
  sugarCubes: number;
  milk?: Whole | LowFat | HalfHalf;
}

function pourCoffee(volume?: Milliliter) {
  // ...
}
```

**原則說明：**

- 可選參數隱含地包含 `|undefined` 在其型別中
- 但它們不同之處在於可以在建構值或呼叫方法時省略
- 例如，`{sugarCubes: 1}` 是有效的 `CoffeeOrder`，因為 `milk` 是可選的

**建議做法：**

- 使用可選字段（在 interface 或 class 上）和參數，而非 `|undefined` 型別
- 對於 class，盡量避免這種模式，並盡可能初始化更多字段

```ts
// 推薦
interface User {
  id: number;
  name: string;
  email?: string; // 可選字段
}

function createUser(name: string, email?: string): User {
  // ...
}

// 避免
interface User {
  id: number;
  name: string;
  email: string | undefined;
}

// 對於 class，優先初始化字段
class MyClass {
  field = ""; // 推薦：初始化預設值
}
```

### 2.2 避免 `null`，優先使用 `undefined`

- TypeScript 社群普遍建議使用 `undefined` 代表「無值」，減少混淆。

### 2.3 使用 `readonly` 保護不可變資料

```ts
interface Config {
  readonly apiUrl: string;
}
```

### 2.4 使用 `enum` 或 `as const` 限制常數值

```ts
const ROLES = ["admin", "user", "guest"] as const;
type Role = (typeof ROLES)[number];
```

### 2.5 盡量使用 `interface` 定義資料結構

- 若需要 union/intersection 或 utility types，再考慮 `type`。

### 2.6 Type assertions 與 object literals

使用 **type annotations** (`: Foo`) 而非 **type assertions** (`as Foo`) 來指定 object literal 的型別。這樣能在介面欄位變更時偵測到重構錯誤。

#### 2.6.1 避免的寫法 - 使用 type assertions

```ts
interface Foo {
  bar: number;
  baz?: string; // 原本是 "bam"，後來重新命名為 "baz"
}

// 錯誤做法：使用 as 斷言
const foo = {
  bar: 123,
  bam: "abc", // 沒有錯誤！但 bam 在介面中已不存在
} as Foo;

function func() {
  return {
    bar: 123,
    bam: "abc", // 沒有錯誤！但這會導致執行時問題
  } as Foo;
}
```

#### 2.6.2 推薦的寫法 - 使用 type annotations

```ts
interface Foo {
  bar: number;
  baz?: string;
}

// 正確做法：使用 type annotation
const foo: Foo = {
  bar: 123,
  bam: "abc", // TypeScript 編譯錯誤：'bam' 不存在於 Foo 型別中
};

function func(): Foo {
  return {
    bar: 123,
    bam: "abc", // TypeScript 編譯錯誤：'bam' 不存在於 Foo 型別中
  };
}
```

#### 2.6.3 為什麼 type annotations 更安全

**Type assertions (`as Foo`) 的問題：**

- 告訴 TypeScript "相信我，這個物件就是 Foo 型別"
- 跳過型別檢查，可能隱藏錯誤
- 當介面變更時，不會提醒開發者更新程式碼

**Type annotations (`: Foo`) 的優點：**

- 要求物件必須完全符合介面定義
- 介面變更時會立即顯示編譯錯誤
- 提供更好的 IDE 支援和自動完成

#### 2.6.4 實際應用範例

```ts
// Angular Component 範例
interface UserData {
  id: number;
  name: string;
  email?: string;
}

@Component({...})
export class UserComponent {
  // 推薦：使用 type annotation
  private defaultUser: UserData = {
    id: 0,
    name: 'Guest',
    email: undefined // 可選欄位可以明確設為 undefined
  };

  // 避免：使用 type assertion
  private badDefaultUser = {
    id: 0,
    name: 'Guest',
    // 如果忘記某些欄位，type assertion 不會警告
  } as UserData;

  createUser(name: string, email?: string): UserData {
    // 推薦：return type annotation 確保回傳值正確
    return {
      id: Date.now(),
      name,
      email
    };
  }

  // Angular HTTP 回應處理
  loadUser(id: number): Observable<UserData> {
    return this.http.get<UserData>(`/api/users/${id}`).pipe(
      map((response): UserData => {
        // 明確的型別轉換，確保回應格式正確
        return {
          id: response.id,
          name: response.name,
          email: response.email
        };
      })
    );
  }
}
```

#### 2.6.5 例外情況

在以下情況可以考慮使用 type assertions：

```ts
// 1. DOM 元素型別轉換
const canvas = document.getElementById("myCanvas") as HTMLCanvasElement;

// 2. 與外部 API 整合時，回應格式無法完全控制
const apiResponse = await fetch("/api/data");
const data = (await apiResponse.json()) as ApiResponseType;

// 3. 測試程式碼中的 mock 物件
const mockUser = {
  id: 1,
  name: "Test User",
} as User; // 測試時可能只需要部分欄位
```

---

> ### **重點總結**
>
> - **優先使用 type annotations (`: Type`)**，提供更好的型別安全性
> - **避免 type assertions (`as Type`)**，除非確實必要
> - 利用 TypeScript 的編譯時檢查，及早發現重構時的錯誤
> - 在 Angular 中特別注意 HTTP 回應和 Component 資料的型別定義

---

### 2.7 switch 語句規則

#### 2.7.1 必須包含 default 子句

每個 `switch` 都必須有一個 `default` 區塊，即使裡面什麼都不做也要寫。`default` 必須放在最後。

```ts
// 推薦：基本 default 用法
switch (x) {
  case Y:
    doSomethingElse();
    break;
  default:
  // nothing to do.
}

// 推薦：處理意外值
function getStatusColor(status: "active" | "inactive" | "pending") {
  switch (status) {
    case "active":
      return "green";
    case "inactive":
      return "gray";
    case "pending":
      return "yellow";
    default:
      // 明確處理未預期的值
      throw new Error(`Unknown status: ${status}`);
  }
}
```

**原因：**

- **防禦性程式設計**：處理意外的輸入值
- **未來擴展性**：當聯合型別或枚舉增加新值時，提醒開發者更新程式碼
- **明確的錯誤處理**：避免靜默失敗或返回 `undefined`

#### 2.7.2 非空的 case 必須以終止語句結束

每個非空的 case 必須明確結束，方式包括：`break`、`return`、`throw`。
非空的 case 不能隱式穿透，編譯器會強制檢查。

```ts
// 正確：每個非空 case 都有終止語句
switch (x) {
  case X:
    doSomething();
    break; // 必須終止
  case Y:
    doSomethingElse();
    return; // 或 break / throw
  case Z:
    throw new Error("Invalid case");
  default:
  // fallback
}
```

```ts
// 不允許穿透：
switch (x) {
  case X:
    doSomething();
  // fall through - 禁止!
  case Y:
    doSomethingElse();
    break;
}
```

#### 2.7.3 空的 case 可以穿透

如果某個 case 是空的，可以穿透到下一個 case，通常用於多個 case 執行相同邏輯：

```ts
// 正確：空 case 穿透
switch (x) {
  case X:
  case Y:
    doSomething();
    break;
  case Z:
    doSomethingElse();
    break;
  default:
  // 不需要做任何事
}

// 正確：複雜的空 case 穿透邏輯
switch (userRole) {
  case "admin":
  case "superuser":
  case "moderator":
    hasAdvancedPermissions = true;
    break;
  case "user":
  case "guest":
    hasAdvancedPermissions = false;
    break;
  default:
    throw new Error(`Unknown role: ${userRole}`);
}
```

**TypeScript 技巧：使用 exhaustive check**

```ts
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}

function getStatusColor(status: Status) {
  switch (status) {
    case "active":
      return "green";
    case "inactive":
      return "gray";
    case "pending":
      return "yellow";
    default:
      return assertNever(status); // TypeScript 會檢查是否遺漏 case
  }
}
```

---

> ### **重點總結**
>
> - 必須有 `default`，並放最後
> - 非空 case 不能穿透，必須終止
> - 空 case 可以穿透，通常用於合併多個 case
> - 所有非空 case 都要明確結束（`break` / `return` / `throw`）

---

### 2.8 註解的使用原則

#### 2.8.1 註解語言規範

**必須使用英文撰寫註解**

- 所有程式碼註解必須使用英文，包括單行註解 `//` 和多行註解 `/* */`
- JSDoc 註解也必須使用英文
- 英文註解有利於團隊國際化協作和程式碼維護

```ts
// 正確：使用英文註解
// Calculate user's total score based on completed tasks
function calculateScore(tasks: Task[]): number {
  // Filter completed tasks only
  const completedTasks = tasks.filter((task) => task.completed);
  return completedTasks.length * 10;
}

// 錯誤：使用中文註解
// 計算使用者總分數
function calculateScore(tasks: Task[]): number {
  // 只篩選已完成的任務
  const completedTasks = tasks.filter((task) => task.completed);
  return completedTasks.length * 10;
}
```

#### 2.8.2 何時需要寫註解

**必須寫註解的情況：**

1. **複雜的業務邏輯或演算法**

   ```ts
   // Validate credit card using Luhn algorithm
   function validateCreditCard(cardNumber: string): boolean {
     // Remove spaces and dashes
     const cleanNumber = cardNumber.replace(/[\s-]/g, "");

     // Process each digit from right to left
     let sum = 0;
     let isEven = false;

     for (let i = cleanNumber.length - 1; i >= 0; i--) {
       let digit = parseInt(cleanNumber[i]);

       // Double every second digit
       if (isEven) {
         digit *= 2;
         // If result > 9, add the digits together
         if (digit > 9) {
           digit = (digit % 10) + Math.floor(digit / 10);
         }
       }

       sum += digit;
       isEven = !isEven;
     }

     return sum % 10 === 0;
   }
   ```

2. **不直觀的解決方案或 Workaround**

   ```ts
   // Fix Safari date parsing issue on iOS 15
   // Must convert 'YYYY-MM-DD' format to 'YYYY/MM/DD'
   function parseDate(dateString: string): Date {
     const safariCompatibleDate = dateString.replace(/-/g, "/");
     return new Date(safariCompatibleDate);
   }
   ```

3. **警告或重要提醒**

   ```ts
   /**
    * Warning: This method modifies the original array
    * If you need to keep the original array unchanged, use [...array] to copy first
    */
   function sortUsers(users: User[]): User[] {
     return users.sort((a, b) => a.name.localeCompare(b.name));
   }
   ```

4. **API 文件註解 (JSDoc)**
   ```ts
   /**
    * Calculate working days between two dates
    * @param startDate Start date
    * @param endDate End date
    * @param excludeWeekends Whether to exclude weekends (default: true)
    * @returns Number of working days
    * @throws {Error} When end date is earlier than start date
    */
   function calculateWorkingDays(
     startDate: Date,
     endDate: Date,
     excludeWeekends = true
   ): number {
     // Implementation logic...
   }
   ```

#### 2.8.3 不必要的註解

**避免這些冗餘註解：**

1. **重述程式碼內容**

   ```ts
   // Bad: Comment just repeats the code
   const userId = user.id; // Get user ID

   // Good: Code is self-explanatory, no comment needed
   const userId = user.id;
   ```

2. **顯而易見的註解**

   ```ts
   // Bad: Too obvious
   function addNumbers(a: number, b: number): number {
     return a + b; // Return sum of a and b
   }

   // Good: Function name and types explain everything
   function addNumbers(a: number, b: number): number {
     return a + b;
   }
   ```

3. **過時或錯誤的註解**
   ```ts
   // Dangerous: Comment doesn't match the code
   // Check if user is admin
   function canDeletePost(user: User): boolean {
     return user.role === "moderator" || user.role === "admin";
   }
   ```

#### 2.8.4 註解品質原則

**好註解的特徵：**

- **解釋「為什麼」而非「是什麼」**
- **提供上下文和背景資訊**
- **警告潛在問題或副作用**
- **說明不直觀的決策原因**

```ts
// Good comment: Explains why this approach is used
// Use setTimeout instead of setImmediate to ensure consistency between Node.js and browser environments
setTimeout(() => {
  this.processQueue();
}, 0);

// Good comment: Provides important business context
// According to FSC regulations, credit card transactions over NT$30,000 require additional verification
if (transaction.amount > 30000) {
  return this.requireAdditionalVerification(transaction);
}
```

---

> ### **重點總結**
>
> - **必須使用英文撰寫所有註解**
> - 註解應該解釋「為什麼」，而非「做什麼」
> - 程式碼應該自我解釋，必要時才加註解
> - 保持註解與程式碼同步更新
> - 使用 JSDoc 為公開 API 提供完整文件

---

### 2.9 函數呼叫時的參數註解

#### 2.9.1 何時使用參數名稱註解

當**方法名稱**和**參數值**無法充分表達參數含義時，應使用參數名稱註解。

**需要參數註解的情況：**

```ts
// 不清楚：無法理解參數含義
processOrder(order, true, false, "urgent");

// 清楚：使用參數註解
processOrder(
  order,
  /* shouldValidate= */ true,
  /* sendNotification= */ false,
  /* priority= */ "urgent"
);
```

#### 2.9.2 優先考慮重構

在加入參數註解前，考慮重構方法接受 interface 並使用解構，可大幅提升呼叫端的可讀性：

```ts
// 最佳做法：使用 interface
interface ProcessOrderOptions {
  shouldValidate: boolean;
  sendNotification: boolean;
  priority: "low" | "normal" | "urgent";
}

function processOrder(order: Order, options: ProcessOrderOptions) {
  const { shouldValidate, sendNotification, priority } = options;
  // 處理邏輯...
}

// 呼叫時非常清楚
processOrder(order, {
  shouldValidate: true,
  sendNotification: false,
  priority: "urgent",
});
```

#### 2.9.3 參數註解格式規範

**推薦格式：**參數註解放在參數值**之前**，包含參數名稱和 `=` 後綴：

```ts
// 推薦格式
someFunction(
  obviousParam,
  /* shouldRender= */ true,
  /* name= */ "hello",
  /* timeout= */ 5000
);

// 複雜呼叫的範例
createUser(
  userData,
  /* validateEmail= */ true,
  /* sendWelcomeEmail= */ false,
  /* assignDefaultRole= */ true
);
```

**舊版格式：**參數註解放在參數值**之後**，不含 `=`：

```ts
// 舊版格式（為了一致性可在現有檔案中繼續使用）
someFunction(
  obviousParam,
  true /* shouldRender */,
  "hello" /* name */,
  5000 /* timeout */
);
```

#### 2.9.4 實際應用範例

```ts
// 好例子：布林參數需要註解
dialog.open(UserFormComponent, {
  width: '400px',
  /* disableClose= */ true,
  /* hasBackdrop= */ false
});

// 好例子：數字參數需要說明
setTimeout(
  () => this.refreshData(),
  /* delayMs= */ 3000
);

// 好例子：字串參數可能不明確
logger.log(
  message,
  /* level= */ 'error',
  /* category= */ 'auth'
);

// 不需要：參數意義已經很明確
calculateTotal(price, tax); // price 和 tax 很明確，不需要註解
setUserName('John Doe'); // 只有一個參數且意義明確
```

#### 2.9.5 Angular 特定範例

```ts
// Angular HTTP 呼叫
this.http.get('/api/users', {
  /* observe= */ 'response',
  /* responseType= */ 'json'
});

// Router navigation
this.router.navigate(['/dashboard'], {
  /* replaceUrl= */ true,
  /* skipLocationChange= */ false
});

// FormControl 設定
new FormControl('', {
  /* nonNullable= */ true,
  /* updateOn= */ 'blur'
});
```

---

> ### **重點總結**
>
> - 優先考慮重構為 interface，而非依賴參數註解
> - 當參數意義不明確時，使用參數名稱註解
> - 推薦格式：`/* paramName= */ value`
> - 保持檔案內格式一致性
> - 避免為明顯的參數添加不必要的註解

---

### 2.10 Angular Service 與 Component 的 API

- 輸入輸出需明確型別化（避免 `any`）
- Service 回傳值應使用 `Observable<T>`，避免 `Observable<any>`

```ts
// 避免
getUser(): Observable<any>;

// 推薦
getUser(): Observable<User>;
```

---

> ### **重點總結**
>
> - 所有 Component 和 Service 的 API 必須明確型別化
> - 避免在公開介面使用 `any` 型別
> - Service 方法回傳值使用具體的 `Observable<T>` 型別
> - Input/Output 屬性需要明確的型別定義

---

### 2.11 依賴注入：Constructor Injection vs inject()

Angular 提供兩種依賴注入方式，建議優先使用新的 `inject()` 函數（Angular 14+）。

#### 2.11.1 推薦：使用 inject() 函數

**優點：**

- **更簡潔**：減少 constructor 參數數量
- **更靈活**：可在任何地方使用（函數、初始化器等）
- **更好的 Tree-shaking**：未使用的服務更容易被移除
- **支援條件注入**：可根據條件決定是否注入

```ts
import { Component, inject } from "@angular/core";
import { UserService } from "./user.service";
import { Router } from "@angular/router";

@Component({
  selector: "app-user",
  template: `...`,
})
export class UserComponent {
  // 推薦：使用 inject() 函數
  private userService = inject(UserService);
  private router = inject(Router);

  // 可選依賴注入
  private logger = inject(LoggerService, { optional: true });

  ngOnInit() {
    this.userService.getUsers().subscribe((users) => {
      console.log(users);
    });
  }

  navigateToProfile() {
    this.router.navigate(["/profile"]);
  }
}
```

#### 2.11.2 條件和可選注入

```ts
@Component({...})
export class FeatureComponent {
  // 可選依賴：如果服務不存在也不會報錯
  private analytics = inject(AnalyticsService, { optional: true });

  // 條件注入：在特定環境下注入不同實作
  private apiService = inject(environment.production ? ProdApiService : DevApiService);

  // 使用 SkipSelf 避免注入自身
  private parentComponent = inject(ParentComponent, { skipSelf: true });

  trackEvent(event: string) {
    // 安全地使用可選依賴
    this.analytics?.track(event);
  }
}
```

#### 2.11.3 在函數中使用 inject()

```ts
// 在初始化函數中使用
function createUserValidator() {
  const userService = inject(UserService);

  return (control: AbstractControl) => {
    return userService.validateUser(control.value);
  };
}

@Component({...})
export class SignupComponent {
  // 在表單驗證器中使用注入的服務
  userForm = new FormGroup({
    username: new FormControl('', [
      Validators.required,
      createUserValidator() // 這裡會使用注入的 UserService
    ])
  });
}
```

#### 2.11.4 舊版：Constructor Injection

**仍可使用的場合：**

- 需要在 constructor 中立即使用服務
- 團隊尚未升級到 Angular 14+
- 需要與舊代碼保持一致

```ts
@Component({...})
export class UserComponent {
  // 舊做法：Constructor injection（仍然有效但不推薦）
  constructor(
    private userService: UserService,
    private router: Router,
    @Optional() private logger?: LoggerService
  ) {}

  ngOnInit() {
    this.userService.getUsers().subscribe(users => {
      console.log(users);
    });
  }
}
```

#### 2.10.5 混合使用的最佳實踐

```ts
@Component({...})
export class ModernComponent {
  // 推薦：大部分依賴使用 inject()
  private userService = inject(UserService);
  private router = inject(Router);

  // 可接受：少數需要在 constructor 中使用的依賴
  constructor(
    @Inject(DOCUMENT) private document: Document
  ) {
    // 只在確實需要在 constructor 中立即使用時才用 constructor injection
    this.setupDocumentListeners();
  }

  private setupDocumentListeners() {
    // 使用注入的 document
    this.document.addEventListener('click', this.handleClick);
  }
}
```

#### 2.10.6 注意事項

**inject() 的限制：**

- 只能在注入上下文中使用（component、directive、service、guard 等）
- 不能在異步回調中使用
- 必須在頂層作用域中呼叫

```ts
@Component({...})
export class ExampleComponent {
  private userService = inject(UserService); // 正確：頂層作用域

  ngOnInit() {
    // 錯誤：不能在方法內部使用 inject()
    // const service = inject(SomeService);

    setTimeout(() => {
      // 錯誤：不能在異步回調中使用 inject()
      // const anotherService = inject(AnotherService);
    }, 1000);
  }
}
```

---

> ### **重點總結**
>
> - **優先使用 `inject()` 函數**（Angular 14+）
> - 利用可選注入和條件注入提高靈活性
> - Constructor injection 僅在確實需要在建構函數中立即使用時採用
> - 注意 `inject()` 的使用限制和上下文要求
> - 保持團隊內一致的注入風格

---

---

## 3. Component 生命週期與最佳實踐

### 3.1 檔案與類別結構順序

Component 類別內容應按照以下順序組織，以提高程式碼可讀性和維護性：

```ts
@Component({
  selector: "app-user",
  templateUrl: "./user.component.html",
  styleUrls: ["./user.component.scss"],
  changeDetection: ChangeDetectionStrategy.OnPush, // 推薦明確設定
})
export class UserComponent implements OnInit, OnDestroy {
  // 1. Public properties (輸入輸出屬性)
  @Input() userId!: string;
  @Output() userSelected = new EventEmitter<User>();

  // 2. Public reactive properties
  public title = "User Management";
  public loading = signal(false);
  public users = signal<User[]>([]);

  // 3. Private properties and services
  private userService = inject(UserService);
  private router = inject(Router);
  private destroyRef = inject(DestroyRef);

  // 4. Constructor (僅在必要時使用)
  constructor() {
    // 只在需要立即初始化時使用 constructor
  }

  // 5. Lifecycle hooks (按執行順序排列)
  ngOnInit(): void {
    this.loadUsers();
  }

  ngAfterViewInit(): void {
    // View 初始化完成後的邏輯
  }

  ngOnDestroy(): void {
    // 清理邏輯（使用 takeUntilDestroyed 時通常不需要）
  }

  // 6. Public methods
  public onUserClick(user: User): void {
    this.userSelected.emit(user);
  }

  public refreshUsers(): void {
    this.loadUsers();
  }

  // 7. Private methods (最後)
  private loadUsers(): void {
    this.loading.set(true);
    this.userService
      .getUsers()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe({
        next: (users) => {
          this.users.set(users);
          this.loading.set(false);
        },
        error: (error) => this.handleError(error),
      });
  }

  private handleError(error: any): void {
    console.error("Failed to load users:", error);
    this.loading.set(false);
  }
}
```

### 3.2 生命週期 Hook 使用原則

#### 3.2.1 明確實作必要的介面

```ts
// 推薦：明確實作需要的生命週期介面
export class UserComponent implements OnInit, OnDestroy, AfterViewInit {
  ngOnInit(): void {
    // 組件初始化邏輯
  }

  ngAfterViewInit(): void {
    // View 和子組件初始化完成後的邏輯
  }

  ngOnDestroy(): void {
    // 清理邏輯
  }
}
```

#### 3.2.2 常用生命週期 Hook 的適用場景

```ts
export class ExampleComponent implements OnInit, AfterViewInit, OnDestroy {
  // ngOnInit: 組件屬性初始化、訂閱 Observable、HTTP 請求
  ngOnInit(): void {
    this.loadInitialData();
    this.setupFormValidation();
  }

  // ngAfterViewInit: DOM 操作、第三方函式庫初始化
  ngAfterViewInit(): void {
    this.initializeChartLibrary();
    this.setupDOMEventListeners();
  }

  // ngOnDestroy: 取消訂閱、清理計時器、移除事件監聽器
  ngOnDestroy(): void {
    this.clearIntervals();
    this.removeEventListeners();
  }
}
```

### 3.3 變更檢測策略

#### 3.3.1 強烈建議使用 OnPush

**為什麼必須使用 OnPush？**

Angular 的預設變更檢測策略（Default）會在每次可能發生變更的事件後檢查整個組件樹，這會造成嚴重的效能問題：

**Default 策略的問題：**

```ts
// 問題：預設策略會過度檢測
@Component({
  selector: "app-inefficient",
  template: `
    <div>{{ expensiveCalculation() }}</div>
    @for (item of items; track $index) {
    <div>{{ item.name }}</div>
    }
  `,
  // changeDetection: ChangeDetectionStrategy.Default // 預設值，效能差
})
export class InefficientComponent {
  items: Item[] = [];

  // 問題：每次變更檢測都會執行這個昂貴的計算
  expensiveCalculation(): string {
    console.log("昂貴的計算被執行了！"); // 可能每秒執行數十次
    return this.items
      .map((item) => item.value)
      .reduce((a, b) => a + b, 0)
      .toString();
  }
}
```

**OnPush 策略的優勢：**

1. **減少不必要的檢測**：只在輸入屬性變更、事件觸發或明確標記時檢測
2. **提升應用效能**：大幅減少 CPU 使用率，特別是在大型應用中
3. **強制最佳實踐**：促使開發者使用不可變資料和響應式程式設計
4. **更可預測的行為**：明確控制何時觸發變更檢測

```ts
@Component({
  selector: "app-optimized",
  template: `
    @for (item of items; track trackByFn($index, item)) {
    <div>{{ item.name }}</div>
    }
    <div>總計: {{ totalValue() }}</div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush, // 必須明確設定
})
export class OptimizedComponent {
  @Input() items: Item[] = [];

  // TrackBy 函數提升 @for 效能
  trackByFn(index: number, item: Item): string {
    return item.id;
  }

  // 使用 computed signal 替代昂貴的函數呼叫
  private itemsSignal = signal(this.items);
  protected totalValue = computed(() =>
    this.itemsSignal().reduce((sum, item) => sum + item.value, 0)
  );

  // 當 Input 變更時更新 signal
  ngOnChanges(changes: SimpleChanges): void {
    if (changes["items"]) {
      this.itemsSignal.set(this.items);
    }
  }
}
```

**OnPush 觸發變更檢測的條件：**

```ts
@Component({
  selector: "app-onpush-demo",
  template: `
    <button (click)="updateCounter()">計數器: {{ counter }}</button>
    <input [(ngModel)]="inputValue" placeholder="輸入文字" />
    <div>使用者: {{ user.name }}</div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OnPushDemoComponent {
  @Input() user!: User; // 1. Input 屬性變更會觸發檢測

  counter = 0;
  inputValue = "";

  private cdr = inject(ChangeDetectorRef);

  // 2. 組件內部事件會觸發檢測
  updateCounter(): void {
    this.counter++; // 這會自動觸發變更檢測
  }

  // 3. 手動觸發變更檢測
  manualUpdate(): void {
    // 在某些情況下需要手動標記檢測
    this.cdr.markForCheck();
  }

  ngOnInit(): void {
    // 4. 外部 Observable 需要手動觸發
    this.userService
      .getUserUpdates()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe((user) => {
        this.user = user;
        this.cdr.markForCheck(); // 必須手動標記
      });
  }
}
```

**效能比較示例：**

```ts
// 測試：1000 個組件的效能差異
@Component({
  selector: "app-performance-test",
  template: `
    <div class="stats">
      檢測次數: {{ detectionCount }}
      <button (click)="triggerUpdate()">觸發更新</button>
    </div>
    @for (item of items; track trackByFn($index, item)) {
    <app-child [data]="item"></app-child>
    }
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PerformanceTestComponent {
  items = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    value: `Item ${i}`,
  }));
  detectionCount = 0;

  ngDoCheck(): void {
    this.detectionCount++;
    // OnPush: 只在真正需要時增加
    // Default: 每次任何地方有事件都會增加
  }

  triggerUpdate(): void {
    // OnPush: 只有這個組件和其子組件會被檢測
    // Default: 整個應用的所有組件都會被檢測
    this.items = [...this.items]; // 不可變更新
  }

  trackByFn(index: number, item: any): number {
    return item.id;
  }
}
```

**最佳實踐：OnPush + 不可變資料**

```ts
@Component({
  selector: "app-immutable-example",
  template: `
    @for (todo of todos(); track trackByFn($index, todo)) {
    <div>
      <input
        type="checkbox"
        [checked]="todo.completed"
        (change)="toggleTodo(todo.id)"
      />
      {{ todo.title }}
    </div>
    }
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ImmutableExampleComponent {
  private todosSignal = signal<Todo[]>([]);
  protected todos = this.todosSignal.asReadonly();

  // 正確：使用不可變更新
  toggleTodo(id: number): void {
    const currentTodos = this.todosSignal();
    const updatedTodos = currentTodos.map((todo) =>
      todo.id === id
        ? { ...todo, completed: !todo.completed } // 建立新物件
        : todo
    );
    this.todosSignal.set(updatedTodos); // 設定新陣列
  }

  // 錯誤：可變更新（OnPush 不會檢測到）
  toggleTodoWrong(id: number): void {
    const todos = this.todosSignal();
    const todo = todos.find((t) => t.id === id);
    if (todo) {
      todo.completed = !todo.completed; // 直接修改物件
      // OnPush 不會檢測到這個變更！
    }
  }

  trackByFn(index: number, todo: Todo): number {
    return todo.id;
  }
}
```

**OnPush 的限制與解決方案：**

```ts
@Component({
  selector: "app-onpush-limitations",
  template: `
    <div>時間: {{ currentTime | date : "HH:mm:ss" }}</div>
    <div>狀態: {{ asyncData() || "Loading..." }}</div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class OnPushLimitationsComponent {
  currentTime = new Date();

  private cdr = inject(ChangeDetectorRef);
  private destroyRef = inject(DestroyRef);

  // 解決方案1：使用 Signal 處理非同步資料
  private asyncData = toSignal(
    this.dataService.getData().pipe(startWith(null))
  );

  ngOnInit(): void {
    // 解決方案2：需要定期更新時間的情況
    interval(1000)
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(() => {
        this.currentTime = new Date();
        this.cdr.markForCheck(); // 手動標記檢測
      });
  }
}
```

#### 3.3.2 使用 Signal 的現代做法

```ts
@Component({
  selector: "app-modern",
  template: `
    @for (item of items(); track trackByFn($index, item)) {
    <div>{{ item.name }}</div>
    }
    <p>總數: {{ itemCount() }}</p>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ModernComponent {
  // 使用 Signal 自動優化變更檢測
  protected items = signal<Item[]>([]);
  protected itemCount = computed(() => this.items().length);

  private itemService = inject(ItemService);

  ngOnInit(): void {
    // 將 Observable 轉為 Signal
    effect(() => {
      this.itemService
        .getItems()
        .pipe(takeUntilDestroyed(this.destroyRef))
        .subscribe((items) => this.items.set(items));
    });
  }

  trackByFn(index: number, item: Item): string {
    return item.id;
  }
}
```

### 3.4 表單處理最佳實踐

#### 3.4.1 Reactive Forms 與型別安全

```ts
interface UserForm {
  name: string;
  email: string;
  age: number;
}

@Component({...})
export class UserFormComponent implements OnInit {
  // 型別安全的表單定義
  userForm = new FormGroup<{[K in keyof UserForm]: FormControl<UserForm[K]>}>({
    name: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required, Validators.minLength(2)]
    }),
    email: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required, Validators.email]
    }),
    age: new FormControl(0, {
      nonNullable: true,
      validators: [Validators.min(0), Validators.max(120)]
    })
  });

  // 使用 Signal 追蹤表單狀態
  protected formValid = computed(() => this.userForm.valid);
  protected formValue = computed(() => this.userForm.value as UserForm);

  ngOnInit(): void {
    // 監聽表單變化
    this.userForm.valueChanges
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(value => {
        console.log('Form changed:', value);
      });
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      const formData = this.formValue();
      this.submitUser(formData);
    }
  }

  private submitUser(user: UserForm): void {
    // 提交邏輯
  }
}
```

### 3.5 錯誤處理模式

```ts
@Component({...})
export class UserComponent implements OnInit {
  protected users = signal<User[]>([]);
  protected loading = signal(false);
  protected error = signal<string | null>(null);

  private userService = inject(UserService);
  private destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.loading.set(true);
    this.error.set(null);

    this.userService.getUsers()
      .pipe(
        takeUntilDestroyed(this.destroyRef),
        catchError(error => {
          this.handleError(error);
          return of([]); // 返回空陣列作為備用值
        }),
        finalize(() => this.loading.set(false))
      )
      .subscribe(users => this.users.set(users));
  }

  private handleError(error: any): void {
    console.error('Failed to load users:', error);

    // 根據錯誤類型提供不同的使用者訊息
    if (error.status === 404) {
      this.error.set('找不到使用者資料');
    } else if (error.status >= 500) {
      this.error.set('伺服器發生錯誤，請稍後再試');
    } else {
      this.error.set('載入資料時發生錯誤');
    }
  }
}
```

### 3.6 組件間通訊最佳實踐

```ts
// 父子組件通訊
@Component({
  selector: "app-parent",
  template: `
    <app-child [data]="parentData()" (dataChange)="onChildDataChange($event)">
    </app-child>
  `,
})
export class ParentComponent {
  protected parentData = signal<string>("initial data");

  onChildDataChange(newData: string): void {
    this.parentData.set(newData);
  }
}

@Component({
  selector: "app-child",
  template: ` <button (click)="updateData()">Update</button> `,
})
export class ChildComponent {
  @Input() data!: string;
  @Output() dataChange = new EventEmitter<string>();

  updateData(): void {
    this.dataChange.emit("updated data");
  }
}
```

---

> ### **重點總結**
>
> - **明確的結構順序**：屬性 → 建構子 → 生命週期 → 方法
> - **使用 OnPush 變更檢測策略**提升效能
> - **優先使用 Signal**處理狀態管理
> - **型別安全的表單處理**
> - **完善的錯誤處理機制**
> - **清晰的組件間通訊模式**
> - **採用新的 Angular Control Flow 語法**提升效能和可讀性

---

### 3.7 Angular Control Flow 語法 (Angular 17+)

Angular 17 引入了全新的內建控制流語法，使用 `@` 符號提供更簡潔、效能更好的結構指令替代方案。

#### 3.7.1 新的控制流語法概述

**優點：**

- **更好的效能**：編譯器優化，無需載入額外的指令
- **更好的型別推斷**：TypeScript 能更準確地推斷型別
- **更簡潔的語法**：減少樣板程式碼
- **更好的 IDE 支援**：語法高亮和自動完成更準確

#### 3.7.2 @if 條件渲染

**新語法 (`@if`)：**

```html
<!-- 基本條件渲染 -->
@if (user(); as currentUser) {
<div class="user-info">
  <h2>{{ currentUser.name }}</h2>
  <p>{{ currentUser.email }}</p>
</div>
} @else {
<div class="loading">Loading user...</div>
}

<!-- 多重條件 -->
@if (userRole() === 'admin') {
<admin-panel></admin-panel>
} @else if (userRole() === 'moderator') {
<moderator-panel></moderator-panel>
} @else {
<user-panel></user-panel>
}

<!-- 複雜條件與型別保護 -->
@if (users() && users()!.length > 0; as userList) { @for (user of userList;
track user.id) {
<div>{{ user.name }}</div>
} }
```

**對比舊語法 (`*ngIf`)：**

```html
<!-- 舊語法：需要多個 ng-container -->
<ng-container *ngIf="user$ | async as currentUser; else loadingTemplate">
  <div class="user-info">
    <h2>{{ currentUser.name }}</h2>
    <p>{{ currentUser.email }}</p>
  </div>
</ng-container>
<ng-template #loadingTemplate>
  <div class="loading">Loading user...</div>
</ng-template>
```

#### 3.7.3 @for 迴圈渲染

**新語法 (`@for`)：**

```html
<!-- 基本迴圈 -->
@for (item of items(); track item.id) {
<div class="item">{{ item.name }}</div>
} @empty {
<div class="no-items">No items found</div>
}

<!-- 使用索引和其他變數 -->
@for (user of users(); track user.id; let i = $index, isFirst = $first, isLast =
$last) {
<div class="user-row" [class.first]="isFirst" [class.last]="isLast">
  {{ i + 1 }}. {{ user.name }}
</div>
}

<!-- 複雜的 trackBy 函數 -->
@for (item of complexItems(); track trackByFn($index, item)) {
<complex-item [data]="item"></complex-item>
}
```

**對比舊語法 (`*ngFor`)：**

```html
<!-- 舊語法 -->
<div *ngFor="let item of items; trackBy: trackByFn; let i = index">
  {{ item.name }}
</div>
<div *ngIf="items.length === 0">No items found</div>
```

#### 3.7.4 @switch 條件分支

**新語法 (`@switch`)：**

```html
<!-- 基本 switch -->
@switch (userStatus()) { @case ('active') {
<span class="status-active">✅ Active</span>
} @case ('inactive') {
<span class="status-inactive">⏸️ Inactive</span>
} @case ('suspended') {
<span class="status-suspended">🚫 Suspended</span>
} @default {
<span class="status-unknown">❓ Unknown</span>
} }

<!-- 複雜的 switch 與組件 -->
@switch (currentView()) { @case ('list') {
<user-list [users]="users()"></user-list>
} @case ('grid') {
<user-grid [users]="users()"></user-grid>
} @case ('table') {
<user-table [users]="users()" [columns]="tableColumns()"></user-table>
} @default {
<div class="error">Unknown view type</div>
} }
```

**對比舊語法 (`*ngSwitch`)：**

```html
<!-- 舊語法 -->
<div [ngSwitch]="userStatus">
  <span *ngSwitchCase="'active'" class="status-active">✅ Active</span>
  <span *ngSwitchCase="'inactive'" class="status-inactive">⏸️ Inactive</span>
  <span *ngSwitchCase="'suspended'" class="status-suspended">🚫 Suspended</span>
  <span *ngSwitchDefault class="status-unknown">❓ Unknown</span>
</div>
```

#### 3.7.5 與傳統結構指令的比較

| 特性            | 新語法 (@if/@for/@switch)   | 舊語法 (*ngIf/*ngFor/\*ngSwitch) |
| --------------- | --------------------------- | -------------------------------- |
| **效能**        | ✅ 編譯器優化，更快         | ❌ 運行時指令載入                |
| **型別推斷**    | ✅ 更準確的 TypeScript 支援 | ⚠️ 有限的型別推斷                |
| **Bundle 大小** | ✅ 更小的打包大小           | ❌ 需要載入指令                  |
| **語法簡潔性**  | ✅ 更直觀和簡潔             | ⚠️ 需要更多樣板程式碼            |
| **IDE 支援**    | ✅ 更好的語法高亮和補全     | ⚠️ 有限的 IDE 支援               |
| **學習曲線**    | ✅ 更容易理解               | ⚠️ 需要理解指令概念              |

#### 3.7.6 最佳實踐建議

**1. 優先使用新的控制流語法**

```html
<!-- 推薦：使用新語法 -->
@if (isLoading()) {
<loading-spinner></loading-spinner>
} @else if (hasError()) {
<error-message [error]="error()"></error-message>
} @else {
<main-content [data]="data()"></main-content>
}

<!-- 避免：混合新舊語法 -->
@if (isLoading()) {
<loading-spinner></loading-spinner>
} @if (!isLoading() && hasData()) {
<main-content></main-content>
}
```

**2. 善用 track 函數提升效能**

```html
<!-- 推薦：明確的 track 函數 -->
@for (item of items(); track item.id) {
<item-component [data]="item"></item-component>
}

<!-- 可接受：使用索引 track -->
@for (item of simpleItems(); track $index) {
<div>{{ item }}</div>
}
```

**3. 利用 @empty 提供更好的使用者體驗**

```html
<!-- 推薦：使用 @empty 區塊 -->
@for (user of users(); track user.id) {
<user-card [user]="user"></user-card>
} @empty {
<div class="empty-state">
  <p>沒有找到使用者</p>
  <button (click)="loadUsers()">重新載入</button>
</div>
}
```

**4. 結合 Signal 獲得最佳效能**

```ts
@Component({
  template: `
    @if (loading()) {
    <loading-spinner></loading-spinner>
    } @else { @for (user of filteredUsers(); track user.id) {
    <user-card [user]="user"></user-card>
    } @empty {
    <div>No users match your search</div>
    } }
  `,
})
export class UserListComponent {
  // 使用 Signal 與新控制流語法的完美組合
  protected users = signal<User[]>([]);
  protected searchTerm = signal("");
  protected loading = signal(false);

  // Computed signals 自動重新計算
  protected filteredUsers = computed(() =>
    this.users().filter((user) =>
      user.name.toLowerCase().includes(this.searchTerm().toLowerCase())
    )
  );
}
```

**5. 遷移策略**

```html
<!-- 階段性遷移：先遷移簡單的條件 -->
<!-- Phase 1: 遷移簡單的 *ngIf -->
@if (showContent) {
<div class="content">...</div>
}

<!-- Phase 2: 遷移複雜的 *ngFor -->
@for (item of items; track item.id) {
<item-component [data]="item"></item-component>
}

<!-- Phase 3: 遷移 *ngSwitch -->
@switch (viewMode) { @case ('list') { <list-view></list-view> } @case ('grid') {
<grid-view></grid-view> } @default { <default-view></default-view> } }
```

---

> ### **重點總結**
>
> - **優先使用新的控制流語法**（@if、@for、@switch）提升效能
> - **結合 Signal** 獲得最佳的響應式體驗
> - **善用 track 函數**優化列表渲染效能
> - **利用 @empty 區塊**提供更好的空狀態處理
> - **採用階段性遷移策略**從舊語法平滑過渡
> - **保持語法一致性**，避免新舊語法混用

---

---

## 4. RxJS 與訂閱管理 (Unsubscribing)

### 3.1 必須正確取消訂閱

- 未取消訂閱會造成 **記憶體洩漏 (Memory Leak)** 與 **效能問題**。\
- 尤其在 Component Destroy 後仍存活的訂閱，可能導致非預期的行為。

### 3.2 推薦做法

#### 使用 Signal (Angular 16+) - **最推薦**

- Signal 是 Angular 16+ 引入的響應式狀態管理方案，完全避免訂閱管理問題。

```ts
import { signal, computed, effect } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({...})
export class UserComponent {
  // 方法1：直接使用 Signal
  private userSignal = signal<User | null>(null);

  // 方法2：將 Observable 轉換為 Signal
  private user$ = this.userService.getUser();
  private userFromObservable = toSignal(this.user$);

  // Computed Signal
  readonly userName = computed(() => this.userSignal()?.name ?? 'Anonymous');

  ngOnInit() {
    // 使用 effect 處理副作用，無需手動取消訂閱
    effect(() => {
      const user = this.userSignal();
      if (user) {
        console.log('User loaded:', user.name);
      }
    });
  }
}
```

```html
<!-- Template 中直接使用 Signal -->
@if (userSignal(); as user) {
<div>{{ user.name }}</div>
}
<p>用戶名稱: {{ userName() }}</p>
```

#### 使用 `async pipe`

- Angular 內建 `async` pipe 會自動管理訂閱生命週期。

```html
@if (user$ | async; as user) {
<div>{{ user.name }}</div>
}
```

#### 使用 `takeUntilDestroyed` (Angular 16+)

- 官方推薦用法，結合 `DestroyRef` 自動在 component destroy 時清理。

```ts
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { Component, inject } from '@angular/core';

@Component({...})
export class UserComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.userService.getUser()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(user => console.log(user));
  }
}
```

#### 使用 `takeUntil` (舊版 Angular)

```ts
private destroy$ = new Subject<void>();

ngOnInit() {
  this.userService.getUser()
    .pipe(takeUntil(this.destroy$))
    .subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

### 3.3 禁止做法

```ts
// 避免：手動管理太多 Subscription 物件
ngOnInit() {
  this.sub = this.userService.getUser().subscribe();
}

ngOnDestroy() {
  this.sub.unsubscribe(); // 容易遺漏
}
```

### 3.4 Signal vs RxJS 選擇建議

| 情境            | 推薦方案                  | 原因                   |
| --------------- | ------------------------- | ---------------------- |
| 簡單狀態管理    | Signal                    | 無需訂閱管理，語法簡潔 |
| 複雜資料流處理  | RxJS + takeUntilDestroyed | 豐富的操作符           |
| HTTP 請求轉狀態 | toSignal()                | 結合兩者優勢           |
| 表單驗證        | Signal + computed         | 響應式計算             |

---

> ### **重點總結**
>
> - **必須正確取消訂閱**，避免記憶體洩漏
> - **優先使用 Signal**（Angular 16+），無需手動管理訂閱
> - 使用 `async pipe` 自動管理 Observable 生命週期
> - 使用 `takeUntilDestroyed` 結合 DestroyRef 自動清理
> - 避免手動管理多個 Subscription 物件
> - 根據使用情境選擇 Signal 或 RxJS 解決方案

---
