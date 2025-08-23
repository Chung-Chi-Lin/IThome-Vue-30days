# 【Day 5】組件的彈性設計：掌握 Slot 與動態組件

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天我們學會了如何像組合樂高一樣，用 `Props` 和 `Emit` 讓父子組件溝通，搭建出結構清晰的應用。`Props` 讓我們可以傳遞「資料」，像是文字、數字或陣列，這非常強大。

但如果我們想傳遞的不是單純的資料，而是**一整塊 HTML 結構**呢？

想像一下，你想做一個通用的「卡片」組件，但希望卡片的「內容」可以每次都不同。有時放圖片，有時放表單，有時甚至放其他組件。如果只用 props 傳遞 HTML 字串，那也太不優雅了！

今天，我們就要來學習兩個讓組件彈性倍增的進階技巧：**插槽 (Slot)** 和 **動態組件 (Dynamic Components)**。它們將徹底解放你對組件複用的想像力！

## 插槽 (Slot)：為你的組件開一個「內容投入口」

`Slot` 是 Vue 組件中一個非常核心的概念，它允許父組件將模板內容「注入」到子組件的指定位置。

你可以把帶有插槽的子組件想像成一個「**便當盒 (Bento Box)**」。便當盒本身（子組件）提供了外觀和格子，但裡面要裝什麼菜（內容），則由做便當的人（父組件）決定。

### 預設插槽 (Default Slot)

最簡單的插槽，就像是便當盒裡那個**最大、最主要的菜色區**。在子組件中，你只需要放一個 `<slot>` 標籤，它就會變成一個內容的「入口」。

**`Card.vue` (子組件 - 便當盒)**
```html
<template>
  <div class="card">
    <div class="card-content">
      <!-- 主菜區：父組件的內容將會被放到這裡 -->
      <slot></slot>
    </div>
  </div>
</template>

<style>
.card { 
  border: 1px solid #ccc; 
  border-radius: 8px; 
  padding: 16px; 
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
</style>
```

**`App.vue` (父組件 - 做便當的人)**
```html
<script setup>
import Card from './Card.vue';
</script>

<template>
  <h1>今天的便當</h1>

  <Card>
    <!-- 在主菜區放入圖片和文字 -->
    <h2>這是一張圖片卡片</h2>
    <img src="https://vuejs.org/images/logo.png" alt="Vue Logo">
    <p>Vue 的 Logo 真是好看！</p>
  </Card>

  <Card>
    <!-- 這次主菜區改放登入表單 -->
    <h2>這是一個登入表單</h2>
    <input type="text" placeholder="使用者名稱">
    <input type="password" placeholder="密碼">
    <button>登入</button>
  </Card>
</template>
```
> 看到了嗎？同一個 `Card` 便當盒，因為父層放入了不同的主菜，展現了完全不同的樣貌。這就是 Slot 的威力：**將結構與內容分離**。

### 具名插槽 (Named Slots)

有時候，一個便當盒不只有一個大格子，還會有放白飯、小菜、水果的**特定小格子**。這時，我們就可以使用「具名插槽」。

**`PageLayout.vue` (子組件 - 多格便當盒)**
```html
<template>
  <div class="layout">
    <header>
      <!-- 'header' 小格子 -->
      <slot name="header"></slot>
    </header>
    <main>
      <!-- 預設主菜區 -->
      <slot></slot> 
    </main>
    <footer>
      <!-- 'footer' 小格子 -->
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

**`App.vue` (父組件 - 做便當的人)**
```html
<script setup>
import PageLayout from './PageLayout.vue';
</script>

<template>
  <PageLayout>
    <!-- 使用 v-slot 指令 (縮寫 #) 來指定菜色要放進哪個格子 -->
    <template #header>
      <h1>我的網站標題</h1>
    </template>

    <!-- 沒有指定格子的內容，會自動放入預設的主菜區 -->
    <p>這是我的主要內容...</p>

    <template #footer>
      <p>版權所有 © 2023</p>
    </template>
  </PageLayout>
</template>
```
> 透過 `v-slot:名稱` (或縮寫 `#名稱`)，我們就能精準地將不同的菜色（內容）放置到便當盒（子組件）對應的格子（具名插槽）中。

## 動態組件：讓組件根據狀態切換

學會了 Slot，我們的組件已經很靈活了。但還有另一種場景：根據使用者的操作，在同一個位置切換顯示完全不同的組件。例如，在一個分頁 (Tabs) 介面中點擊不同的頁籤。

你當然可以用 `v-if`, `v-else-if`... 來做，但當選項一多，模板就會變得很臃腫。

Vue 提供了一個更優雅的方案：特殊的 `<component>` 元素。

**語法**：`<component :is="currentComponent"></component>`

這個元素的 `:is` 屬性可以綁定一個變數，Vue 會根據這個變數的值來決定要渲染哪一個組件。

**舉個例子**：一個簡單的分頁切換。

**`TabHome.vue`**
`<template><div>這裡是首頁內容</div></template>`

**`TabPosts.vue`**
`<template><div>這裡是文章列表</div></template>`

**`TabArchive.vue`**
`<template><div>這裡是封存資料</div></template>`

**`App.vue` (父組件)**
```html
<script setup>
import { ref, shallowRef } from 'vue';
import TabHome from './TabHome.vue';
import TabPosts from './TabPosts.vue';
import TabArchive from './TabArchive.vue';

const tabs = {
  TabHome,
  TabPosts,
  TabArchive
};

// 使用 shallowRef 來存放組件，避免不必要的效能開銷
const currentTab = shallowRef(TabHome);
</script>

<template>
  <div class="tabs">
    <button v-for="(_, tabName) in tabs" @click="currentTab = tabs[tabName]">
      {{ tabName }}
    </button>
  </div>

  <!-- 動態渲染選擇的組件 -->
  <component :is="currentTab" />
</template>
```
> 在這個例子中，我們只需要改變 `currentTab` 這個變數的值，Vue 就會自動在 `<component>` 的位置切換顯示對應的組件，程式碼非常乾淨！

**小提示**：搭配 `<keep-alive>` 標籤可以將切換掉的組件緩存在記憶體中，避免重複渲染，優化效能。
`<keep-alive><component :is="currentTab" /></keep-alive>`

## 本篇自我挑戰

- **思考一：設計一個通用的「彈出視窗」組件**
  如果你要設計一個 `Modal.vue` 組件，你會如何使用「具名插槽」來讓它的「標題 (header)」、「內容 (body)」和「操作按鈕 (footer)」都可以被父層完全客製化？

- **思考二：`v-if` vs `<component :is>`**
  在什麼情況下，你會選擇使用 `<component :is>` 而不是一長串的 `v-if`/`v-else-if` 來切換組件？使用動態組件有哪些好處？

## 總結

今天我們學會了打造高彈性組件的兩大殺手鐧：
-   **插槽 (Slot)**：讓我們可以從父層向子層「注入」HTML 內容，實現了**內容與結構的分離**，讓組件的用途更廣泛。
-   **動態組件 (`<component :is>`)**：提供了一種優雅的方式，讓我們可以根據狀態動態地切換要渲染的組件，讓介面互動更流暢。

掌握了這兩項技巧，你已經具備了設計大型、可複用、高彈性 UI 元件庫的能力。你的組件不再是寫死的積木，而是可以千變萬化的「變形金剛」！

本日關鍵字回顧

- `Slot`: 插槽，組件的內容「入口」，就像便當盒的格子。
- `Default Slot`: 預設插槽，接收沒有具名指定的所有內容。
- `Named Slots`: 具名插槽，允許有多個獨立的內容入口。
- `v-slot` (或 `#`): 用於將內容放入指定插槽的指令。
- `Dynamic Components`: 動態組件，可以根據資料變化而切換的組件。
- `<component :is="...">`: 渲染動態組件的內建元素。

我們已經知道如何建立組件、讓它們溝通、甚至讓它們變形。但組件從被建立到被銷毀，它到底經歷了什麼？明天，我們將一起探索組件的「生命週期」，了解在不同階段我們可以做些什麼事。敬請期待！
