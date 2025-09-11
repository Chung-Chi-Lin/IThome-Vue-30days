# 【Day 23】元件的X光片：深入測試 Props、事件與 Slot

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天，我們學會了如何使用 `mount` 將元件渲染出來，並用 `expect(wrapper.text()).toContain(...)` 做了最基本的斷言。這就像我們在汽車工廠裡，確認了一台車「看起來像一台車」。

但這樣還不夠。一台車不只要有外觀，它的油門、煞車、方向盤都必須正常運作。同樣地，一個好的 Vue 元件，光是能渲染出來是不夠的，我們必須確保它的「**公開 API**」——也就是 **Props、事件 (Emits) 和插槽 (Slots)** ——都能正常工作。

今天，我們就要扮演「**品管工程師**」的角色，拿起測試的「**X 光機**」，深入掃描我們在 Day 21 建立的 `BaseButton` 元件，確保它的每一個接口、每一個互動細節都堅固可靠。

把元件的 API 想成是樂高積木的「卡榫」和「插槽」：

-   **Props**：是積木上的「插孔」，決定了這塊積木的特性（顏色、大小）。
-   **Emits**：是積木成功接合時發出的「咔噠」聲，通知其他部分「我被觸發了」。
-   **Slots**：是積木上預留的「空間」，讓你可以嵌入其他小積木，增加客製化彈性。

我們的任務，就是確保這些卡榫尺寸精準、聲音清脆、空間恰到好處。

## 測試對象：`SimpleButton.vue`

為了聚焦，我們使用一個稍微簡化版的按鈕元件作為今天的測試主角。

```vue
// src/components/SimpleButton.vue
<template>
  <button
    class="simple-button"
    :class="{ 'is-disabled': disabled }"
    :disabled="disabled"
    @click="handleClick"
  >
    <slot>Default Text</slot> <!-- 提供了預設內容的插槽 -->
  </button>
</template>

<script setup>
const props = defineProps({ disabled: Boolean });
const emit = defineEmits(['click']);

function handleClick(event) {
  emit('click', event);
}
</script>
```

## 1. 測試 Props：元件的遙控器

Props 是父層用來控制子層行為的「遙控器」。測試 Props 的核心就是：**按下遙控器上的按鈕 (傳入 Props)，然後檢查元件的反應是否正確。**

**測試情境**：當 `disabled` prop 為 `true` 時，按鈕是否真的被禁用了？

```javascript
// src/components/SimpleButton.spec.js
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import SimpleButton from './SimpleButton.vue';

describe('SimpleButton.vue', () => {
  it('renders correctly when not disabled', () => {
    const wrapper = mount(SimpleButton);
    // attributes(): 取得 DOM 元素的屬性
    expect(wrapper.attributes('disabled')).toBeUndefined();
    // classes(): 取得 DOM 元素的 class 列表
    expect(wrapper.classes('is-disabled')).toBe(false);
  });

  it('is disabled when the disabled prop is true', () => {
    const wrapper = mount(SimpleButton, {
      props: {
        disabled: true,
      },
    });

    // 斷言 <button> 元素確實有 disabled 屬性
    expect(wrapper.attributes('disabled')).toBeDefined();
    // 斷言 <button> 元素有 is-disabled 這個 class
    expect(wrapper.classes('is-disabled')).toBe(true);
  });
});
```

> **測試工具箱**：
> - `wrapper.attributes()`: 讓你檢查 HTML 元素的屬性，例如 `disabled`, `href`, `type`。
> - `wrapper.classes()`: 讓你檢查元素擁有哪些 CSS class。
> - `toBeDefined()` / `toBeUndefined()`: 非常適合用來檢查一個屬性是否存在。

## 2. 測試 Emits：元件的對外通訊

Emits 是子層向父層「回報工作」的機制。測試 Emits 的核心是：**模擬一個使用者操作，然後竊聽元件是否發出了正確的「訊號」(事件)。**

**測試情境**：點擊按鈕時，是否觸發了 `click` 事件？如果按鈕被禁用，是否就不會觸發？

```javascript
// ...接續上面的 describe 區塊

it('emits a click event when clicked', async () => {
  const wrapper = mount(SimpleButton);

  // trigger(): 模擬觸發一個 DOM 事件
  await wrapper.trigger('click');

  // emitted(): 取得這個元件實例發出過的所有事件
  // toHaveProperty(): 斷言物件是否擁有某個屬性
  expect(wrapper.emitted()).toHaveProperty('click');

  // 也可以斷言事件被觸發了幾次
  expect(wrapper.emitted().click).toHaveLength(1);
});

it('does not emit a click event when disabled', async () => {
  const wrapper = mount(SimpleButton, {
    props: { disabled: true },
  });

  await wrapper.trigger('click');

  // not: Vitest/Jest 的反向斷言
  expect(wrapper.emitted()).not.toHaveProperty('click');
});
```

> **非同步注意**：`wrapper.trigger()` 會觸發 DOM 更新，而 Vue 的 DOM 更新是**非同步**的。因此，觸發事件的這一步最好都加上 `async / await`，確保我們在斷言時，元件已經更新完畢。

## 3. 測試 Slots：元件的客製化空間

Slots 是元件的「開放世界」，讓使用者可以自由填入內容。測試 Slots 的核心是：**在掛載時，從「插槽」塞入一些內容，然後檢查這些內容是否被正確地渲染出來。**

**測試情境**：插槽的預設內容是否正常？傳入新的內容後，是否會取代預設值？

```javascript
// ...接續上面的 describe 區塊

it('renders default slot content when nothing is passed', () => {
  const wrapper = mount(SimpleButton);
  expect(wrapper.text()).toBe('Default Text');
});

it('renders content passed to the default slot', () => {
  const slotContent = 'Click Me Now!';
  const wrapper = mount(SimpleButton, {
    slots: {
      // slots 物件的 key 對應到插槽的 name
      // `default` 是預設插槽的固定名稱
      default: slotContent,
    },
  });

  expect(wrapper.text()).toBe(slotContent);
});

// 假設我們還有一個具名插槽 <slot name="icon" />
it('renders named slots content', () => {
  const wrapper = mount(SimpleButton, {
    slots: {
      default: 'Submit',
      icon: '<span class="icon">🚀</span>' // 可以是純文字、HTML 字串，甚至是另一個元件
    }
  });

  // find(): 在元件中尋找符合 CSS 選擇器的第一個元素
  const icon = wrapper.find('.icon');
  // exists(): 檢查找到的元素是否存在
  expect(icon.exists()).toBe(true);
  expect(icon.text()).toBe('🚀');
});
```

> **測試工具箱**：
> - `mount({ slots: { ... } })`: 在掛載時傳入插槽內容。
> - `wrapper.find()`: 你的「元素探測器」，用 CSS 選擇器在元件內部尋找目標。
> - `wrapper.exists()`: 確認探測器是否真的找到了東西。

## 本篇自我挑戰

1.  **測試 Props 樣式**：回到 Day 21 的 `BaseButton.vue`，它有一個 `color` prop。試著寫一個測試：當 `color` prop 為 `primary` 時，斷言按鈕是否擁有 `base-button--primary` 這個 class？(提示：使用 `wrapper.classes()`)。
2.  **測試 Emit Payload**：修改你的按鈕，讓它在 click 時 `emit('click', 'some-payload')`。然後寫一個測試，斷言 `wrapper.emitted().click[0]` 是否等於 `['some-payload']`。
3.  **測試具名插槽**：為你的 `BaseButton.vue` 加上一個名為 `icon-left` 的插槽，並撰寫測試來驗證傳入的圖示是否被正確渲染。

## 總結

今天我們像品管工程師一樣，用 X 光徹底檢查了元件的外部接口。現在，你已經掌握了測試一個獨立 Vue 元件所需的核心技巧：

-   **測試 Props**：在 `mount` 時傳入 `props`，然後檢查元件的 **屬性 (`attributes`)**、**樣式 (`classes`)** 或 **文字 (`text`)**。
-   **測試 Emits**：用 `wrapper.trigger()` 模擬操作，然後用 `wrapper.emitted()` 檢查事件是否如期發出。
-   **測試 Slots**：在 `mount` 時傳入 `slots`，然後用 `wrapper.text()` 或 `wrapper.find()` 檢查內容是否被正確渲染。

掌握了這些，你就等於掌握了驗證元件「公開合約」的能力。這能確保你的元件不僅現在能用，未來在別人手中、在不同的場景下，依然能穩定地運作。

明天，我們將更進一步，學習如何模擬真實的使用者行為，讓我們的測試不只關心「元件的 API」，更關心「使用者的體驗」。

## 本日關鍵字回顧

-   `元件 API (Component API)`
-   `mount options: props`
-   `mount options: slots`
-   `wrapper.attributes()`
-   `wrapper.classes()`
-   `wrapper.trigger()`
-   `wrapper.emitted()`
-   `wrapper.find()`
-   `wrapper.exists()`
-   `非同步 DOM 更新 (Async DOM Updates)`
