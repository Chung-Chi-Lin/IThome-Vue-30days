# 【Day 12】跨組件的橋樑：Pinia 狀態管理入門

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天，我們用 Vue Router 成功地將我們的應用程式劃分成了多個頁面，就像蓋好了一棟有許多房間的房子。但很快地，我們就會遇到新的問題：

- Navbar 組件需要知道使用者是否登入，來決定要顯示「登入」還是「會員頭像」。
- 商品列表頁把商品加入了購物車，但購物車圖示（在另一個組件）的數量卻沒有更新。
- 使用者在設定頁面切換了「夜間模式」，整個網站的佈景主題都需要跟著改變。

這些「**跨組件的共享狀態**」如果只靠 `Props` 和 `Emit` 來層層傳遞，當應用程式一大，就會變成一場災難，我們稱之為「**屬性鑽鑿 (Prop Drilling)**」。

為了解決這個問題，我們需要一個「**中央儲藏室**」來統一管理這些共享的狀態。今天，我們就來認識 Vue 3 生態系中最受歡迎的狀態管理庫：**Pinia**！

## 為什麼需要狀態管理？

想像一下，如果你的 `App.vue` 有一個 `isLoggedIn` 的狀態，你需要把它傳給 `TheHeader` 組件，`TheHeader` 再傳給 `UserMenu` 組件，`UserMenu` 再傳給 `Avatar` 組件... 這條鏈路只要中間斷了一層，資料就傳不過去了。

如果 `Avatar` 組件想改變登入狀態，又得反向地用 `Emit` 一層一層往上傳遞事件，維護起來非常痛苦。

![Prop Drilling](https://i.imgur.com/E13b3q9.png)

**狀態管理庫 (State Management Library)** 的作用，就是提供一個全域的、集中的地方來存放你的應用程式狀態。任何組件都可以直接從這個「中央儲藏室」讀取或修改資料，而不需要關心組件之間的層級關係。

![Pinia Store](https://i.imgur.com/G5g4gC1.png)

## Pinia 的核心概念

Pinia 的設計非常簡潔，被譽為「Vuex 的實質性繼任者」。它的 API 設計非常貼近 Vue 3 的 Composition API，學習起來非常直觀。你只需要掌握它的三大核心元素：`State`、`Getters` 和 `Actions`。

### 1. 定義一個 Store

一個 "Store" 就是一個獨立的狀態儲藏室，例如 `userStore`、`cartStore` 等。我們使用 `defineStore` 函式來定義它。

**`stores/counter.js`**
```javascript
import { defineStore } from 'pinia';

// `defineStore` 的第一個參數是這個 store 的唯一 ID
export const useCounterStore = defineStore('counter', {
  // State: 存放狀態的地方，必須是一個回傳物件的函式
  state: () => ({
    count: 0,
    name: 'Eduardo',
  }),
  
  // Getters: 如同組件中的 computed，用來包裝 state
  getters: {
    doubleCount: (state) => state.count * 2,
  },

  // Actions: 如同組件中的 methods，用來修改 state
  actions: {
    increment() {
      // 在 action 中，你可以透過 `this` 來存取 state
      this.count++;
    },
    randomizeCounter() {
      this.count = Math.round(100 * Math.random());
    },
  },
});
```

### 2. 在組件中使用 Store

在任何組件的 `<script setup>` 中，你只需要像呼叫一個 Composable 函式一樣，就能取得 Store 的實例。

**`CounterComponent.vue`**
```html
<script setup>
import { useCounterStore } from '../stores/counter';

// 取得 store 的實例
const counterStore = useCounterStore();
</script>

<template>
  <div>
    <!-- 直接存取 state 和 getters -->
    <p>Current Count: {{ counterStore.count }}</p>
    <p>Double Count: {{ counterStore.doubleCount }}</p>
    
    <!-- 呼叫 actions -->
    <button @click="counterStore.increment">Increment</button>
  </div>
</template>
```
> 看到了嗎？沒有任何 Props 或 Emit，`CounterComponent` 直接就和 `counter` store 溝通了。這就是 Pinia 的魅力：直觀、型別安全、且與 Vue Devtools 完美整合。

## 本篇自我挑戰

- **思考一：設計購物車 Store**
  試著設計一個 `cartStore`。它應該包含：
  1.  `state`：一個 `items` 陣列，用來存放商品物件。
  2.  `getters`：一個 `totalPrice`，用來計算購物車中所有商品的總價。
  3.  `actions`：一個 `addItem(item)` 方法，用來將商品加入 `items` 陣列。

- **思考二：本地狀態 vs. 全域狀態**
  什麼時候你應該使用 Pinia 來管理狀態？什麼時候又該使用組件自己的本地狀態 (`ref`, `reactive`)？有沒有一個判斷的標準？

## 總結

今天我們學習了如何使用 Pinia 來解決跨組件狀態共享的難題。Pinia 以其簡潔的 API 和強大的功能，成為了 Vue 3 開發的首選狀態管理方案。

- **Prop Drilling**：只用 Props/Emit 在深層組件間傳遞狀態的痛苦過程。
- **Pinia**：Vue 的官方狀態管理庫，提供一個中央化的方式來管理共享狀態。
- **Store**：一個獨立的狀態單元，由 `defineStore` 創建。
- **`state`**：Store 的核心資料，必須是函式回傳的物件。
- **`getters`**：Store 的計算屬性，用來衍生出新的狀態。
- **`actions`**：Store 的方法，用來執行同步或非同步操作來修改狀態。

我們現在可以管理來自客戶端的狀態了，但真實世界的應用程式，資料大多來自遠端的伺服器。明天，我們將學習如何**串接 API**，並將非同步獲取的資料，優雅地整合進我們的 Pinia store 中。敬請期待！
