# 【Day 2】Vue 的核心互動：深入 `v-bind` 與 `v-on` 的魔法

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天我們聊了 Vue 3 的核心演進，知道了它背後那套讓我們開發更順暢的「組合式 API」與效能優化的黑魔法。但光說不練，總覺得有點空虛。今天，我們就來動手玩點真的，深入 Vue 世界最基本、也最重要的互動核心：**資料綁定 (`v-bind`)** 與 **事件處理 (`v-on`)**。

這兩個指令，可以說是 Vue 的任督二脈。一旦打通，你的網頁就能從「靜態的展示品」變成「能與使用者互動的應用程式」。準備好讓你的網頁活起來了嗎？

## 資料綁定：讓畫面跟著你的「資料」走

想像一下，你的網頁是一具提線木偶，而你的資料（在 Vue 的 `<script setup>` 裡定義的變數）就是控制木偶的線。**資料綁定** 就是把這些線（資料）跟木偶的特定部位（HTML 元素）綁在一起的過程。

### `v-bind`：單向的資料流

`v-bind` 就像一條單行道，資料只能從 `<script>` 流向 `template` (HTML)。當你的資料改變時，畫面會自動更新，但反過來不行。

最常見的用途就是綁定 HTML 元素的屬性 (Attribute)，例如圖片的 `src`、連結的 `href`，或是元素的 `class` 和 `style`。

**語法**：`v-bind:屬性名="資料變數"`，不過在實務上，大家更愛用它的**縮寫**：一個簡單的冒號 `:屬性名`。

**舉個例子：** 讓我們來綁定一張圖片的 URL 和一個 `class`。

```html
<script setup>
import { ref, computed } from 'vue';

const vueLogoUrl = ref('https://vuejs.org/images/logo.png');
const isImportant = ref(true);

// 我們也可以綁定一個 computed property
const textClasses = computed(() => ({
  'important': isImportant.value,
  'normal-text': !isImportant.value,
}));
</script>

<template>
  <!-- 基本綁定 -->
  <img :src="vueLogoUrl" alt="Vue Logo">

  <!-- 綁定 class -->
  <p :class="textClasses">這段文字會根據 isImportant 的值改變樣式。</p>

  <!-- 切換按鈕 -->
  <button @click="isImportant = !isImportant">
    切換重要性
  </button>
</template>

<style>
.important {
  color: red;
  font-weight: bold;
}
.normal-text {
  color: black;
}
</style>
```
> 在這個例子中，圖片的 `src` 被 `vueLogoUrl` 這個 ref 變數控制。`p` 標籤的 `class` 則由 `textClasses` 這個計算屬性動態決定。當你點擊按鈕，`isImportant` 的值改變，`textClasses` 會重新計算，`p` 標籤的樣式也跟著更新。這就是單向綁定的威力！

#### 深入一點：Vue 編譯器做了什麼？
你可能會好奇，`<img :src="vueLogoUrl">` 這段看起來像 HTML 的程式碼，瀏覽器又看不懂，Vue 到底是怎麼讓它動起來的？

這就是 Vue **編譯器 (Compiler)** 的功勞。在你執行 `npm run dev` 或 `npm run build` 時，Vue 會把你的 `.vue` 檔案模板 (template) 進行編譯，轉換成瀏覽器看得懂的 JavaScript **渲染函式 (Render Function)**。

所以，`<img :src="vueLogoUrl">` 會被轉換成類似這樣的程式碼：
```javascript
import { h } from 'vue'

// ... 渲染函式內部 ...
return h('img', { src: vueLogoUrl.value, alt: 'Vue Logo' })
```
`h()` 函式會建立一個 **虛擬 DOM (Virtual DOM)** 節點。當 `vueLogoUrl` 的值改變時，Vue 的響應式系統會偵測到變化，並重新執行渲染函式，產生新的虛擬 DOM。接著，Vue 會比較新舊虛擬 DOM 的差異，並只更新真實 DOM 中真正改變的部分（也就是 `src` 屬性）。

這個過程非常高效，也是 Vue 效能優異的核心祕密之一。

#### 如何優雅地加入判斷？ `computed` vs `template` 內聯判斷

當綁定的值需要一些邏輯判斷時，你有兩種主要選擇：

1.  **內聯表達式 (Inline Expression)**：直接在模板裡寫 JavaScript 表達式，最常見的是三元運算子。
    ```html
    <p :class="isImportant ? 'important' : 'normal-text'">
      這段文字很重要。
    </p>
    ```
2.  **計算屬性 (Computed Property)**：將邏輯封裝在 `<script>` 的 `computed` 函式中。
    ```html
    <!-- :class="textClasses" -->
    ```
    (如我們上方的 `textClasses` 範例)

**應該選哪個？ -> 強烈推薦 `computed`！**

雖然內聯表達式對於極度簡單的邏輯來說很方便，但只要你的邏輯稍微複雜一點，`computed` 就會是更好的選擇。理由如下：

-   **可讀性 (Readability)**：將邏輯從模板中分離，可以讓模板保持乾淨、語意化。模板應該專注於「呈現什麼」，而不是「如何計算」。
-   **可複用性 (Reusability)**：如果多個地方都需要用到相同的邏輯，`computed` 可以讓你輕鬆複用，而不用複製貼上一堆表達式。
-   **效能 (Performance)**：`computed` 是基於其響應式依賴進行快取的。只有當依賴的 ref (例如 `isImportant`) 發生變化時，它才會重新計算。而內聯表達式或 `method` 則是在每次元件重新渲染時都會重新執行，效能較差。
-   **可測試性 (Testability)**：在 `<script>` 中的邏輯遠比在 `<template>` 中的邏輯更容易進行單元測試。

**結論**：當你的綁定需要任何形式的邏輯運算時，優先考慮使用 `computed`。這是一個能顯著提升你程式碼品質的好習慣。

### `v-model`：雙向的溝通橋樑

如果 `v-bind` 是單行道，那 `v-model` 就是一條雙向高速公路。它通常用在表單元素上，像是 `<input>`, `<textarea>`, `<select>`。

它同時做了兩件事：
1.  用 `v-bind` 把資料綁定到輸入框的 `value`。
2.  用 `v-on` 監聽輸入框的 `input` 事件，並在事件觸發時更新資料。

**簡單來說：資料變，輸入框跟著變；輸入框變，資料也跟著變。**

**舉個例子：** 一個簡單的即時回饋輸入框。

```html
<script setup>
import { ref } from 'vue';

const message = ref('');
</script>

<template>
  <input v-model="message" placeholder="隨便打點什麼...">
  <p>你輸入的是：{{ message }}</p>
</template>
```
> 試著在輸入框裡打字，你會發現底下的文字會即時更新。這就是 `v-model` 的魔力，它讓處理表單資料變得無比輕鬆。

## 事件處理：傾聽使用者的聲音

如果資料綁定是讓畫面動起來，那事件處理就是讓使用者「參與」進來。當使用者點擊按鈕、滑動頁面、或是在鍵盤上敲敲打打時，我們需要一個機制來「聽到」這些動作，並做出回應。這個機制就是 `v-on`。

### `v-on`：你的事件監聽器

`v-on` 用來監聽 DOM 事件，並在事件觸發時執行指定的 JavaScript 程式碼。

**語法**：`v-on:事件名="函式"`，同樣地，它也有個更受歡迎的**縮寫**：`@事件名`。

**舉個例子：** 一個簡單的計數器。

```html
<script setup>
import { ref } from 'vue';

const count = ref(0);

function increment() {
  count.value++;
}

function decrement() {
  count.value--;
}
</script>

<template>
  <p>目前的計數：{{ count }}</p>

  <!-- 綁定一個已定義的函式 -->
  <button @click="increment">增加</button>

  <!-- 也可以直接寫簡單的表達式 -->
  <button @click="decrement">減少</button>
</template>
```
> 這裡，我們用 `@click` 分別在兩個按鈕上監聽了點擊事件。點擊時，它們會分別呼叫 `increment` 函式或執行 `count.value--` 這個表達式。

### 事件修飾符：讓事件處理更優雅

有時候我們需要對事件做一些額外的處理，例如「阻止預設行為」（像 a 標籤的跳轉）或「停止事件冒泡」。在傳統 JS 中，我們得手動呼叫 `event.preventDefault()` 或 `event.stopPropagation()`。

Vue 提供了一系列好用的**事件修飾符**，讓我們可以優雅地處理這些情況。

- `.prevent`：阻止預設行為。
- `.stop`：停止事件冒泡。
- `.once`：事件只觸發一次。
- `.self`：只有當事件是從這個元素本身觸發時才觸發。
- `.capture`：使用事件捕獲模式。

**舉例：** 阻止表單提交後重新整理頁面。

```html
<form @submit.prevent="onSubmit">
  <button type="submit">提交</button>
</form>
```
> 加上 `.prevent` 修飾符後，點擊提交按鈕時，表單將不會觸發預設的頁面重整行為，而是只會執行我們的 `onSubmit` 函式。是不是乾淨俐落多了？

## 本篇自我挑戰

- **今日挑戰**：結合今天所學，做一個「文字反轉器」。
  1.  建立一個 `<input>` 輸入框，並使用 `v-model` 綁定一個 ref 變數 `text`。
  2.  在輸入框下方，顯示 `text` 反轉後的結果。
  3.  新增一個按鈕，點擊後可以清空輸入框的內容。
- **思考**：`v-model` 實際上是 `v-bind` 和 `v-on` 的語法糖，你能想像它背後大概是如何運作的嗎？

## 總結

今天我們掌握了 Vue 中最核心的兩個互動工具：`v-bind` 和 `v-on`。`v-bind`（與它的雙向版本 `v-model`）負責讓我們的資料與畫面同步，而 `v-on` 則負責傾聽使用者的操作並做出反應。

這兩者結合起來，就構成了 Vue 響應式系統的基礎。無論多複雜的應用，其底層的互動邏輯都離不開這兩位好朋友。

本日關鍵字回顧

- `v-bind`: 單向資料綁定，縮寫是 `:`。
- `v-model`: 雙向資料綁定，主要用於表單。
- `v-on`: 事件監聽，縮寫是 `@`。
- `Event Modifiers`: 事件修飾符，如 `.prevent` 和 `.stop`，讓事件處理更簡潔。

明天，我們將繼續探索 Vue 的模板語法，學習如何根據條件顯示或隱藏元素 (`v-if`)，以及如何渲染一個列表 (`v-for`)。敬請期待！
