# 【Day 1】Vue 3 核心演進：不只是改版，更是思維的躍遷

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

今年，再次挑戰自我！

也謝謝我的好朋友們，邀請我繼續參加這場「攀登技術頂峰」的旅程——雖然山頂看似遙不可及，但沿途的風景與 **新的技能點**，永遠是最好的回報。

今年，我們的隊伍甚至還多了一些新勇士！

AI 的出現，確實改變了很多事。工具與技術的更新速度，快到讓人焦慮。如果不持續學習，我們很快就會變成只懂一身潮牌、卻沒有實力的「**球場裝備哥**」。

所以，我決定繼續挑戰鐵人賽，用學習來對抗這份焦慮。

下方是這次 30 天的挑戰目錄，同時，我也會保留互動的傳統，**每天的自我挑戰**都非常歡迎大家一起來留言、討論！

為了應對這個 AI 時代，我稍微改變了今年的寫作風格。

我會刻意在文章中放入更多 **關鍵的專有名詞**。因為我發現，無論是問 AI 還是查 Google，你都需要有「主動意識」和「精準的關鍵字」才能挖到真正的寶藏。

畢竟，AI 通常只會回答你所問的，而不會主動教你那些能讓你功力大增的「武功祕笈」名稱！
## 目錄
| 天數 | 主題 | 描述 |
| :--- | :--- | :--- |
| Day 1 | Vue 3 核心演進 | 介紹 Vue 3 相較於 Vue 2 的核心優勢與設計理念。 |
| Day 2 | 資料綁定與事件處理 | 深入 Vue 的響應式核心：資料綁定與事件處理的最佳實踐。 |
| Day 3 | 條件與列表渲染 | 掌握 `v-if` 與 `v-for` 的高效使用場景與效能考量。 |
| Day 4 | 組件化基礎 (Props & Emit) | 探討 Props 單向數據流與 Emit 事件的標準用法。 |
| Day 5 | 組件進階 (Slots & 動態組件) | 學習使用 Slot 與動態組件，打造高彈性、可複用的元件。 |
| Day 6 | 探索組件生命週期 | 了解組件從建立到銷毀的完整過程與各階段的應用。 |
| Day 7 | Composition API 入門 (`<script setup>`) | 學習 Vue 3 的主力 `setup` 語法糖，以及它如何改善程式碼組織。 |
| Day 8 | `ref` vs `reactive` 的選擇 | 深入探討 `ref` 與 `reactive` 的核心差異與最佳使用時機。 |
| Day 9 | `computed` vs `watch` 的權衡 | 分析計算屬性與監聽器的不同應用場景，避免誤用。 |
| Day 10 | 打造自己的 `Composable` (自訂 Hook) | 學習封裝與複用邏輯，建立帶有型別的自訂 Hook。 |
| Day 11 | Vue Router 路由管理 | 掌握前端路由設計，包含動態路由與巢狀路由。 |
| Day 12 | Pinia 狀態管理 | 學習使用新一代的狀態管理庫 Pinia，管理跨組件的共享資料。 |
| Day 13 | API 串接與狀態管理 | 整合 Axios 進行 API 請求，並優雅地處理 loading 與 error 狀態。 |
| Day 14 | 表單處理與驗證 | 探討 Vue 中處理複雜表單的資料綁定與驗證策略。 |
| Day 15 | 在 Vue 中引入 TypeScript | 了解在 Vue 專案中設定與使用 TypeScript 的基礎。 |
| Day 16 | 型別化 Props 與 Emits | 使用 `defineProps` 與 `defineEmits` 的 TS 寫法，增強組件的穩定性。 |
| Day 17 | `ref`、`reactive` 的型別推論 | 學習 TypeScript 如何自動推斷響應式資料的型別。 |
| Day 18 | 為 `Composable` 加上型別 | 延續 Day 10，為自訂 Hook 加上完整的型別定義。 |
| Day 19 | SCSS 整合與模組化 | 學習在 Vue 中整合 SCSS 並使用 CSS Modules 避免樣式衝突。 |
| Day 20 | 使用 SCSS 變數與 Mixin | 透過 SCSS 建立可維護的全域樣式系統。 |
| Day 21 | 建立共用 UI 元件庫策略 | 探討如何規劃與建立團隊內可共享的基礎 UI 元件。 |
| Day 22 | Vitest 單元測試入門 | 學習使用 Vitest 為 Vue 元件與 Composable 撰寫單元測試。 |
| Day 23 | 測試 Props、事件與 Slot | 針對 Vue 組件的各種互動情境進行測試。 |
| Day 24 | 模擬使用者行為 (Testing Library) | 使用 Testing Library 撰寫更貼近真實使用者操作的測試案例。 |
| Day 25 | Mocking API 請求 | 學習如何在測試中模擬 API 回應，讓測試更穩定。 |
| Day 26 | 單元測試 vs E2E 測試 | 了解不同測試類型的目的與使用時機。 |
| Day 27 | Vue 專案效能優化 | 分享 `keep-alive`、`v-memo` 與虛擬滾動等實用的效能優化技巧。 |
| Day 28 | Code Splitting 與 Lazy Loading | 學習如何對路由與組件進行代碼分割，加速頁面載入。 |
| Day 29 | 專案重構與可維護性 | 探討如何識別壞味道 (Bad Smell) 並進行程式碼重構。 |
| Day 30 | Vue 的未來與生態系 | 總結 Vue 的開發最佳實踐，並展望未來的發展趨勢。 |

## 介紹：嘿，老朋友，我們聊聊 Vue 3

嘿，各位 Vue 的老司機們！還記得當年我們在 Vue 2 的世界裡，那個充滿 `data`、`methods`、`computed` 的 Options API 嗎？日子過得挺滋潤的，直到 Vue 3 帶著一位名叫「Composition API」的新朋友橫空出世，江湖從此不再平靜。

很多人第一眼看到 Vue 3，可能會想：「啊？又要學新東西？我 Vue 2 用得好好的，幹嘛沒事找事？」

如果你曾有過這種想法，那這篇文章就是為你準備的。我們不談虛的，就來聊聊 Vue 3 到底強在哪，以及它如何讓我們寫出更優雅、更強壯的程式碼。這不是一次簡單的升級，這是一次開發思維的進化！

## 為什麼我們需要 Vue 3？Vue 2 有什麼不好？

這麼說吧，Vue 2 就像一輛很棒的家用車，應付日常通勤、買菜、接送小孩都綽綽有餘。但當你今天想把車開上賽道，參加一場專業比賽時，你就會發現家用車的極限了。

隨著專案規模越來越大、邏輯越來越複雜，Vue 2 的一些問題就慢慢浮現了：

1.  **邏輯分散的困擾**：在大型組件中，一個功能（例如：使用者登入）的相關程式碼，會被 `data`、`methods`、`computed`、`watch` 等選項拆得支離破碎。想修改一個功能，你得在檔案裡上下反覆橫跳，像在玩打地鼠一樣，心累不？

2.  **TypeScript 的貌合神離**：Vue 2 對 TypeScript 的支援有點像硬湊合的，雖然能用，但總覺得哪裡卡卡的，型別推斷能力有限，開發體驗稱不上絲滑。

3.  **效能的天花板**：Vue 2 的響應式系統是基於 `Object.defineProperty`，它在初始化時需要遍歷所有屬性，有效能瓶頸。而且 Virtual DOM 的比對演算法也還有優化空間。

Vue 3 的出現，就是為了解決這些「甜蜜的負擔」。

## Vue 3 的三大殺手鐧

### 1. Composition API：你的程式碼，從「按類別分」到「按功能分」

這是 Vue 3 最核心的變革。Composition API 讓我們可以將同一個功能的程式碼（狀態、方法、計算屬性等）放在一起，形成一個高內聚的「邏輯單元」。

光說不練假把戲，我們直接看個例子。假設我們要實作一個簡單的滑鼠位置追蹤器：

**在 Vue 2 (Options API) 中，你可能會這樣寫：**
```javascript
// options-api-example.js
export default {
  data() {
    return {
      x: 0,
      y: 0,
    };
  },
  methods: {
    update(event) {
      this.x = event.pageX;
      this.y = event.pageY;
    },
  },
  mounted() {
    window.addEventListener('mousemove', this.update);
  },
  unmounted() {
    window.removeEventListener('mousemove', this.update);
  },
};
```
> 注意到了嗎？`x`、`y` 在 `data`，`update` 在 `methods`，生命週期鉤子又在另外兩處。這只是一個功能，程式碼就散落四方了。

**而在 Vue 3 (Composition API) 中，我們可以這樣寫：**
```javascript
// composition-api-example.js
import { ref, onMounted, onUnmounted } from 'vue';

export function useMouse() {
  const x = ref(0);
  const y = ref(0);

  function update(event) {
    x.value = event.pageX;
    y.value = event.pageY;
  }

  onMounted(() => window.addEventListener('mousemove', update));
  onUnmounted(() => window.removeEventListener('mousemove', update));

  return { x, y };
}
```
> 看到差別了嗎？所有跟「滑鼠追蹤」相關的邏輯，都被漂亮地封裝在 `useMouse` 這個函式裡。它不僅可讀性高，還能輕易地被任何組件複用！這就是「邏輯內聚」的威力。

### 2. 更快的效能：不只快，還很聰明

Vue 3 在效能上下足了功夫，這不是玄學，而是有具體技術支撐的：

-   **編譯期的優化 (Compiler-Informed)**：Vue 3 的編譯器會像個偵探一樣，在編譯時就找出模板中永遠不會改變的部分（例如純文字或靜態 class），並將它們「提升」(hoist) 出來，變成一個常數，這樣在每次重新渲染時，就完全不用理會它們。
-   **補丁標記 (Patch Flags)**：對於會變動的動態內容，編譯器會給它打上一個「補丁標記」，例如 `PatchFlags.TEXT`。這就像給 VDOM 比對開了個「快速通道」，更新時，Vue 看到這個標記，就知道「嘿，這裡只有文字會變」，於是它就能跳過所有其他檢查（如 class、style 等），直奔主題，大幅提升效率。
-   **Proxy-based Reactivity**：告別 `Object.defineProperty`，擁抱 ES6 的 `Proxy`。Vue 2 的 `Object.defineProperty` 必須在一開始就遞迴地把所有屬性都轉換一遍，如果你的物件有 1000 個屬性，它就得執行 1000 次。而 Vue 3 的 `Proxy` 是「懶漢模式」，只有當你真正用到某個屬性時，它才會進行攔截。更棒的是，`Proxy` 能直接監聽整個物件，所以你新增或刪除屬性，Vue 都能立刻知道，這在 Vue 2 是個很頭痛的問題。

### 3. 完美的 TypeScript 擁抱

因為 Vue 3 的原始碼本身就是用 TypeScript 重寫的，並且 Composition API 的設計天然就對型別友善。

舉個例子，在 Composition API 中，你從 `setup()` 或 `composable` 函式中回傳的所有東西，都有著非常清晰、可靠的型別，IDE 也能提供精準的自動補全。

相較之下，在 Options API 中，`this` 的型別常常像個謎，需要各種 `shims`（墊片）或裝飾器才能被正確推斷，開發體驗相差甚遠。可以說，Vue 3 真正讓 TypeScript 從一個「可選項」變成了一個「頭等公民」。

## 本篇自我挑戰

-   **今日挑戰**：試著回想一個你過去在 Vue 2 中寫過最複雜的組件，它的 `data`, `methods`, `computed`, `watch` 是不是散落在各處？思考一下，如果用「功能」來組織這些程式碼，會不會更清晰？
-   **反思**：Vue 3 的 Composition API 真的有解決你的痛點嗎？還是你覺得 Options API 在某些場景下，其實也沒那麼糟？

## 總結

今天我們鳥瞰了 Vue 3 這片新大陸，從效能、開發體驗到程式碼組織方式，它都帶來了巨大的進步。這不僅僅是一次版本的更新，更是一次開發思維的躍遷。

希望透過今天的介紹，你對這些 Vue 3 的核心概念有了更深的印象。

本日關鍵字回顧

- Composition API: 組織程式碼的新方式，強調「邏輯內聚」。
- Options API: Vue 2 的傳統寫法，在 Vue 3 依然支援。
- Compiler-Informed: 編譯器更智能，能在編譯期進行大量優化。
- Patch Flags & Hoisting: VDOM 優化的兩大黑魔法，讓畫面更新更快。
- Proxy-based Reactivity: Vue 3 響應式系統的基石，取代了 Object.defineProperty。

是不是有點心癢癢了？別急，明天我們就親手來玩玩 Vue 3 最核心的資料綁定與事件處理，看看它到底藏了什麼黑魔法。準備好了嗎？

