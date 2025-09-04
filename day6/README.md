# 【Day 6】探索組件生命週期：掌握 Vue 組件的生老病死

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！前幾天我們學會了如何讓畫面動起來，今天我們要更進一步，學會如何掌握這些畫面上元素的「生命」。想像一下，你的 Vue 組件就像一個個有生命的個體，它們從出生、成長、活躍，到最後功成身退，每個階段都有它獨特的「心跳」和「任務」。而我們今天要學的，就是如何傾聽這些心跳，並在關鍵時刻給予它們正確的指令！

每個 Vue 組件從被創造出來，到被放到網頁上跟大家見面，再到資料更新時的「變身」，直到最後功成身退從網頁上消失，都會經歷一連串的「人生階段」。在這些關鍵時刻，Vue 會很貼心地觸發一些「生命週期鉤子函式」(Lifecycle Hooks)，就像是組件人生中的重要「里程碑」事件。掌握這些鉤子，你就能在組件的「生老病死」各個環節中，精準地插入你的魔法，處理資料、執行副作用、清理資源等。

## 組件生命週期概覽：組件的「人生」旅程

想像你的 Vue 組件，就像一個從出生到退休的「人生」旅程。每個生命週期鉤子，就是它人生中的重要「里程碑」，Vue 會在這些里程碑上敲敲門，問你：「嘿，現在發生了這件事，你有什麼想做的嗎？」

1.  **建立階段 (Creation)**：組件的「誕生」
    *   `beforeCreate` (Options API): 就像嬰兒剛出生，還沒看到世界，身體機能（資料觀察、事件/計算屬性）都還沒準備好。這時候組件實例剛被建立，但你還摸不到 `data` 或 `methods`。
    *   `created` (Options API): 嬰兒的身體機能已經準備好了！組件實例已建立，資料觀察和事件/計算屬性都已設定完畢，你可以開始訪問 `data` 和 `methods` 了。但它還沒被放到網頁上，所以還看不到真實的 DOM。
    *   **小補充 (Composition API)**：在 Composition API 中，`beforeCreate` 和 `created` 的邏輯，通常直接寫在 `setup()` 函式裡面就行了。因為 `setup()` 執行時，組件的響應式系統就已經準備好了，非常方便！

2.  **掛載階段 (Mounting)**：組件的「登場」
    *   `beforeMount`: 組件即將被「抱」到網頁上，準備跟大家見面了！這時候模板已經編譯完成，但還沒渲染到真實 DOM。
    *   `mounted`: 燈光、音效、掌聲！組件終於被「放」到網頁上，第一次跟大家見面了！這時候，你就可以開始讓它做一些需要跟網頁互動的事情，比如去跟後端打聲招呼（發送 API 請求），或是跟其他網頁元素玩耍（操作 DOM）。**這是你第一次能真正摸到組件在網頁上的樣子！**

3.  **更新階段 (Updating)**：組件的「變身」
    *   `beforeUpdate`: 組件長大了，資料變了，就像你換了新衣服、剪了新髮型，需要更新一下外觀。Vue 偵測到響應式資料變化，準備重新渲染組件了。
    *   `updated`: 新衣服穿好了，新髮型也剪好了！組件已經更新完畢，網頁上的真實 DOM 也已經是最新狀態了。如果你需要在資料更新後，對 DOM 進行一些操作，可以在這裡進行。
    *   **小技巧**：如果你在 `onUpdated` 裡操作 DOM，但發現資料更新後 DOM 卻還沒完全同步，可以試試 `nextTick`。它會確保你的操作在 DOM 更新週期結束後執行，就像是：「等 Vue 把所有該更新的都更新完，你再動手！」

4.  **銷毀階段 (Unmounting)**：組件的「謝幕」
    *   `beforeUnmount`: 組件功成身退，要從網頁上「畢業」了！它還在網頁上，但即將被移除。這時候是做最後清理的好時機。
    *   `unmounted`: 謝幕完成，組件已從網頁上「消失」了。所有事件監聽器、子組件、響應式副作用都已經被清理乾淨。它已經不再佔用任何資源了。
    *   **地雷警告**：如果你在 `onMounted` 裡設定了定時器（`setInterval`）或添加了全局事件監聽器，但沒有在 `onUnmounted` 裡清理掉，那這些「幽靈」程式碼會繼續在背景運行，佔用記憶體，導致你的網頁越來越慢，甚至崩潰！所以，**有借有還，再借不難**，記得清理！

## 父子組件的生命週期順序：誰先誰後？

當你的應用程式不只一個組件，而是由父組件、子組件、甚至孫組件層層嵌套時，它們的生命週期鉤子執行順序就變得很有趣了！想像這是一個「表演團隊」的準備與撤場過程：

### 1. 掛載階段 (Mounting)：從外到內準備，從內到外登場

當父組件要掛載時，它會先通知子組件準備，子組件再通知孫組件準備。但實際「登場」的順序，則是從最內層的孫組件開始。

*   **準備 (`beforeMount`) 順序**：**父組件** -> **子組件** -> **孫組件**
    *   就像導演（父）先說「準備開始」，然後通知舞台組（子）「準備」，舞台組再通知燈光音響（孫）「準備」。
*   **登場 (`mounted`) 順序**：**孫組件** -> **子組件** -> **父組件**
    *   當燈光音響（孫組件）準備好並「就位」後，舞台組（子組件）才能「就位」，最後導演（父組件）才宣布「表演開始」。

### 2. 更新階段 (Updating)：資料變動，層層遞進

當父組件的資料發生變化，導致子組件和孫組件也需要更新時，更新的準備和完成順序也類似掛載。

*   **準備 (`beforeUpdate`) 順序**：**父組件** -> **子組件** -> **孫組件**
*   **完成 (`updated`) 順序**：**孫組件** -> **子組件** -> **父組件**

### 3. 銷毀階段 (Unmounting)：從外到內準備，從內到外撤場

當父組件要被銷毀時，它會先通知子組件準備銷毀，子組件再通知孫組件準備銷毀。但實際「撤場」的順序，也是從最內層的孫組件開始。

*   **準備 (`beforeUnmount`) 順序**：**父組件** -> **子組件** -> **孫組件**
    *   就像導演（父）先說「準備撤場」，然後通知舞台組（子）「準備撤」，舞台組再通知燈光音響（孫）「準備撤」。
*   **撤場 (`unmounted`) 順序**：**孫組件** -> **子組件** -> **父組件**
    *   當燈光音響（孫組件）「撤離」後，舞台組（子組件）才能「撤離」，最後導演（父組件）才宣布「全部撤場完畢」。

## Composition API 中的生命週期鉤子：更直覺的魔法咒語

在 Composition API 中，生命週期鉤子都變成以 `on` 開頭的函式形式，例如 `onMounted`、`onUnmounted`。它們必須在 `setup()` 函式中同步呼叫。這讓相關邏輯可以更好地組織在一起，就像把同一個功能的魔法咒語都寫在同一個章節裡，一目瞭然。

```javascript
<script setup>
import { ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted, nextTick } from 'vue';

const message = ref('Hello Vue!');
const domElement = ref(null); // 用來獲取模板中的元素

onBeforeMount(() => {
  console.log('onBeforeMount: 組件即將掛載到 DOM，但還摸不到真實 DOM 元素。');
});

onMounted(() => {
  console.log('onMounted: 組件已掛載到 DOM，可以操作真實 DOM 了！');
  // 例如：初始化第三方函式庫、發送 API 請求
  console.log('真實 DOM 元素:', domElement.value); // 這時候 domElement.value 已經有值了
  // 假設這裡要發送一個 API 請求
  // fetchSomeData();
});

onBeforeUpdate(() => {
  console.log('onBeforeUpdate: 組件即將更新 DOM，資料已變，但畫面還沒變。');
});

onUpdated(() => {
  console.log('onUpdated: 組件已更新 DOM，畫面也跟著變了！');
  // 如果需要在資料更新後操作 DOM，且確保 DOM 已完全更新，可以使用 nextTick
  nextTick(() => {
    console.log('nextTick 確保 DOM 已更新:', domElement.value.textContent);
  });
});

onBeforeUnmount(() => {
  console.log('onBeforeUnmount: 組件即將銷毀，做最後的告別與清理。');
  // 例如：清理定時器、移除事件監聽器
  // clearInterval(myInterval);
  // window.removeEventListener('resize', handleResize);
});

onUnmounted(() => {
  console.log('onUnmounted: 組件已銷毀，所有資源都已釋放，再見了！');
});

// 點擊按鈕改變 message，觸發更新階段
const changeMessage = () => {
  message.value = 'Vue Lifecycle is awesome!';
};
</script>

<template>
  <div ref="domElement">
    <p>{{ message }}</p>
    <button @click="changeMessage">改變訊息</button>
  </div>
</template>
```

## 常見應用場景：什麼時候該用哪個鉤子？

-   **`onMounted`**: 組件「出生」後，第一次跟世界打招呼！
    *   **發送初始 API 請求**：組件一出現，就去跟後端要資料，這是最常見的用法。
    *   **初始化需要訪問 DOM 的第三方函式庫**：比如圖表庫 (Chart.js)、地圖 (Google Maps API) 等，它們需要組件的真實 DOM 元素才能初始化。
    *   **添加全局事件監聽器**：例如監聽視窗大小變化 (`resize`) 或滾動事件 (`scroll`)。**切記：在 `onUnmounted` 中移除！**

-   **`onUpdated`**: 組件「變身」後，需要做點什麼！
    *   當組件的資料更新後，需要執行某些 **依賴於最新 DOM 狀態** 的操作時。
    *   **注意**：應盡量避免在 `onUpdated` 中直接修改響應式資料，這可能導致無限循環更新，讓你的應用程式陷入「更新地獄」！

-   **`onUnmounted`**: 組件「謝幕」前，收拾殘局！
    *   **清理在 `onMounted` 或其他地方設定的副作用**：這是防止記憶體洩漏的關鍵！
    *   **移除事件監聽器**：例如 `window.removeEventListener`。
    *   **清除定時器**：例如 `clearInterval` 或 `clearTimeout`。
    *   **取消訂閱**：如果你有使用 WebSocket 或其他訂閱服務，在這裡取消訂閱。

## 本篇自我挑戰

-   **思考一：清理的重要性**
    為什麼在 `onMounted` 中添加的全局事件監聽器，必須在 `onUnmounted` 中移除？如果不移除會發生什麼問題？（提示：想想「幽靈」組件和記憶體洩漏）

## 總結

今天我們深入探索了 Vue 組件的生命週期，了解了組件從誕生到消亡的各個階段，以及在這些階段中可以利用的生命週期鉤子函式。掌握這些鉤子，就像掌握了組件的「人生劇本」，能讓你更精準地控制組件的行為，編寫出更健壯、更高效的 Vue 應用程式。

本日關鍵字回顧

-   **生命週期鉤子 (Lifecycle Hooks)**: 在組件生命週期特定階段執行的函式。
-   **`onBeforeMount`**: 掛載前，組件即將登場。
-   **`onMounted`**: 掛載後，組件已登場，可操作真實 DOM。
-   **`onBeforeUpdate`**: 更新前，組件即將變身。
-   **`onUpdated`**: 更新後，組件已變身，DOM 已更新。
-   **`onBeforeUnmount`**: 銷毀前，組件即將謝幕，做最後清理。
-   **`onUnmounted`**: 銷毀後，組件已謝幕，所有資源已釋放。
-   **`nextTick`**: 確保在 DOM 更新週期結束後執行操作的小技巧。

明天，我們將正式進入 Vue 3 的主力語法糖——`Composition API` 的入門，學習 `<script setup>` 如何改善程式碼組織。準備好迎接 Vue 3 的真正魅力了嗎？
