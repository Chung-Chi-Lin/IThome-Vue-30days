# 【Day 4】Vue 的組件化：用 Props 和 Emit 打造你的樂高積木

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

大家好！在前三天，我們學會了資料綁定、事件處理，甚至還能用 `v-if` 和 `v-for` 來動態控制畫面結構。現在，我們的頁面已經具備了基礎的互動能力。

但隨著功能越來越多，你可能會發現所有的程式碼都擠在同一個檔案裡，開始變得混亂、難以管理。這就像你試圖用一整塊木頭雕刻一輛汽車，而不是用方向盤、輪胎、座椅等零件組裝起來。

今天，我們要學習 Vue 最核心、最迷人的概念之一：**組件化 (Componentization)**。我們將學習如何將介面拆分成一個個獨立、可複用的「積木」，並透過 `Props` 和 `Emit` 讓這些積木互相溝通。

準備好從「工匠」思維轉變為「工程師」思維了嗎？

## 組件的溝通：設定你的樂高積木

在 Vue 的世界裡，一個**組件 (Component)** 就是一個獨立、可複用的 Vue 實例。你可以把它想像成一塊功能獨特的**樂高積木**。例如，一個是「按鈕積木」，一個是「頭像積木」。

當我們搭建應用（父組件）時，就是將這些不同功能的積木組合起來。

為什麼要用組件？
-   **可複用性 (Reusability)**：同一個按鈕積木，你只需要設計一次，就可以在網站的任何地方使用。
-   **可維護性 (Maintainability)**：當你的應用程式由數十個小積木構成時，修改或除錯就變得非常簡單。你只需要專注在出問題的那個「積木」上，而不用大海撈針。
-   **程式碼組織**：將複雜的介面拆分成小積木，讓你的專案結構更清晰、更有條理。

### Props：為你的積木進行「配置」

如果組件是獨立的積木，那它們要如何溝通？最基本的方式就是 **Props (屬性)**。

Props 允許父組件將資料傳遞給子組件。你可以把它想像成在「使用」一塊積木時，為它進行**初始配置**。

**舉個例子**：我們有一個通用的「評論積木」(`Comment.vue`)。當我們在父層 (`App.vue`) 使用它時，需要配置它要顯示的作者和內容。

**`Comment.vue` (評論積木)**
```html
<script setup>
// 這塊積木宣告：我需要接收 author 和 content 這兩個配置
const props = defineProps({
  author: String,
  content: String
});
</script>

<template>
  <div class="comment">
    <h4>{{ props.author }} 說：</h4>
    <p>{{ props.content }}</p>
  </div>
</template>

<style>
.comment {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
}
</style>
```

**`App.vue` (父層)**
```html
<script setup>
import Comment from './Comment.vue';
</script>

<template>
  <h1>今日評論</h1>
  <!-- 使用第一塊積木，並為它配置 author 和 content -->
  <Comment author="小明" content="Vue 3 真的太棒了！" />
  <!-- 使用第二塊積木，並給予不同配置 -->
  <Comment author="老師" content="同學，你的作業寫得很好。" />
</template>
```
> 在這個例子中，父層 (`App.vue`) 透過像 HTML attribute 一樣的方式 (`author="..."`)，將不同的配置傳遞給了每一塊評論積木。積木則忠實地根據接收到的配置來顯示內容。

#### 單向數據流：一個重要的規則

Vue 強制規定了「單向數據流」。這意味著積木（子組件）**絕對不能**直接修改從父層傳來的配置（prop）。

**為什麼？** 這能防止子組件意外地改變父層的狀態，讓應用的資料流變得難以追蹤。如果資料可以隨意雙向流動，一旦出錯，你很難定位問題的根源。

### Emit：當積木上的按鈕被「按下」

既然配置只能從父層向下傳遞，那子組件要如何通知父層發生了某件事呢？（例如：「嘿，有使用者點擊我了！」）

答案是 **`$emit`**。子組件可以透過「觸發事件 (emitting an event)」的方式來向父層發出通知。

**舉個例子**：我們來做一個「按讚積木」(`LikeButton.vue`)。它本身不管理總數，只負責在被點擊時，向外發出一個「被點了」的通知。

**`LikeButton.vue` (按讚積木)**
```html
<script setup>
// 1. 這塊積木宣告：我可能會發出一個名為 'add-like' 的通知
const emit = defineEmits(['add-like']);

function onButtonClick() {
  // 2. 發出通知，並可以夾帶一些資訊（例如：這次增加了 1 個讚）
  emit('add-like', 1); 
}
</script>

<template>
  <button @click="onButtonClick">
    幫我按讚 👍
  </button>
</template>
```

**`App.vue` (父層)**
```html
<script setup>
import { ref } from 'vue';
import LikeButton from './LikeButton.vue';

const totalLikes = ref(0);

// 3. 父層準備好一個函式，用來處理接收到的通知
function handleAddLike(amount) {
  totalLikes.value += amount;
}
</script>

<template>
  <h2>總按讚數：{{ totalLikes }}</h2>
  
  <!-- 4. 父層用 @ 來監聽來自積木的 'add-like' 通知 -->
  <LikeButton @add-like="handleAddLike" />
</template>
```
> 在這裡，按讚積木並不知道總讚數是多少，它只負責在被點擊時發出「`add-like`」的通知。父層聽到了這個通知，並執行 `handleAddLike` 函式來更新自己的 `totalLikes` 狀態。這就是一個清晰、解耦的溝通模式。

### Bonus：讓你的積木支援 `v-model`！

你可能會有個疑問：我們在 Day 2 學到的 `v-model` 只能用在原生的 `<input>` 上嗎？當然不！任何積木都可以透過 `Props` 和 `Emit` 的組合，來支援 `v-model`。

`v-model` 在組件上，其實是一個簡寫，它等同於：
1.  傳遞一個名為 `modelValue` 的 **prop**。
2.  監聽一個名為 `update:modelValue` 的**自訂事件**。

**何時使用？**
當你開發的**自訂積木**其核心功能就像一個「輸入框」時，就非常適合使用。例如：一個自訂的搜尋欄、一個價格滑桿、一個富文本編輯器等。

**實作一個 `CustomInput.vue`：**
```html
<!-- CustomInput.vue (子組件) -->
<script setup>
// 1. 接收來自 v-model 的 prop
defineProps(['modelValue']);

// 2. 宣告 v-model 需要的事件
const emit = defineEmits(['update:modelValue']);

function onInput(event) {
  // 3. 當內部 input 值改變時，觸發事件通知父層更新
  emit('update:modelValue', event.target.value);
}
</script>

<template>
  <input 
    :value="modelValue" 
    @input="onInput"
    placeholder="這是一個自訂輸入框"
  >
</template>
```

**父層如何使用：**
```html
<!-- App.vue (父組件) -->
<script setup>
import { ref } from 'vue';
import CustomInput from './CustomInput.vue';

const message = ref('');
</script>

<template>
  <CustomInput v-model="message" />
  <p>你輸入的內容是：{{ message }}</p>
</template>
```
> 看到了嗎？我們遵循了 `prop` 向下傳，`emit` 向上通知的規則，就輕鬆實現了 `v-model` 的雙向綁定效果。這讓我們的自訂積木使用起來就跟原生元素一樣方便！

## 本篇自我挑戰

- **思考一：單向數據流的重要性**
  如果 Vue 允許積木隨意修改從父層那裡得到的配置（props），在一個大型專案中可能會發生什麼混亂的情況？

- **思考二：設計一個「刪除按鈕」積木**
  假設你要做一個 `DeleteButton` 積木，用在一個文章列表的每一項後面。它被點擊時，需要通知父層將對應的文章刪除。你會如何使用 `props` 和 `emit` 來設計這個積木？(提示：它可能需要接收一個 `postId` 作為 prop)。

- **思考三：設計一個 `v-model` 組件**
  除了輸入框，你還能想到什麼情境適合用 `v-model` 來自訂組件？例如一個可以切換開關狀態的 `Switch.vue` 組件？它應該接收什麼 `modelValue`，又該在何時 `emit` 事件呢？

## 總結

今天我們踏入了 Vue 組件化的大門，學會了最重要的兩個溝通工具：
-   **Props**：實現了從父到子的**單向數據流**，如同為積木進行初始配置，讓資料傳遞清晰可控。
-   **Emit**：允許子組件向父組件發送訊息，如同積木上的按鈕發出通知，實現了從子到父的**事件回報**。
-   **`v-model` on Components**：`props` 和 `emit` 的組合應用，可以讓我們創造出支持雙向綁定的高複用性組件。

掌握了這些，你就掌握了組件之間最核心的溝通方式。這讓我們的應用程式不再是一團亂麻，而是一座由無數個職責分明、可獨立運作的樂高積木搭建起來的宏偉城堡。

### 深度思考：`v-model` vs. 單獨的 Props/Emit

看到 `v-model` 的方便，你可能會問：「那為什麼不全部都用 `v-model` 就好了？」

這是一個好問題。`v-model` 雖然方便，但它有特定的使用情境。它最適合用在**「一個」**主要的、需要雙向同步的**「值」**上，就像輸入框的值一樣。但如果你的組件需要更複雜的溝通，濫用 `v-model` 反而會讓程式碼更難理解。

**情境區分：**
-   **使用 `v-model`**：當你的組件行為像一個**表單控件**時。例如：`CustomInput`、`MyCheckbox`、`ColorPicker`。這些組件的核心職責就是管理一個值。
-   **使用 Props/Emit**：當你的組件需要**多個輸入配置**，或需要回報**多種不同事件**時。例如一個「文章卡片」組件，它可能接收 `title`、`author` 等多個 `props`，並可能發出 `@like`、`@share`、`@comment` 等多種 `emit`。如果這些都用 `v-model`，語意上會非常混亂，也讓父層難以追蹤是哪個互動觸發了值的改變。

**結論：** 請將 `v-model` 視為一個強大的「語法糖」，而不是萬靈丹。當語意符合「值」的雙向綁定時，大膽使用它；對於其他更複雜的互動，明確的 `Props` 和具名的 `Emit` 事件會是更清晰、更易維護的選擇。

### 版本小筆記

在組件上使用 `v-model` 的概念並不是 Vue 3 才有的新功能，Vue 2 就已經支援。但它們的實作方式不同：
-   **Vue 2**: 預設使用 `value` prop 和 `input` 事件。
-   **Vue 3**: 預設使用 `modelValue` prop 和 `update:modelValue` 事件。

Vue 3 的改動讓 `v-model` 的運作更清晰，並且還能透過 `v-model:參數名` 的方式在一個組件上綁定多個 `v-model`（這是更進階的用法）。只要你使用的是 Vue 3，本篇教學介紹的 `modelValue` 寫法就能直接使用，不需要任何額外升級。

## 本日關鍵字回顧

- `Component`: 組件，Vue 應用的基本單位，可複用、獨立的「樂高積木」。
- `Props`: 屬性，父組件向子組件傳遞資料的方式（配置）。
- `defineProps`: 在 `<script setup>` 中用來宣告組件可接收的 props。
- `Emit`: 事件觸發，子組件向父組件通信的方式（通知）。
- `defineEmits`: 在 `<script setup>` 中用來宣告組件可觸發的事件。
- `One-Way Data Flow`: 單向數據流，Vue 的核心原則，資料只能從父級流向子級。
- `v-model (on component)`: 實現組件雙向綁定的語法糖。
- `modelValue`: `v-model` 預設使用的 prop 名稱。
- `update:modelValue`: `v-model` 預設監聽的事件名稱。

光有資料傳遞還不夠，如果我們想讓組件的「內容」也變得更有彈性呢？明天，我們將探索 `Slots` 的魔法，學習如何打造真正高彈性、可複用的萬用組件。敬請期待！
