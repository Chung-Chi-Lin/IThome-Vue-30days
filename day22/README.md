# 【Day 22】為你的程式碼買保險：Vitest 單元測試入門

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

你是否曾經歷過這樣的恐懼：

在一個夜深人靜的晚上，你修好了一個 Bug，心滿意足地提交了程式碼。結果隔天早上，團隊成員跑來跟你說：「你昨天的修改，把另外三個功能搞壞了！」

這種「修好 A，卻搞壞 B 和 C」的慘劇，在軟體開發中被稱為「**迴歸 (Regression)**」。它就像一個潛伏在暗處的幽靈，總在你意想不到的時候出現。

那麼，我們要如何才能安心地修改、重構、或添加新功能，而不用擔心會意外破壞現有的程式碼呢？答案就是：**自動化測試 (Automated Testing)**。

自動化測試就像是為你的程式碼買了一份「**保險**」。它是一個不知疲倦的機器人，在你每次修改程式碼後，都會自動把所有功能重新檢查一遍，確保一切都還正常運作。這份保險，帶給你的是重構的勇氣與上線的信心。

今天，我們將認識 Vue 生態系中的測試新星 **Vitest**，並為我們的專案撰寫第一支「**單元測試 (Unit Test)**」。

## 1. 什麼是單元測試 (Unit Test)？

在所有測試類型中，單元測試是金字塔的底座，數量最多，執行速度也最快。

**它的核心思想是：將程式碼拆分成最小的、可測試的「單元」，並獨立地驗證這個單元的功能是否正確。**

-   一個「單元」可以是一個函式、一個 Vue 元件、或是一個 Class。
-   「獨立地驗證」意味著我們在測試 A 時，不應該關心 B 或 C 的運作情況。

**打個比方**：在測試我們 Day 21 做的 `BaseButton` 元件時，我們只關心這個「按鈕」本身。例如：

-   給它 `disabled` prop，它真的不能被點擊嗎？
-   給它 `loading` prop，它真的會顯示讀取中的圖示嗎？

我們不關心這個按鈕被放在哪個頁面、點了之後會觸發什麼複雜的業務邏輯。我們只測試這個「單元」本身，確保它的行為符合預期。

## 2. 認識測試工具：Vitest + Vue Test Utils

-   **Vitest**：一個由 Vite 團隊打造的現代化測試框架 (Test Runner)。它的優點是：
    -   **快**：利用 Vite 的即時熱更新（HMR）引擎，測試速度極快。
    -   **設定簡單**：能直接沿用 `vite.config.js` 的設定，無痛整合。
    -   **API 友好**：與流行的 Jest 框架 API 兼容，學習曲線平緩。

-   **Vue Test Utils**：Vue 官方的元件測試工具庫。它提供了一系列輔助函式，讓我們可以「**掛載 (mount)**」一個元件，並對其進行互動和斷言。它就像是我們在測試環境中，用來操作 Vue 元件的那雙「手」。

## 3. 環境設定

首先，讓我們把這兩位新夥伴請進專案裡。

```bash
npm install -D vitest @vue/test-utils
```

接著，我們需要稍微調整一下 `vite.config.js`，告訴 Vite 我們要使用 Vitest。

```javascript
// vite.config.js
/// <reference types="vitest" />
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  test: {
    // 讓 Vitest 在 Node.js 環境中模擬瀏覽器 API
    environment: 'jsdom',
    // 讓 expect() 等全域 API 無需 import 就能使用
    globals: true,
  },
});
```
> `/// <reference types="vitest" />` 這行註解是給 TypeScript 看的，能讓設定檔獲得 Vitest 的型別提示。

最後，在 `package.json` 中加入測試指令：

```json
// package.json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview",
  "test": "vitest" // 新增這一行
},
```

## 4. 你的第一支測試：測試 Composable

測試最簡單的目標，是那些不涉及 UI 的純粹邏輯。我們在 Day 10 學到的 Composable 就是絕佳的例子。

假設我們有一個簡單的計數器 Composable：

```javascript
// src/composables/useCounter.js
import { ref } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  const increment = () => count.value++;
  const decrement = () => count.value--;

  return { count, increment, decrement };
}
```

現在，我們來為它寫測試。依照慣例，測試檔案會與源檔案放在一起，並以 `.spec.js` 或 `.test.js` 結尾。

```javascript
// src/composables/useCounter.spec.js

// 從 vitest 引入測試三劍客：describe, it, expect
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

// describe: 用來將一組相關的測試包裝在一起，形成一個「測試套件」
describe('useCounter', () => {

  // it: 代表一個獨立的「測試案例」，描述這個單元應該有的行為
  it('should initialize with the default value 0', () => {
    const { count } = useCounter();
    // expect: 斷言，也就是我們期望的結果
    // .toBe(): 是一個「匹配器 (Matcher)」，用來比較結果是否「全等於」期望值
    expect(count.value).toBe(0);
  });

  it('should initialize with a given value', () => {
    const { count } = useCounter(10);
    expect(count.value).toBe(10);
  });

  it('should increment the count after calling increment', () => {
    const { count, increment } = useCounter();
    increment();
    expect(count.value).toBe(1);
  });

  it('should decrement the count after calling decrement', () => {
    const { count, decrement } = useCounter(5);
    decrement();
    expect(count.value).toBe(4);
  });
});
```

寫完後，執行 `npm test`，你將會看到 Vitest 漂亮的輸出，告訴你所有測試都通過了！

## 5. 你的第一支元件測試

邏輯測試很單純，但元件測試稍微複雜一點，因為它涉及到渲染。這時 `@vue/test-utils` 就要登場了。

讓我們測試一個簡單的 `Greeting.vue` 元件：

```vue
// src/components/Greeting.vue
<template>
  <p>Hello, {{ name }}!</p>
</template>

<script setup>
  defineProps({ name: { type: String, default: 'World' } });
</script>
```

測試程式碼如下：

```javascript
// src/components/Greeting.spec.js
import { describe, it, expect } from 'vitest';
// 從 @vue/test-utils 引入 mount 函式
import { mount } from '@vue/test-utils';
import Greeting from './Greeting.vue';

describe('Greeting.vue', () => {
  it('renders the default greeting if no prop is passed', () => {
    // mount: 將元件掛載到一個虛擬的 DOM 中，並回傳一個「包裝器 (wrapper)」
    const wrapper = mount(Greeting);

    // wrapper.text(): 取得元件渲染出來的純文字內容
    expect(wrapper.text()).toContain('Hello, World!');
  });

  it('renders the name passed as a prop', () => {
    const name = 'Vue';
    const wrapper = mount(Greeting, {
      // 在 mount 的第二個參數中，我們可以傳入 props, slots 等設定
      props: {
        name: name,
      },
    });

    expect(wrapper.text()).toContain(`Hello, ${name}!`);
  });
});
```

`mount` 函式是元件測試的核心，它就像一個小小的瀏覽器，幫我們把元件渲染出來，而回傳的 `wrapper` 物件，就是我們用來檢查渲染結果的放大鏡。

## 本篇自我挑戰

1.  **安裝工具**：跟著文章，在你的專案中安裝 `vitest` 與 `@vue/test-utils`，並完成相關設定。
2.  **測試純函式**：如果你的專案中有一些工具函式 (utils)，例如格式化日期、計算金額等，試著為其中一個寫下你的第一個單元測試。
3.  **測試簡單元件**：挑一個最簡單的、只負責顯示資料的元件，試著用 `mount` 掛載它，並斷言它渲染的文字內容是否正確。

## 總結

恭喜你！你已經踏出了自動化測試的第一步，為你的程式碼品質加上了第一道防線。

今天我們學到了：

-   **單元測試** 是驗證最小程式碼單元行為的測試，能有效防止「迴歸」。
-   **Vitest** 是 Vue 專案的絕佳測試夥伴，快速且易於設定。
-   **Vue Test Utils** 提供了 `mount` 函式，讓我們能在測試環境中渲染 Vue 元件。
-   測試的基本結構由 **`describe`**, **`it`**, 和 **`expect`** 組成。

一開始寫測試可能會覺得有點繁瑣，但這份「保險」的價值，將在你未來的每一次重構和功能迭代中，給你帶來無比的信心與安心。

明天，我們將深入探討如何測試 Vue 元件更複雜的互動：**Props、事件 (Emits) 與插槽 (Slots)**。

## 本日關鍵字回顧

-   `單元測試 (Unit Testing)`
-   `迴歸 (Regression)`
-   `Vitest`
-   `@vue/test-utils`
-   `describe / it / expect`
-   `斷言 (Assertion)`
-   `匹配器 (Matcher)`
-   `mount`
-   `包裝器 (Wrapper)`
-   `環境 (Environment: jsdom)`
