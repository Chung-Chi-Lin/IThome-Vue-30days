# 【Day 14】當個稱職的表單警衛：用 v-model 與驗證守護你的數據大門

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

嘿，各位 Vue 的魔法師們！前幾天我們學會了怎麼跟後端 API 大哥「喬事情」，拿數據、管數據。但很多時候，數據的源頭，其實是我們親愛的使用者在前端戳戳點點、辛苦填寫的表單。

今天，我們要來聊聊前端應用程式中那個既重要又讓人抓狂的環節——**表單處理與驗證**！

想像一下，你的應用程式是個高級私人會所，而你的表單就是門口的那個**警衛**。使用者輸入的每一筆資料，都像是想進來 high 的客人。身為一個稱職的警衛，你總不能讓阿貓阿狗隨便闖進來吧？你得確保他們「衣著得體」（數據格式正確）、「帶了邀請函」（必填項已填寫），才能放行。

一個好的表單，不僅能讓客人（使用者）感覺流暢舒適，還能有效攔截那些想來搗亂的「奧客」（無效數據），保護你家會所的安寧！

## 表單數據綁定：`v-model` 的讀心術

在 Vue 的世界裡，要馴服表單這頭猛獸，最厲害的法寶就是 `v-model`。它就像一對有心電感應的「對講機」，使用者在輸入框裡說什麼，你 script 裡的數據就立刻聽到；你 script 裡的數據更新了，輸入框的內容也跟著變。這就是所謂的「**雙向綁定**」，一條龍服務，夠貼心吧！

### 基本用法

`v-model` 可以用在各種表單元素上，就像萬能鑰匙一樣。

```html
<script setup>
import { ref } from 'vue';

// 就像警衛手上的登記簿，準備記錄客人的資料
const username = ref(''); // 客人姓名
const message = ref(''); // 客人想留的悄悄話
const selectedOption = ref('A'); // 客人選的酒
const isChecked = ref(false); // 客人是否同意遵守會所規定
const checkedNames = ref([]); // 一起來的 VIP 賓客名單
const picked = ref('One'); // 客人選的包廂
</script>

<template>
  <div>
    <h2>基本文字輸入 (Text)</h2>
    <label>用戶名:</label>
    <input type="text" v-model="username" placeholder="請報上大名">
    <p>登記簿上的名字: {{ username }}</p>

    <h2>多行文本 (Textarea)</h2>
    <label>留言:</label>
    <textarea v-model="message" placeholder="有什麼想對我說的嗎？"></textarea>
    <p>他說: {{ message }}</p>

    <h2>下拉選單 (Select)</h2>
    <label>選擇選項:</label>
    <select v-model="selectedOption">
      <option disabled value="">請選擇</option>
      <option>A</option>
      <option>B</option>
      <option>C</option>
    </select>
    <p>他點了: {{ selectedOption }}</p>

    <h2>單選框 (Checkbox)</h2>
    <input type="checkbox" id="checkbox" v-model="isChecked">
    <label for="checkbox">我同意本店的霸王條款</label>
    <p>是否同意: {{ isChecked ? '他同意了！' : '還在猶豫...' }}</p>

    <h2>多選框 (Checkboxes)</h2>
    <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
    <label for="jack">Jack</label>
    <input type="checkbox" id="john" value="John" v-model="checkedNames">
    <label for="john">John</label>
    <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
    <label for="mike">Mike</label>
    <p>他的朋友們: {{ checkedNames }}</p>

    <h2>單選按鈕 (Radio Buttons)</h2>
    <input type="radio" id="one" value="One" v-model="picked">
    <label for="one">包廂 A</label>
    <br>
    <input type="radio" id="two" value="Two" v-model="picked">
    <label for="two">包廂 B</label>
    <p>他選了: {{ picked }}</p>
  </div>
</template>
```

### `v-model` 的龜毛朋友們 (修飾符)

`v-model` 不是一個人來的，它還帶了幾個有特殊專長的「助理」，也就是修飾符，讓你的數據處理更精準。

*   **`.lazy` (懶人助理)**：一般的 `v-model` 很急，使用者打一個字，它就報告一次。但懶人助理不一樣，它會等到使用者忙完、輸入框失去焦點（`change` 事件）後，才慢悠悠地去同步數據。適合不喜歡被打擾的場景。
    ```html
    <!-- 等客人寫完信再收，而不是寫一個字看一次 -->
    <input type="text" v-model.lazy="username">
    ```
*   **`.number` (數學家助理)**：這位助理有強迫症，會自動把使用者輸入的值轉成數字。如果轉不成功，它也就算了，直接回報原始的文字。
    ```html
    <!-- 自動把年齡從 "18" 文字變成 18 數字 -->
    <input type="text" v-model.number="age">
    ```
*   **`.trim` (潔癖助理)**：這位助理最討厭多餘的空白。它會自動把使用者輸入內容前後的空白全部擦掉，讓數據乾乾淨淨。
    ```html
    <!-- "  admin@example.com  " -> "admin@example.com" -->
    <input type="text" v-model.trim="email">
    ```

## 表單驗證：警衛的「安檢」SOP

光有登記簿還不夠，警衛還得有套「安檢 SOP」來確保客人沒問題。這就是表單驗證。在前端做即時驗證，就像在門口就提醒客人：「先生，你的領帶歪了」，能大大提升使用者體驗。

### 1. 基本 HTML5 驗證 (公發的簡易對講機)

你可以直接用 HTML5 自帶的驗證屬性，比如 `required`、`minlength`、`type="email"`。瀏覽器會幫你做最基本的檢查和提示。這就像警衛用公發的對講機喊話，簡單方便，但功能有限，樣式也不好客製。

```html
<form>
  <!-- required: 這位客人，請務必報上名來！ -->
  <input type="email" required placeholder="請輸入 Email">
  <!-- minlength: 密碼太短了，像金魚的記憶一樣不可靠！ -->
  <input type="password" minlength="6" placeholder="密碼至少6位">
  <button type="submit">提交</button>
</form>
```

### 2. 手動 JavaScript 驗證 (客製化的精銳裝備) (推薦)

想玩點花的，就需要動用 JavaScript 來打造你自己的驗證軍火庫。這通常包含：

*   定義你的「黑名單」規則。
*   在客人準備闖關時（輸入或提交）觸發安檢。
*   如果發現問題，立刻亮起紅燈（顯示錯誤訊息）。

```html
<script setup>
import { ref, computed } from 'vue';

const email = ref('');
const password = ref('');
const emailError = ref('');
const passwordError = ref('');

// 安檢 SOP 1: 檢查 Email
const validateEmail = () => {
  if (!email.value) {
    emailError.value = 'Email 是必填的，別想混進來！';
  } else if (!/^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$/.test(email.value)) {
    emailError.value = '你這 Email 看起來像假的，再檢查一下！';
  } else {
    emailError.value = ''; // 沒問題，放行
  }
};

// 安檢 SOP 2: 檢查密碼
const validatePassword = () => {
  if (!password.value) {
    passwordError.value = '沒密碼？你想靠臉進來嗎？';
  } else if (password.value.length < 6) {
    passwordError.value = '密碼太短了，至少要 6 個字！';
  } else {
    passwordError.value = ''; // 沒問題，放行
  }
};

// 警衛的最終判斷：所有 SOP 都通過了嗎？
const isFormValid = computed(() => {
  // 確保所有錯誤訊息都是空的，而且該填的都填了
  return !emailError.value && !passwordError.value && email.value && password.value;
});

// 處理提交事件：按下「註冊」按鈕的那一刻
const handleSubmit = () => {
  // 在客人衝進來之前，再完整檢查一遍
  validateEmail();
  validatePassword();

  if (isFormValid.value) {
    alert('歡迎光臨！資料已提交！');
    // 在這裡把數據送到後端老大那裡
  } else {
    alert('等等！你的資料有問題，先去旁邊修正一下！');
  }
};
</script>

<template>
  <div>
    <h2>會所註冊表單</h2>
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="email">Email:</label>
        <!-- @blur: 客人一移開視線，就立刻檢查 -->
        <input type="text" id="email" v-model="email" @blur="validateEmail" @input="validateEmail">
        <p v-if="emailError" style="color: red;">{{ emailError }}</p>
      </div>
      <div>
        <label for="password">密碼:</label>
        <input type="password" id="password" v-model="password" @blur="validatePassword" @input="validatePassword">
        <p v-if="passwordError" style="color: red;">{{ passwordError }}</p>
      </div>
      <!-- 如果表單無效，註冊按鈕就像拉起的吊橋，按不了 -->
      <button type="submit" :disabled="!isFormValid">註冊</button>
    </form>
  </div>
</template>
```

### 3. 第三方驗證函式庫 (外包的菁英保全公司) (推薦)

如果你的會所規模宏大，規則超多，自己當警衛太累了。這時候，花點錢請「菁英保全公司」是個聰明的選擇。這些第三方驗證函式庫能讓你事半功倍。

*   **VeeValidate**：功能超強的保全公司，提供各種現成的安檢規則、漂亮的錯誤報告、還能幫你管理整個保全團隊的狀態。
*   **Vuelidate**：另一家知名的保全公司，以簡潔的語法和基於模型的驗證著稱。

用了它們，你就像是坐在監控室裡翹著二郎腿的總管，看著保全們自動化地處理各種狀況，輕鬆又高效。

## 小技巧與警衛守則：

*   **即時回饋**：別等客人走到大廳了才跟他說他鞋子穿反了。在他輸入的時候就給提示 (`input` 或 `blur` 事件)，使用者體驗會好很多。
*   **錯誤訊息要說人話**：不要只顯示「錯誤！」。要說清楚：「密碼太短了，至少需要 6 位」，讓客人知道怎麼改。
*   **禁用提交按鈕**：如果客人資料不合格，就把大門的「提交」按鈕鎖起來（禁用），防止他硬闖。
*   **後端驗證才是王道**：**記住！前端警衛是可以被收買或繞過去的（使用者可以禁用 JS）！** 所有數據在進入你家金庫（資料庫）之前，都必須經過**後端那位鐵面無私的大法官**再次嚴格審查。**永遠不要完全相信來自前端的任何數據！**
*   **提供重置按鈕**：給客人一個「我全搞砸了，重來吧」的機會，讓他可以一鍵清空所有輸入。

## 本篇自我挑戰

-   **今日挑戰**：設計一個包含至少三個輸入框（例如：姓名、Email、密碼）的註冊表單。為每個輸入框添加基本的驗證規則（例如：必填、Email 格式、密碼長度），並在用戶輸入時即時顯示錯誤訊息。嘗試在表單提交時，判斷所有驗證是否通過。
-   **反思**：在你心目中，前端警衛（前端驗證）和後端大法官（後端驗證）各自扮演什麼角色？為什麼說兩者缺一不可？

## 總結

今天我們學會了如何當一個稱職的表單警衛。我們掌握了 `v-model` 這對心電感應對講機，認識了它那幾位性格各異的助理（修飾符），還學會了如何制定一套從基本到菁英級別的安檢 SOP（驗證）。

記住，一個健壯的表單是你應用程式數據品質的第一道防線。前端驗證與後端驗證的完美配合，才能打造出固若金湯的數據堡壘。

本日關鍵字回顧

-   **`v-model`**: 表單雙向數據綁定，像心電感應對講機。
-   **`v-model` 修飾符**: `.lazy` (懶人), `.number` (數學家), `.trim` (潔癖)。
-   **表單驗證**: 警衛的安檢 SOP。
-   **HTML5 驗證**: 公發的簡易驗證工具。
-   **前端驗證**: 提升使用者體驗的門口警衛。
-   **後端驗證**: 保障數據安全的最終大法官。
-   **VeeValidate / Vuelidate**: 可以外包的菁英保全公司。

明天，我們將踏入一個讓程式碼「紀律嚴明」的新領域——TypeScript！學習如何在 Vue 專案中引入型別，讓你的程式碼從「游擊隊」升級為「正規軍」！