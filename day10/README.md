# 【Day 10】打造自己的 `Composable`：Vue 邏輯複用的魔法工具箱

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！前幾天我們深入探討了 `ref`、`reactive`、`computed` 和 `watch` 這些響應式魔法工具，它們讓我們的數據能夠「活」起來。今天，我們要學習如何將這些魔法工具打包，打造出屬於自己的「魔法工具箱」——也就是 Vue 3 中最酷炫的邏輯複用機制：**`Composable` (自訂 Hook)**！

還記得 Day 1 我們提到 Composition API 解決了 Options API 邏輯分散的問題嗎？`Composable` 就是 Composition API 實現「邏輯內聚」和「邏輯複用」的終極武器。它就像一個可以隨插即用的「功能模組」，讓你把那些重複的、複雜的邏輯抽離出來，變成一個個獨立、可測試、可複用的函式。從此，你的程式碼將告別複製貼上，變得更加優雅、高效！

## 什麼是 `Composable`？為什麼需要它？

`Composable` 本質上就是一個**函式**，它利用 Vue Composition API 的能力，封裝了響應式狀態、相關的邏輯和生命週期操作。它的命名通常以 `use` 開頭（這是一個約定俗成的慣例，就像 React 的 `use` Hook 一樣）。

**為什麼需要它？**

想像一下，你的應用程式裡有很多地方都需要用到「追蹤滑鼠位置」的功能，或者「處理表單輸入」的邏輯。如果每次都複製貼上同一段程式碼，不僅冗餘，而且一旦需要修改，你得改好幾個地方，簡直是惡夢！

`Composable` 就是來解決這個問題的。它讓你把這些重複的邏輯抽離出來，變成一個獨立的函式。當你需要這個功能時，只需要呼叫這個函式，就像從工具箱裡拿出一個工具一樣方便！

## 打造你的第一個 `Composable`：以「滑鼠追蹤器」為例

還記得 Day 1 我們用來對比 Options API 和 Composition API 的「滑鼠追蹤器」嗎？現在，我們就把它變成一個 `Composable`！

### 1. 建立 `Composable` 檔案

通常，我們會把 `Composable` 放在一個單獨的檔案裡，例如 `src/composables/useMouse.js` (或 `.ts` 如果你使用 TypeScript)。

```javascript
// src/composables/useMouse.js
import { ref, onMounted, onUnmounted } from 'vue';

export function useMouse() {
  const x = ref(0);
  const y = ref(0);

  function update(event) {
    x.value = event.pageX;
    y.value = event.pageY;
  }

  onMounted(() => window.addEventListener('mousemove', update));
  onUnmounted(() => window.removeEventListener('mousemove', update));

  return { x, y }; // 記得回傳你希望組件能使用的響應式數據或函式
}
```

### 2. 在組件中使用 `Composable`

現在，任何組件都可以輕鬆地使用這個 `useMouse` 函式了！

```html
<!-- src/components/MouseTracker.vue -->
<script setup>
import { useMouse } from '../composables/useMouse';

const { x, y } = useMouse(); // 呼叫你的 Composable
</script>

<template>
  <h1>滑鼠位置追蹤器</h1>
  <p>X: {{ x }}</p>
  <p>Y: {{ y }}</p>
</template>
```

> 看到沒？原本散落在組件各處的滑鼠追蹤邏輯，現在被漂亮地封裝在 `useMouse` 裡。組件本身變得非常乾淨，只負責使用這個功能，而不需要關心它是如何實現的。這就是「關注點分離」的極致體現！

## `Composable` 的超能力：輸入與輸出

`Composable` 不僅可以封裝邏輯，它還可以接收參數作為「輸入」，並返回響應式數據或函式作為「輸出」。這讓它變得更加靈活和強大！

### 帶有輸入的 `Composable`：以「計數器」為例

我們可以建立一個可自訂初始值的計數器 `Composable`。

```javascript
// src/composables/useCounter.js
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);

  const increment = () => {
    count.value++;
  };

  const decrement = () => {
    count.value--;
  };

  const doubleCount = computed(() => count.value * 2);

  return { count, increment, decrement, doubleCount };
}
```

在組件中使用：

```html
<!-- src/components/MyCounter.vue -->
<script setup>
import { useCounter } from '../composables/useCounter';

const { count, increment, decrement, doubleCount } = useCounter(10); // 設定初始值為 10
const { count: anotherCount, increment: anotherIncrement } = useCounter(); // 使用預設初始值 0
</script>

<template>
  <div>
    <h2>我的計數器</h2>
    <p>Count: {{ count }} (Double: {{ doubleCount }})</p>
    <button @click="increment">增加</button>
    <button @click="decrement">減少</button>
  </div>

  <div>
    <h2>另一個計數器</h2>
    <p>Count: {{ anotherCount }}</p>
    <button @click="anotherIncrement">增加</button>
  </div>
</template>
```

> 看到沒？你可以根據需要傳入不同的參數，創建出多個獨立的計數器實例，而且它們之間互不影響！這就是 `Composable` 實現邏輯複用的精髓。

## 小技巧與注意事項：

*   **命名慣例**：`Composable` 函式通常以 `use` 開頭，這是一個強烈的慣例，有助於識別和理解程式碼。當你看到 `useXxx`，你就知道它是一個 `Composable`，裡面可能包含了響應式狀態和生命週期鉤子。
*   **只在 `setup()` 或其他 `Composable` 中呼叫**：`Composable` 函式必須在 Vue 組件的 `setup()` 函式中，或者在另一個 `Composable` 函式中同步呼叫。這是因為它們依賴於 Vue 內部的一些上下文，如果不在正確的環境中呼叫，響應式功能可能無法正常工作。
*   **返回響應式數據**：`Composable` 應該返回響應式數據（`ref` 或 `reactive` 物件），這樣組件才能響應這些數據的變化。如果你返回的是普通 JavaScript 變數，它們將失去響應性。
*   **TypeScript 支援**：`Composable` 與 TypeScript 是天作之合！為你的 `Composable` 加上型別定義，可以大大提升程式碼的健壯性和開發體驗。Day 18 我們會深入探討如何為 `Composable` 加上完整的型別定義。

## 本篇自我挑戰

-   **今日挑戰**：回想你過去寫過的一些重複性邏輯（例如：處理表單輸入、管理一個開關狀態、或是一個簡單的計時器）。嘗試將其中一個邏輯抽離出來，寫成一個 `Composable` 函式，並在兩個不同的組件中引用它。觀察程式碼的變化和複用性。
-   **反思**：你覺得 `Composable` 如何改變了你組織和複用程式碼的方式？它解決了你過去在大型專案中遇到的哪些痛點？

## 總結

今天我們學習了 Vue 3 中強大的邏輯複用機制——`Composable` (自訂 Hook)。它讓我們能夠將複雜的、重複的邏輯封裝成獨立、可複用的函式，大大提升了程式碼的組織性、可讀性和可維護性。從此，你的 Vue 應用程式將變得更加模組化，開發效率也將更上一層樓！

本日關鍵字回顧

-   **`Composable`**: 封裝響應式邏輯的函式，通常以 `use` 開頭。
-   **自訂 Hook**: `Composable` 的另一種稱呼。
-   **邏輯複用**: 避免重複程式碼，提升開發效率。
-   **邏輯內聚**: 將相關邏輯集中在一起。
-   **`use` 慣例**: `Composable` 函式的命名約定。
-   **關注點分離**: 讓組件只關注 UI 呈現，邏輯交由 `Composable`。

明天，我們將離開組件的內部世界，開始探索 Vue 應用程式的「導航系統」——Vue Router，學習如何管理前端路由！
