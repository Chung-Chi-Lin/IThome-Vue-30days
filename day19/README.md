# 【Day 19】為你的樣式穿上盔甲：SCSS 整合與 CSS Modules

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的造型師！過去幾天，我們一直在 `<script>` 的世界裡鑽研 TypeScript 的內功心法，讓我們的邏輯變得堅不可摧。今天，我們要換個場景，把目光移到 `<style>` 標籤，來解決另一個讓開發者頭痛不已的問題：**CSS 樣式衝突**。

想像一下，你的應用程式是個盛大的化妝舞會。你精心為 `Card` 組件設計了一套華麗的 `.title` 樣式。結果，另一個 `Header` 組件也定義了一個 `.title` 樣式。最後，在舞會上，你的 `Card` 標題莫名其妙地戴上了 `Header` 的帽子！這就是 **CSS 全域污染** 的噩夢。

今天，我們要學習兩大神器，來徹底解決這個問題：
1.  **SCSS**：為你的 CSS 語法升級，提供變數、巢狀等強大功能，就像是給你的造型師一套「專業化妝工具」。
2.  **CSS Modules**：為每個組件的樣式穿上一件「**獨立的隱形盔甲**」，讓它的樣式只對自己生效，永不外洩！

## 第一步：升級你的工具 (整合 SCSS)

SCSS (Sassy CSS) 是 CSS 的超集，它提供了許多 CSS 原生沒有的便利功能，讓寫樣式變得更有趣、更有效率。在 Vue 中整合它，只需要一步。

**1. 安裝 `sass`**

```bash
npm install -D sass
```

**2. 開始使用**

就這樣！現在你可以在你的 `.vue` 檔案中，將 `<style>` 標籤改為 `<style lang="scss">`，然後就可以開始使用 SCSS 的魔法了。

```html
<style lang="scss">
// 1. 變數：把常用的顏色、字體大小存起來
$primary-color: #42b983;
$font-size-large: 18px;

.card {
  border: 1px solid $primary-color;

  // 2. 巢狀：樣式結構和 HTML 結構一樣，清晰明瞭
  .title {
    font-size: $font-size-large;
    color: $primary-color;

    &:hover {
      color: darken($primary-color, 10%); // 內建函式，加深顏色
    }
  }

  .content {
    padding: 10px;
  }
}
</style>
```

> SCSS 就像是 CSS 的「語法糖」，它讓你的樣式碼更易讀、更易維護，但它本身並不能解決「全域污染」的問題。

## 終極解決方案：CSS Modules 的隱形盔甲

即使有了 SCSS，如果你在 `ComponentA` 和 `ComponentB` 中都寫了 `.card` 樣式，它們依然會在全域打架。為了解決這個問題，Vue 提供了一個大殺器：**CSS Modules**。

你只需要做一個小小的改動：在 `<style>` 標籤上加上 `module` 屬性。

```html
<template>
  <!-- 注意這裡的 class 綁定方式 -->
  <div :class="$style.card">
    <h3 :class="$style.title">這是一個卡片標題</h3>
    <p :class="$style.content">這是一些內容。</p>
  </div>
</template>

<script setup>
// CSS Modules 會在組件中注入一個名為 $style 的物件
// 你可以把它 log 出來看看它的結構
// import { useCssModule } from 'vue';
// const style = useCssModule();
// console.log(style);
</script>

<!-- 加上 "module" 屬性，啟動隱形盔甲！ -->
<style module lang="scss">
.card {
  border: 1px solid #ccc;
  padding: 16px;
  text-align: center;
}

.title {
  color: #42b983;
}

.content {
  font-size: 14px;
}
</style>
```

### 盔甲是如何運作的？

當你使用 `<style module>`，Vue 和建置工具在背後會做一件神奇的事：它會把你寫的 class 名稱（如 `.card`）自動轉換成一個**全域唯一的 hash 名稱**。

-   你寫的 `.card` 可能會變成 `.MyComponent_card_Abc123`。
-   你寫的 `.title` 可能會變成 `.MyComponent_title_Xyz789`。

同時，它會把這個「對照表」注入到你的組件裡，成為一個叫做 `$style` 的物件。所以你可以透過 `$style.card` 來存取到那個獨一無二的 class 名稱。

因為每個組件的 class 名稱都帶有自己的「指紋」，所以它們永遠不可能和來自其他組件的樣式發生衝突！這就是**作用域化 CSS (Scoped CSS)** 的完美實現。

### `scoped` vs `module`

你可能會問：Vue 不是還有一個 `<style scoped>` 嗎？它和 `module` 有什麼不同？

-   **`<style scoped>`**：是 Vue 的另一種樣式隔離方案。它透過為 HTML 元素添加一個獨特的屬性（如 `data-v-f3f3eg9`）並修改 CSS 選擇器（如 `.card` -> `.card[data-v-f3f3eg9]`）來實現作用域。它更「自動」，你不需要修改 class 的寫法。
-   **`<style module>`**：更加「明確」。你必須手動透過 `$style` 物件來綁定 class，這讓你的模板能清楚地表明這個 class 是來自於 CSS Module。它提供了更強的控制力，並且與 JS 的互動更自然。

兩者都是解決方案，沒有絕對的好壞。但 CSS Modules 因為其明確性和與 JS 的良好整合，在大型專案和組件庫中越來越受歡迎。

## 總結

今天我們為組件的樣式進行了兩次大升級。

1.  **整合 SCSS**：透過安裝 `sass` 並使用 `<style lang="scss">`，我們解鎖了變數、巢狀、Mixin 等強大功能，讓 CSS 寫起來更像一門程式語言。
2.  **啟用 CSS Modules**：透過在 `<style>` 標籤上添加 `module` 屬性，我們為每個組件的樣式穿上了「隱形盔甲」，並透過 `$style` 物件來存取它們，從而 100% 避免了全域樣式衝突的問題。

從此以後，你可以放心地為每個組件設計樣式，再也不用擔心它會影響到別人，或是被別人影響了！

## 本篇自我挑戰

-   **今日挑戰**：找一個你之前寫的組件，將它的 `<style>` 改為 `<style module lang="scss">`。嘗試使用 SCSS 的變數和巢狀語法來重構你的樣式。最後，記得在 `<template>` 中使用 `:class="$style.yourClassName"` 來綁定樣式。
-   **反思**：你更喜歡 `<style scoped>` 的自動化，還是 `<style module>` 的明確性？思考一下在什麼樣的場景下，你可能會選擇其中一種而不是另一種。

本日關鍵字回顧

-   **SCSS**: CSS 的超集，提供了變數、巢狀等程式化功能。
-   **全域污染 (Global Pollution)**: CSS 樣式因選擇器名稱相同而互相覆蓋的問題。
-   **CSS Modules**: 一種 CSS 檔案的編譯技術，能產生作用域化的、獨一無二的 class 名稱。
-   **`<style module>`**: 在 Vue 中啟用 CSS Modules 的方式。
-   **`$style`**: Vue 注入到組件中的物件，包含了原始 class 名稱到唯一名稱的對應。
-   **作用域化 CSS (Scoped CSS)**: 讓 CSS 樣式只在特定範圍內生效的技術。

明天，我們將學習如何使用 SCSS 建立可維護的全域樣式系統，讓你的專案在擁有獨立盔甲的同時，也能共享一套統一的設計語言！
