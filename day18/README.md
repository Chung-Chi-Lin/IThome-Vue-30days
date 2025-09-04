# 【Day 18】打造你的「型別安全」武功祕笈：為 Composable 加上型別

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 武學大師！在 Day 10，我們學會了如何打造自己的 `Composable` (自訂 Hook)，把可複用的邏輯封裝成一本「武功祕笈」。當時，我們寫了一個 `useMouse` 的祕笈，它能用，但就像一本手抄本，有些地方語焉不詳。

今天，在我們掌握了 TypeScript 這門「內功心法」之後，是時候回去重新精修這本祕笈了！我們要為 `Composable` 加上完整的型別定義，把它從一本「手抄本」，升級成一本圖文並茂、註解清晰、任何人拿到都能練成絕世武功的「**官方出版的型別安全武功祕笈**」！

## 为什么要為 Composable 加上型別？

很簡單，一本好的武功祕笈，必須清楚地告訴你：

1.  **練功需求 (參數型別)**：練此功需要什麼樣的體質、什麼樣的道具？（函式需要什麼參數？）
2.  **功成效果 (回傳值型別)**：練成之後，你能獲得什麼能力？（函式會回傳什麼資料和方法？）

有了型別，你的 `Composable` 就不再是一個黑盒子。使用者（包括未來的你）在調用它時，IDE 會給予完整的提示，從根源上杜絕「練功走火入魔」（傳錯參數、用錯回傳值）的風險。

## 實戰：精修 `useCounter` 祕笈

讓我們從一個簡單的計數器 `Composable` 開始。

### 1. 定義「練功需求」(參數型別)

我們的計數器希望可以接收一個「初始值」。

```typescript
// composables/useCounter.ts
import { ref } from 'vue';

// 直接在參數上定義型別
export function useCounter(initialValue: number = 0) { // 給定預設值
  const count = ref(initialValue);

  const increment = () => {
    count.value++;
  };

  return { count, increment };
}
```

現在，如果有人想練這個武功，但給錯了道具（傳入非數字的值），TS 教頭會立刻阻止他。

```typescript
// 在某個組件中
import { useCounter } from './composables/useCounter';

// const { count, increment } = useCounter('ten'); // TS 錯誤：想用字串當初始值？門都沒有！
```

### 2. 定義「功成效果」(回傳值型別)

我們的 `useCounter` 會回傳一個物件，裡面有 `count` 和 `increment`。雖然 TS 大多時候能自己推斷出來，但為了讓這本「祕笈」更具權威性和可讀性，我們最好手動定義回傳值的「規格」。

```typescript
// composables/useCounter.ts
import { ref, type Ref } from 'vue'; // 導入 Ref 型別

// 步驟 1: 定義回傳物件的介面 (Interface)
export interface UseCounterReturn {
  count: Ref<number>;
  increment: () => void;
}

// 步驟 2: 在函式簽名中，明確宣告回傳值必須符合這個介面
export function useCounter(initialValue: number = 0): UseCounterReturn {
  const count = ref(initialValue);

  const increment = () => {
    count.value++;
  };

  // TS 會確保你回傳的東西，不多不少，剛好符合 UseCounterReturn 的規格
  return { count, increment };
}
```

現在，當其他人在組件中使用 `useCounter` 時，IDE 會提供完美的自動補全，因為它手裡拿著我們定義好的 `UseCounterReturn` 這份「說明書」。

## 究極進化：使用泛型打造 `useFetch` 祕笈

現在來挑戰一個更厲害的武功：`useFetch`。這個 `Composable` 用來獲取遠端資料，但問題是，我們事先不知道會獲取到什麼樣的資料，可能是用戶列表，也可能是一篇文章。

這時候，武學中的上乘心法——**泛型 (Generics)** 就該登場了！泛型就像是祕笈裡的一個「**心法口訣**」，讓這本祕笈可以適用於各種情況。

```typescript
// composables/useFetch.ts
import { ref, onMounted, type Ref } from 'vue';

// T 是一個「型別變數」，由使用者在調用時決定
export function useFetch<T>(url: string) {
  const data = ref<T | null>(null) as Ref<T | null>; // 資料，可能是 T 型別，或一開始是 null
  const error = ref<Error | null>(null); // 錯誤物件
  const isLoading = ref(true); // 是否正在加載

  onMounted(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      data.value = await response.json();
    } catch (e: any) {
      error.value = e;
    } finally {
      isLoading.value = false;
    }
  });

  // 回傳值的型別也會跟著 T 變化
  return { data, error, isLoading };
}
```

**如何使用這本「泛用祕笈」？**

```typescript
// 在組件中
import { useFetch } from './composables/useFetch';

// --- 練習獲取用戶列表 --- 
interface User { id: number; name: string; }

// 告訴 useFetch，這次我們要抓的資料是 User 陣列 (User[])
const { data: users, error: userError } = useFetch<User[]>('https://api.example.com/users');
// 此時 `users` 的型別就是 Ref<User[] | null>，IDE 能完美提示陣列方法

// --- 練習獲取單一文章 --- 
interface Post { title: string; body: string; }

// 告訴 useFetch，這次我們要抓的是單一 Post 物件
const { data: post, error: postError } = useFetch<Post>('https://api.example.com/posts/1');
// 此時 `post` 的型別就是 Ref<Post | null>，IDE 能完美提示 title, body 等屬性
```

看到了嗎？透過泛型 `<T>`，我們的 `useFetch` 變得無比強大且靈活，同時保持了 100% 的型別安全！

## 總結

今天我們將 `Composable` 提升到了專業水準。一本型別安全的 `Composable` 應該具備：

-   **明確的參數型別**：定義了它的「輸入」。
-   **明確的回傳值型別**：定義了它的「輸出」，通常會為此建立一個 `interface`。
-   **善用泛型 `<T>`**：當 `Composable` 需要處理不確定的資料型別時，泛型是你的最佳武器。

從現在開始，你寫的每一個 `Composable` 都應該是一本值得信賴、廣為流傳的「武功祕笈」，而不是只有你自己才看得懂的筆記！

## 本篇自我挑戰

-   **今日挑戰**：回到 Day 10，把你當時寫的 `useMouse` 或其他 `Composable` 拿出來。為它的參數和回傳值加上完整的型別定義。你能為它建立一個 `interface` 來描述回傳值的結構嗎？
-   **反思**：在 `useFetch` 的例子中，如果我們不使用泛型，`data` 的型別可能就只能是 `any`。思考一下，這會對使用這個 `Composable` 的人造成多大的困擾？

本日關鍵字回顧

-   **Composable**: 可複用的邏輯單元，我們的「武功祕笈」。
-   **參數型別**: Composable 的輸入規格。
-   **回傳值型別**: Composable 的輸出規格，通常用 `interface` 定義。
-   **泛型 (Generics)**: 使用 `<T>` 讓函式或介面可以處理多種型別，增加靈活性和複用性。

明天，我們將把目光從 `script` 移到 `style`，學習如何在 Vue 中整合強大的 SCSS，並使用 CSS Modules 為你的樣式穿上「獨立盔甲」，徹底告別樣式衝突的噩夢！
