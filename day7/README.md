# 【Day 7】Composition API 入門：`setup` 語法糖，讓你的 Vue 程式碼更優雅！

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！昨天我們深入探索了組件的生命週期，學會了如何掌握組件的「生老病死」。今天，我們要迎接 Vue 3 的真正明星——**Composition API**，特別是它最閃耀的搭檔：**`<script setup>` 語法糖**！

還記得 Day 1 我們抱怨 Vue 2 Options API 程式碼容易「邏輯分散」的困擾嗎？就像在玩「打地鼠」一樣，一個功能相關的程式碼散落在 `data`、`methods`、`computed` 各處。Composition API 的出現，就是來終結這場「打地鼠」遊戲的！而 `<script setup>` 更是把這場革命推向了高潮，它讓 Composition API 的寫法變得前所未有的簡潔和直觀。

準備好告別過去的繁瑣，迎接更清晰、更優雅的程式碼組織方式了嗎？

## Composition API 是什麼？為什麼需要它？

簡單來說，Composition API 是一種全新的程式碼組織方式。它不再要求你把程式碼按照 `data`、`methods`、`computed` 這些「選項類型」來分類，而是鼓勵你把**同一個功能的邏輯**（包括它的資料、方法、計算屬性、監聽器、生命週期鉤子等）都放在一起。這就是所謂的「**邏輯內聚**」。

想像一下，你以前整理房間，是把所有衣服放一個櫃子，所有書放一個櫃子，所有玩具放一個櫃子。現在 Composition API 告訴你：你可以把「出門」需要的所有東西（衣服、鑰匙、錢包、手機）都放在一個「出門專用包」裡。是不是方便多了？

**它解決了 Options API 的幾個痛點：**
1.  **邏輯分散**：大型組件中，一個功能散落在各處，難以維護。
2.  **`this` 的困惑**：在 Options API 中，`this` 的指向有時會讓人摸不著頭緒，尤其是在回調函式中。Composition API 則避免了 `this` 的使用，讓程式碼更清晰。
3.  **更好的 TypeScript 支援**：由於 Composition API 的設計更符合 JavaScript 的模組化特性，TypeScript 能夠提供更精準的型別推斷和自動補全，開發體驗絲滑無比。

## `<script setup>` 語法糖：Composition API 的最佳拍檔

`<script setup>` 是 Vue 3.2 引入的一個編譯時語法糖，它讓 Composition API 的使用變得極其簡潔和高效。它就像一個魔法開關，打開後，你就可以直接在 `<script setup>` 區塊內寫 Composition API 的程式碼，而不需要額外的 `setup()` 函式包裹。

**它的魔法在哪裡？**

*   **自動暴露**：在 `<script setup>` 中定義的變數、函式、`import` 的內容，都會自動暴露給模板使用，你不需要再手動 `return` 一堆東西了！
*   **頂層 `await`**：你可以在 `<script setup>` 中直接使用 `await`，這對於非同步資料的獲取非常方便，不需要額外的 `async` 函式包裹。
*   **更少的樣板程式碼**：告別 `setup() { return { ... } }` 的繁瑣，程式碼更簡潔、更易讀。

**來看看對比，感受一下它的魅力：**

**Vue 2 (Options API) - 計數器範例**
```javascript
// MyCounter.vue (Options API)
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    increment() {
      this.count++;
    }
  }
};
```

**Vue 3 (Composition API without `<script setup>`) - 計數器範例**
```javascript
// MyCounter.vue (Composition API without <script setup>)
import { ref } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const increment = () => {
      count.value++;
    };
    return {
      count,
      increment
    };
  }
};
```

**Vue 3 (Composition API with `<script setup>`) - 計數器範例**
```javascript
// MyCounter.vue (Composition API with <script setup>)
<script setup>
import { ref } from 'vue';

const count = ref(0);
const increment = () => {
  count.value++;
};
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

> 看到沒？有了 `<script setup>`，程式碼是不是瞬間清爽了許多？就像把所有東西都攤開在桌面上，一目瞭然，再也不用在 `data`、`methods` 之間跳來跳去了！

## 如何組織你的程式碼？「功能」優先！

Composition API 的核心思想就是「**邏輯內聚**」。這意味著，你可以把所有跟某個特定功能相關的程式碼都寫在一起。例如，一個「滑鼠追蹤」的功能，它的 `x, y` 座標、更新座標的方法、以及監聽滑鼠移動的生命週期鉤子，都可以放在一起。

這也為我們未來學習「**Composable**」（自訂 Hook，Day 10 會深入探討）打下了基礎。你可以把這些內聚的邏輯抽離成一個可複用的函式，就像製作一個個「功能模組」，隨插即用！

## 小技巧與注意事項：

*   **`ref` 的 `.value`**：在 `<script setup>` 中，當你定義一個 `ref` 響應式變數時，在 `<script>` 區塊內訪問它的值，記得要加上 `.value`（例如 `count.value++`）。但在 `<template>` 模板中，Vue 會很聰明地幫你自動解包，所以你直接寫 `{{ count }}` 就行了，不用寫 `{{ count.value }}`。
*   **響應式基礎**：今天我們簡單提到了 `ref`。明天 Day 8，我們將會深入探討 `ref` 和 `reactive` 這兩個 Vue 響應式系統的基石，了解它們的核心差異和最佳使用時機。
*   **與 Options API 混用**：雖然 Vue 允許你在同一個組件中同時使用 `<script setup>` 和 Options API（例如，你可以在 `<script setup>` 之外再寫一個 `export default {}`），但通常不推薦這樣做。這會讓程式碼風格不一致，增加維護的複雜性。盡量保持風格統一，讓你的程式碼看起來更「舒服」。

## 本篇自我挑戰

-   **今日挑戰**：嘗試將一個你過去用 Options API 寫的簡單組件（例如一個計數器、一個顯示/隱藏文字的組件），改寫成使用 `<script setup>` 的形式。感受一下程式碼的變化。
-   **反思**：你覺得 `<script setup>` 如何解決了 Day 1 提到的 Options API「邏輯分散」的問題？它讓程式碼看起來更清晰、更易於理解和維護嗎？

## 總結

今天我們正式踏入了 Composition API 的大門，並認識了它的最佳拍檔——`<script setup>` 語法糖。它不僅讓 Vue 3 的程式碼變得更加簡潔、直觀，更重要的是，它鼓勵我們以「功能」為導向來組織程式碼，解決了 Options API 時代的「邏輯分散」問題，大大提升了開發效率和程式碼的可維護性。

本日關鍵字回顧

-   **Composition API**: Vue 3 的核心，以「功能」組織程式碼。
-   **`<script setup>`**: Composition API 的編譯時語法糖，簡化寫法。
-   **邏輯內聚**: 將相關邏輯集中在一起。
-   **語法糖**: 讓程式碼更簡潔易讀的語法特性。
-   **`ref`**: 用於定義響應式基本型別數據。
-   **自動解包**: `ref` 在模板中無需 `.value` 即可訪問。

明天，我們將深入探討 `ref` 和 `reactive` 這兩個 Vue 響應式系統的基石，了解它們的核心差異與最佳使用時機。準備好迎接更多 Vue 3 的魔法了嗎？
