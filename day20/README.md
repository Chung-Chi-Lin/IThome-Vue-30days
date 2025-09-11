# 【Day 20】SCSS 全域樣式管理：變數與 Mixin 的組合技

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

嘿，各位樣式魔法師！

昨天我們成功地將 SCSS 整合進了 Vue 專案，讓我們的 CSS 也能享受到模組化的清爽。但如果只是這樣，SCSS 的威力還遠遠沒有發揮出來。

你有沒有遇過這種情況：專案越做越大，設計師突然說：「我們的主題色從 A 藍改成 B 藍吧！」

於是你開始了 `Ctrl + F` 的漫漫長路，在幾十個檔案裡大海撈針，尋找那個該死的色碼 `#3498db`。改完之後，你還心驚膽顫，深怕漏掉任何一個角落，導致網站上同時出現兩種藍色，像一件縫錯布料的衣服。

這種手動維護的災難，正是今天要解決的問題。我們將學習 SCSS 的兩大王牌功能：**變數 (Variables)** 與 **Mixin**。它們就像是你的樣式軍火庫，讓你從一個疲於奔命的救火隊員，晉升為運籌帷幄的樣式架構師。

準備好告別混亂，打造一個可預測、可維護的全域樣式系統了嗎？

## 1. 變數 (Variables)：給你的樣式值一個家

SCSS 變數非常直觀，它就像是給你的樣式值取一個「**綽號**」。與其記住一長串無意義的色碼 `hsl(210, 14%, 96%)`，不如記住一個有意義的名字，例如 `$background-color`。

這不僅是為了好記，更重要的是它建立了「**單一事實來源 (Single Source of Truth)**」。

**什麼是單一事實來源？**

想像一下，你的專案中所有關於「主色調」的定義，都來自同一個地方。當需要修改時，你只需要改動這一個源頭，所有引用它的地方都會自動更新。這就是變數的威力。

讓我們來建立一個專門存放變數的檔案吧！在 SCSS 中，以底線 `_` 開頭的檔案被稱為 **Partials**，它們告訴編譯器：「嘿，這只是個零件，別單獨編譯我。」

**實戰演練：建立 `_variables.scss`**

```scss
// src/styles/_variables.scss

// Colors - 設計師給的調色盤
$primary-color: #42b983; // Vue 綠
$secondary-color: #35495e; // 深灰藍
$text-color: #2c3e50;
$border-color: #eaecef;
$background-color: #f8f9fa;

// Font Sizes
$font-size-base: 16px;
$font-size-large: 1.25rem;
$font-size-small: 0.875rem;

// Spacing - 統一的間距單位
$spacing-unit: 8px;

// Z-index - 管理圖層堆疊
$z-index-modal: 1000;
$z-index-dropdown: 500;
```

> **實戰技巧**：將這些變數想像成「**設計令牌 (Design Tokens)**」。它們是設計系統的最小單位，是設計與開發之間的共同語言。

## 2. Mixin：你的 CSS 函式產生器

如果說變數是樣式的「名詞」，那 Mixin 就是樣式的「**動詞**」。它允許你將一整組 CSS 規則封裝起來，變成一個可以重複使用的「**樣式配方**」。

這完美體現了程式設計中的 **DRY (Don't Repeat Yourself)** 原則。

最常見的應用場景就是處理那些惱人但又常用的樣式，例如 Flexbox 置中、清除浮動、或是響應式設計的斷點。

**實戰演練：建立 `_mixins.scss`**

```scss
// src/styles/_mixins.scss

// Flexbox 置中
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

// 產生一個有基本樣式的按鈕
@mixin basic-button {
  padding: 8px 16px;
  border: 1px solid $border-color;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;

  &:hover {
    opacity: 0.8;
  }
}

// 響應式設計斷點 (搭配 @content)
@mixin for-tablet-up {
  // 當螢幕寬度大於 768px 時，套用傳進來的樣式
  @media (min-width: 768px) {
    @content;
  }
}
```

**如何使用 Mixin？**

我們使用 `@include` 指令來呼叫這個「配方」。

```vue
<style lang="scss" scoped>
@import '@/styles/_variables.scss';
@import '@/styles/_mixins.scss';

.container {
  @include flex-center; // 一行搞定 Flex 置中
  height: 100vh;
}

.submit-button {
  @include basic-button; // 套用基礎按鈕樣式
  background-color: $primary-color; // 再加上客製化顏色
  color: white;
}

.sidebar {
  width: 100%; // 手機上是滿版

  @include for-tablet-up { // 在平板以上
    width: 300px; // 寬度變為 300px
  }
}
</style>
```
> **關鍵字 `@content`**：它是一個強大的佔位符，允許你在 `@include` Mixin 時，傳入一整塊自訂的 CSS 樣式，極大地增加了 Mixin 的彈性。

## 3. 全域注入：讓變數與 Mixin 無所不在

雖然在每個組件中手動 `@import` 檔案是可行的，但這很繁瑣。我們可以利用 Vite 的設定，將這些 SCSS 工具自動注入到每個 Vue 組件中。

修改你的 `vite.config.js`：

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path';

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
  // CSS 預處理器設定
  css: {
    preprocessorOptions: {
      scss: {
        // 自動將這些檔案注入到每個 SCSS 檔案的頂部
        additionalData: `
          @import "@/styles/_variables.scss";
          @import "@/styles/_mixins.scss";
        `
      }
    }
  }
});
```
設定完成後，你就可以在任何組件的 `<style lang="scss">` 區塊中，直接使用 `$primary-color` 或 `@include flex-center`，無需再手動 `@import`！

## 本篇自我挑戰

1.  **挑戰一**：檢查你過去的專案，找出至少 3 個重複使用的色碼或 `font-size`，將它們提取到 `_variables.scss` 中。
2.  **挑戰二**：你是否常用到某個 CSS 組合（例如設定圖片為圓形、清除 input 預設樣式）？試著將它封裝成一個 `@mixin`。

## 總結

今天，我們學會了使用 SCSS 的變數與 Mixin 來打造一個強健的樣式系統。

-   **變數 (`$`)**：為你的設計令牌 (Design Tokens) 提供一個**單一事實來源**，讓修改樣式變得輕鬆愉快。
-   **Mixin (`@mixin`)**：將重複的樣式規則封裝成可複用的配方，貫徹 **DRY 原則**。
-   **全域注入**：透過 Vite 設定，讓你的樣式工具庫隨手可得，提升開發效率。

掌握了這兩項武器，你就不再是那個被 CSS 追著跑的開發者，而是能優雅地駕馭樣式、建立可擴展設計系統的架構師。明天，我們將探討如何基於這個系統，建立團隊可共享的 UI 元件庫。

## 本日關鍵字回顧

-   `SCSS Variables ($)`
-   `SCSS Mixin (@mixin)`
-   `@include`
-   `@content`
-   `Partials (_filename.scss)`
-   `單一事實來源 (Single Source of Truth)`
-   `設計令牌 (Design Tokens)`
-   `DRY (Don't Repeat Yourself)`
-   `Vite 全域注入 (preprocessorOptions)`
