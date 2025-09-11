# 【Day 21】打造團隊的樂高：建立共用 UI 元件庫的策略

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天，我們學會了如何用 SCSS 變數和 Mixin 建立一個樣式軍火庫。這讓我們的樣式變得井然有序，但戰爭還沒結束。

在一個團隊裡，你可能會發現一個有趣的現象：

-   工程師 A 做了一個「確認」按鈕。
-   工程師 B 在另一個頁面，也做了一個「確認」按鈕，但 padding 小了 2px。
-   工程師 C 為了彈出視窗，又做了一個「確認」按鈕，但圓角和陰影跟前兩個都不一樣。

結果，你的專案裡有三種長得像、但又不完全一樣的按鈕。這就像一個樂高套組，裡面每塊紅色的 2x4 積木都有細微的尺寸差異，根本組不起來！使用者體驗破碎，程式碼也充滿了重複與冗餘。

為了解決這個問題，我們需要建立一個「**官方樂高工廠**」——也就是**共用 UI 元件庫 (Shared UI Component Library)**。這個工廠確保每一塊「積木」（元件）都符合標準、品質一致、隨插即用。

今天，我們就來聊聊如何規劃與建立這樣一個元件庫，讓你的團隊開發效率和專案品質都提升一個檔次。

## 1. 為什麼我們需要 UI 元件庫？

在動手之前，先想清楚「為什麼」。建立元件庫不是為了跟風，而是為了解決實際問題：

1.  **一致性 (Consistency)**：確保整個應用程式的視覺與互動體驗統一，就像出自一人之手。
2.  **效率 (Efficiency)**：不要重複造輪子。開發者可以直接從「元件庫」這個工具箱裡拿出需要的工具，專注於業務邏輯。
3.  **可維護性 (Maintainability)**：當需要修改按鈕樣式或修復 Bug 時，只需修改共用元件這一個地方，所有用到它的地方都會自動更新。
4.  **協作 (Collaboration)**：元件庫是設計師與工程師之間的「共同語言」。它將抽象的設計稿，轉化為具體的、可互動的程式碼。

## 2. 思考的框架：原子化設計 (Atomic Design)

當我們開始思考如何拆分元件時，很容易陷入困境。「這個算一個元件嗎？」「這個卡片應該拆分成幾個部分？」

這時，**原子化設計 (Atomic Design)** 這個概念可以給我們一個清晰的思考框架。它將 UI 介面由小到大，分為五個層級：

1.  **原子 (Atoms)**：最基礎、不可再分的 HTML 元素。例如：標籤、輸入框、**按鈕**。它們是構成你介面的基本粒子。
    -   *對應元件：`BaseButton.vue`, `BaseInput.vue`, `BaseIcon.vue`*

2.  **分子 (Molecules)**：由多個「原子」組成的簡單功能區塊。例如：一個搜尋框（包含一個輸入框原子和一個按鈕原子）。
    -   *對應元件：`SearchForm.vue`*

3.  **組織 (Organisms)**：由「分子」和「原子」組成的更複雜、獨立的介面區塊。例如：網站的導覽列（包含 Logo、導覽連結、搜尋框分子）。
    -   *對應元件：`TheHeader.vue`*

4.  **模板 (Templates)**：頁面的線框稿，專注於內容的佈局結構，將「組織」等元件組合起來，但沒有實際內容。

5.  **頁面 (Pages)**：模板的具體實例，填入真實的內容，是使用者最終看到的樣子。

> **實戰心法**：我們不需要嚴格遵守這五個層級，但「由小到大、組合複用」的核心思想，是建立元件庫的關鍵。通常，我們會把心力集中在建立穩固的「**原子**」和「**分子**」上。

## 3. 設計一個好的基礎元件：以 `BaseButton` 為例

一個好的共用元件，應該是「**低耦合、高內聚**」的。它應該像一個黑盒子，內部邏輯封裝得很好，同時又提供足夠的彈性讓外部使用。

讓我們以最常見的 `BaseButton.vue` 為例，看看它的 API 該如何設計：

-   **Props for Variation (用 Props 決定外觀與狀態)**：一個按鈕可能有多種樣式（主色、次色）、尺寸（大、中、小）、狀態（禁用、載入中）。這些都應該透過 `props` 來控制。

-   **Slots for Flexibility (用 Slot 決定內容)**：按鈕中間的文字或圖示是多變的，我們不應該寫死。使用 `slot` 讓父層決定要放什麼內容進來，是最彈性的做法。

-   **Emits for Interaction (用 Emit 進行溝通)**：當元件內部發生事件時（例如被點擊），它應該透過 `emits` 通知父層。

**實戰演練：`BaseButton.vue`**

```vue
// src/components/base/BaseButton.vue

<template>
  <button
    class="base-button"
    :class="buttonClasses"
    :disabled="disabled || loading"
    @click="handleClick"
  >
    <span v-if="loading" class="loading-spinner"></span>
    <slot v-else /> <!-- 預設插槽，用來放按鈕文字 -->
  </button>
</template>

<script setup>
import { computed } from 'vue';

// 1. Props: 定義元件的 API
const props = defineProps({
  color: {
    type: String,
    default: 'primary', // primary | secondary
  },
  size: {
    type: String,
    default: 'medium', // small | medium | large
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  loading: {
    type: Boolean,
    default: false,
  },
});

// 2. Emits: 宣告這個元件會觸發哪些事件
const emit = defineEmits(['click']);

// 使用 computed 來根據 props 動態產生 class
const buttonClasses = computed(() => ({
  [`base-button--${props.color}`]: true,
  [`base-button--${props.size}`]: true,
}));

// 將原生 click 事件包裝一層，未來可能加入其他邏輯 (如 debounce)
function handleClick(event) {
  if (!props.disabled && !props.loading) {
    emit('click', event);
  }
}
</script>

<style lang="scss" scoped>
// 這裡可以使用我們 Day 20 建立的全域變數與 Mixin！
.base-button {
  @include basic-button; // 引用基礎按鈕樣式

  &--primary {
    background-color: $primary-color;
    color: white;
  }

  &--secondary {
    background-color: $secondary-color;
    color: white;
  }

  // ... 其他尺寸與狀態的樣式
}
</style>
```

## 4. 元件庫的組織與使用

1.  **檔案結構**：建議在 `src` 下建立一個 `components` 資料夾，並在其中區分 `base` (原子)、`modules` (分子/組織) 等。

    ```
    src/
    └── components/
        ├── base/         # 原子元件
        │   ├── BaseButton.vue
        │   └── BaseInput.vue
        └── modules/      # 業務相關的組合元件
            └── TheHeader.vue
    ```

2.  **全域註冊 vs. 手動導入**：
    -   **全域註冊**：對於像按鈕、輸入框這種極度常用的「原子」元件，可以在 `main.js` 中進行全域註冊，省去每次都要 `import` 的麻煩。但這會稍微增加初始載入的體積。
    -   **手動導入**：對於不那麼常用的元件，建議在需要的地方手動 `import`，這對 Code Splitting 更友善。

3.  **文件與預覽**：當元件庫變大時，一份好的文件是必不可少的。**Storybook** 或 **Histoire** 這類工具可以為你的元件建立一個獨立的預覽環境，讓團隊成員能清楚地看到每個元件的用法和外觀。雖然這超出我們 30 天的範圍，但絕對是你未來可以深入的方向。

## 本篇自我挑戰

1.  **設計思考**：看看你目前專案的介面，試著用「原子設計」的眼光去拆解它。哪些是原子？哪些是分子？
2.  **動手實作**：在你的專案中建立 `src/components/base` 資料夾，並試著打造你自己的 `BaseButton.vue` 或 `BaseCard.vue` 元件。思考它需要哪些 `props` 和 `slots` 才能滿足你的需求？

## 總結

今天，我們從「為什麼」到「怎麼做」，探討了建立一個共用 UI 元件庫的完整策略。這不只是在寫 Vue 元件，更是在進行「**系統化設計**」。

-   從 **原子設計** 的思想出發，規劃元件的層級與顆粒度。
-   精心設計每個基礎元件的 **API (Props, Slots, Emits)**，讓它既穩固又彈性。
-   建立清晰的 **檔案結構** 與 **使用規範** (全域 vs. 手動)。

一個好的元件庫，是團隊開發的加速器，也是產品質感的基石。當你下次再需要一個按鈕時，你不再需要從零開始，而是可以自信地從你們團隊的「樂高盒」裡，拿出那塊完美、標準的積木。

明天，我們將進入一個全新的領域：**自動化測試**。我們將學習如何使用 Vitest 為我們的 Vue 元件撰寫第一支單元測試，確保我們的「樂高積木」不僅長得好看，而且堅固耐用！

## 本日關鍵字回顧

-   `UI 元件庫 (UI Component Library)`
-   `原子化設計 (Atomic Design)`
-   `原子 (Atoms)`
-   `分子 (Molecules)`
-   `組織 (Organisms)`
-   `元件 API (Component API)`
-   `Props / Slots / Emits`
-   `全域註C冊 (Global Registration)`
-   `Storybook / Histoire`
