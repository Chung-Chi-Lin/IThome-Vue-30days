# 【Day 16】組件的「身分證」與「麥克風」：精通 defineProps 與 defineEmits

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

嘿，各位紀律嚴明的 Vue 開發者！昨天我們成功地讓 TypeScript 這位嚴格的架構師進駐了我們的專案。今天，我們要來學習如何使用 TS 的超強武器，來規範組件之間最重要的溝通橋樑：**Props** 和 **Emits**。

想像一下，父子組件的關係就像一個「公司」和「外包專家」。

-   **Props**：是公司（父組件）交給專家（子組件）的「**工作委託單**」。
-   **Emits**：是專家（子組件）完成任務後，用來向公司報告進度的「**專用對講機**」。

在純 JS 的世界裡，這份委託單可能是手寫的，字跡潦草，對講機也可能有很多雜音。但有了 TypeScript，我們能把委託單變成一份「**具有法律效力的合約**」，把對講機升級成「**加密的軍用通訊頻道**」！

## defineProps：給你的組件一張「數位身分證」

`defineProps` 是子組件用來聲明它需要從父親那裡接收什麼資料的。在 `<script setup>` 中，它是個「編譯期宏」，你不需要 `import` 就能直接用。

### 傳統作法 (執行時期驗證)

在 JS 中，我們通常這樣定義 props，它會在瀏覽器執行時才檢查型別。

```javascript
// JS 的「手寫委託單」
defineProps({
  name: {
    type: String,
    required: true
  },
  age: {
    type: Number,
    default: 18
  }
});
```

這不錯，但如果傳錯了，要等到執行時才會在 console 看到警告。

### TypeScript 作法 (編譯時期驗證)

有了 TS，我們可以使用泛型語法，這就像是直接給組件一張「數位身分證」。

```html
<script setup lang="ts">
// 定義一張名為 "Props" 的身分證規格
interface Props {
  name: string;       // 姓名，必須是字串
  age?: number;       // 年齡，數字，可選 (注意 ? 符號)
  hobbies: string[];  // 興趣，必須是字串陣列
}

// 使用 withDefaults 來搭配泛型 defineProps，並提供預設值
// 這會回傳一個經過處理的 props 物件，所有屬性都是響應式的
const props = withDefaults(defineProps<Props>(), {
  age: 20
});
</script>
```

**這有什麼好處？**

當父組件在使用這個子組件時，如果它試圖傳遞不符合 `Props` 介面規格的資料，VS Code 和 `vue-tsc` 會在**你寫程式的時候**就直接大聲喝止！

```html
<!-- ParentComponent.vue -->
<template>
  <!-- 試圖傳遞一個數字作為 name -->
  <UserProfile :name="123" :hobbies="['coding']" />
  <!-- TS 錯誤：Type 'number' is not assignable to type 'string'. -->

  <!-- 忘記傳遞必要的 hobbies -->
  <UserProfile name="Alice" />
  <!-- TS 錯誤：Property 'hobbies' is missing in type ... -->
</template>
```

看到了嗎？錯誤在開發階段就被攔截了，這就是「型別安全」的威力！

## defineEmits：設定你的「加密通訊頻道」

`defineEmits` 讓我們定義子組件可以觸發哪些事件，以及這些事件可以攜帶什麼樣的資料。

### 傳統作法

```javascript
// 宣告一個叫做 'update-name' 的頻道
const emit = defineEmits(['update-name']);

// 使用頻道，但沒人知道該傳什麼資料
emit('update-name', 'Bob');
```

### TypeScript 作法

TS 允許我們用一種特殊的函式簽名語法，來精確定義每個「頻道」的規格。

```html
<script setup lang="ts">
// 宣告一個 emit 函式，並定義其擁有的頻道規格
const emit = defineEmits<{
  // 頻道 1: 'change', 攜帶兩個參數 (id: 數字, status: 字串)
  (e: 'change', id: number, status: string): void;

  // 頻道 2: 'update', 只攜帶一個參數 (value: 字串)
  (e: 'update', value: string): void;
}>();

function handleUpdate() {
  // 正確使用：在 'change' 頻道發送一個數字和一個字串
  emit('change', 123, 'completed');

  // 錯誤示範：在 'update' 頻道發送了一個數字，TS 會立刻報錯！
  // emit('update', 456);
  // TS 錯誤：Argument of type 'number' is not assignable to parameter of type 'string'.
}
</script>
```

這種寫法雖然看起來有點奇怪，但它極其強大。它等於是給你的「對講機」安裝了聲紋辨識和內容審查系統，確保你只能在指定的頻道上，說出符合格式的訊息。

## 總結

今天，我們學會了如何用 TypeScript 來武裝 `defineProps` 和 `defineEmits`。這就像是為組件間的溝通建立了一套嚴謹的「通訊協定」。

-   **`defineProps<T>()`**：定義了組件的「輸入規格」，像一張不可偽造的數位身分證。
-   **`withDefaults`**：為這張身分證上的可選欄位提供預設值。
-   **`defineEmits<T>()`**：定義了組件的「輸出規格」，像一組加密的、有嚴格格式要求的通訊頻道。

掌握了這兩樣武器，你的組件將變得前所未有的穩定和可靠，團隊協作的效率也會大大提升！

## 本篇自我挑戰

-   **今日挑戰**：找一個你之前寫的、有 props 和 emits 的父子組件。嘗試使用 `defineProps<T>()` 和 `defineEmits<T>()` 來為它們加上完整的型別定義。看看父組件是否會因為你傳遞了錯誤的 props 而報錯？
-   **反思**：你覺得 `defineEmits` 的語法是不是有點反直覺？但它帶來的好處（在 `emit` 時的型別檢查）是否值得這個學習成本？

本日關鍵字回顧

-   **`defineProps<T>()`**: 編譯期的泛型宏，用來定義 props 型別。
-   **`defineEmits<T>()`**: 編譯期的泛型宏，用來定義 emits 型別。
-   **`withDefaults`**: 搭配 `defineProps<T>()` 使用，提供 props 的預設值。
-   **編譯期宏 (Compiler Macro)**: Vue 特供、無需導入、在編譯時處理的特殊函式。
-   **型別安全 (Type Safety)**: 在編譯階段就確保型別正確性，避免執行時錯誤。

明天，我們將探討 TypeScript 的「讀心術」，看看它是如何為 `ref` 和 `reactive` 這些響應式資料自動推斷型別的！
