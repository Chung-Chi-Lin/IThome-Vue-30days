# 【Day 8】`ref` vs `reactive`：Vue 響應式數據的雙生子，該選誰？

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！昨天我們正式踏入了 Composition API 的世界，感受到了 `<script setup>` 帶來的清爽與便利。但要讓你的程式碼真正「活」起來，光有 `<script setup>` 還不夠，你還需要兩位超級英雄來幫你管理數據的「響應性」：那就是 **`ref`** 和 **`reactive`**！

它們就像 Vue 響應式數據的雙生子，雖然都能讓你的數據在變化時自動更新畫面，但它們的脾氣、能力和適用場景卻大不相同。今天，我們就來深入了解這對兄弟，搞清楚它們的核心差異，以及在什麼時候該請誰出場，才能讓你的程式碼既優雅又高效！

## `ref`：單身貴族，什麼都能裝，但有點小脾氣

想像 `ref` 就像一個萬能的「魔法盒子」，無論你把什麼東西丟進去——數字、字串、布林值，甚至是物件或陣列——它都能把它變成響應式的。但這個盒子有個小脾氣：當你在 `<script>` 裡面想打開它、取出或放入東西時，你必須禮貌地敲敲門，也就是加上 `.value`。

### 概念與原理

`ref` 用來包裝任何型別的數據，使其變成響應式。它會返回一個 `Ref` 物件，這個物件只有一個屬性，就是 `value`。當你修改 `value` 的值時，Vue 就能偵測到變化。

```javascript
<script setup>
import { ref } from 'vue';

// 包裝基本型別
const count = ref(0); // 數字
const message = ref('Hello Vue!'); // 字串
const isActive = ref(true); // 布林值

// 包裝物件型別 (雖然可以，但通常有更好的選擇)
const user = ref({
  name: 'Alice',
  age: 30
});

const increment = () => {
  count.value++; // 在 <script> 中訪問或修改 ref，必須用 .value
};

const changeName = () => {
  user.value.name = 'Bob'; // 即使是物件，也還是要先 .value
};
</script>

<template>
  <p>Count: {{ count }}</p> <!-- 在模板中，ref 會自動解包，不用 .value -->
  <p>Message: {{ message }}</p>
  <p>Is Active: {{ isActive }}</p>
  <p>User Name: {{ user.name }}</p>
  <button @click="increment">Increment Count</button>
  <button @click="changeName">Change User Name</button>
</template>
```

### `ref` 的優點

*   **萬能包裝**：可以包裝任何型別的數據，非常靈活。
*   **模板自動解包**：在 `<template>` 中使用 `ref` 時，Vue 會自動幫你把 `.value` 拿掉，讓模板看起來更簡潔，就像普通變數一樣。
*   **明確提示**：在 `<script>` 中必須使用 `.value`，這其實是一個很好的提示，讓你清楚知道你正在操作的是一個響應式數據，而不是普通的 JavaScript 變數。

## `reactive`：物件專屬，深層響應的「魔法容器」

`reactive` 就像一個專門為「物件」量身打造的「魔法容器」。你只能把物件（包括陣列）放進去，一旦放進去，這個物件的所有屬性（甚至是深層嵌套的屬性）都會變成響應式的。而且，當你在 `<script>` 裡面操作它時，你不需要像 `ref` 那樣加上 `.value`，直接操作物件屬性就行了，非常直覺。

### 概念與原理

`reactive` 用來將一個 JavaScript 物件（或陣列）轉換成響應式物件。它內部是通過 ES6 的 `Proxy` 來實現的，這讓它能夠監聽物件內部屬性的增刪改查，實現「深層響應」。

```javascript
<script setup>
import { reactive } from 'vue';

const state = reactive({
  count: 0,
  user: {
    name: 'Alice',
    age: 30,
    address: {
      city: 'Taipei'
    }
  },
  items: ['apple', 'banana']
});

const increment = () => {
  state.count++; // 直接操作屬性，不需要 .value
};

const changeCity = () => {
  state.user.address.city = 'Kaohsiung'; // 深層屬性也能響應
};

const addItem = () => {
  state.items.push('orange'); // 陣列操作也能響應
};
</script>

<template>
  <p>Count: {{ state.count }}</p>
  <p>User City: {{ state.user.address.city }}</p>
  <ul>
    <li v-for="item in state.items" :key="item">{{ item }}</li>
  </ul>
  <button @click="increment">Increment Count</button>
  <button @click="changeCity">Change City</button>
  <button @click="addItem">Add Item</button>
</template>
```

### `reactive` 的優點

*   **直覺操作**：在 `<script>` 中直接操作物件屬性，無需 `.value`，寫法更自然。
*   **深層響應**：物件內部無論嵌套多深，其屬性的變化都能被 Vue 偵測到，非常方便。

### `reactive` 的陷阱：小心「解構」與「替換」！

`reactive` 雖然好用，但它有兩個常見的「地雷」需要特別注意：

1.  **解構失去響應性**：當你解構 `reactive` 物件的屬性時，這些被解構出來的變數會失去響應性。因為它們已經脫離了 `Proxy` 的監聽範圍。
    ```javascript
    const state = reactive({ count: 0 });
    let { count } = state; // count 變成普通變數，不再響應式
    count++; // state.count 不會變
    ```
2.  **替換整個物件失去響應性**：如果你直接替換掉 `reactive` 物件的引用，而不是修改其內部屬性，那麼新的物件也會失去響應性。
    ```javascript
    let state = reactive({ count: 0 });
    state = { count: 1 }; // state 變成普通物件，不再響應式
    ```
    **正確的做法**：應該使用 `Object.assign()` 或展開運算符來更新 `reactive` 物件的屬性，而不是替換整個物件。
    ```javascript
    Object.assign(state, { count: 1 }); // 正確
    // 或者
    // state.count = 1; // 更簡單直接
    ```

## `ref` vs `reactive`：到底選誰？Vue 數據管理的最佳策略

這是一個 Vue 開發者經常會遇到的選擇題。其實沒有絕對的「最好」，只有「最適合」的場景。以下是一些選擇建議：

### 核心差異總結

| 特性     | `ref`                               | `reactive`                               |
| :------- | :---------------------------------- | :--------------------------------------- |
| **包裝對象** | 任何型別（基本型別、物件、陣列）    | 僅限物件型別（物件、陣列、Map、Set）     |
| **訪問方式** | 在 `<script>` 中需 `.value`，模板自動解包 | 直接訪問屬性，無需 `.value`              |
| **響應深度** | 淺層響應（只監聽 `value` 的變化） | 深層響應（監聽物件內部所有屬性的變化） |
| **解構問題** | 無                                  | 有（解構會失去響應性）                   |
| **替換問題** | 無                                  | 有（替換整個物件會失去響應性）           |

### 選擇建議：我的「懶人包」策略

1.  **優先使用 `ref`**：
    *   當你處理的是 **單一值**（數字、字串、布林值）時，`ref` 是不二之選。
    *   當你處理的是 **物件或陣列**，但你預期會頻繁地 **替換整個物件/陣列** 時，`ref` 更安全，因為它沒有 `reactive` 的替換陷阱。
    *   當你希望在 `<script>` 中明確區分響應式數據和普通數據時，`.value` 是一個很好的視覺提示。

2.  **當數據是複雜物件且需要深層響應時，考慮 `reactive`**：
    *   如果你有一個複雜的數據結構，例如表單數據、用戶配置等，並且你需要對其內部多個屬性進行操作，`reactive` 可以提供更直覺的寫法（無需 `.value`）。
    *   **但請務必搭配 `toRefs` 使用**，以解決解構失去響應性的問題。這就像給 `reactive` 穿上了一層「防彈衣」，讓你可以安全地解構其屬性。

### `toRefs` 的救贖：讓 `reactive` 也能優雅解構

`toRefs` 是一個非常有用的工具函式，它可以將 `reactive` 物件的所有屬性轉換為 `ref` 物件。這樣，當你解構這些 `ref` 時，它們仍然保持響應性！

```javascript
<script setup>
import { reactive, toRefs } from 'vue';

const state = reactive({
  count: 0,
  name: 'Vue'
});

// 使用 toRefs 將 state 的屬性轉換為 ref
const { count, name } = toRefs(state);

const increment = () => {
  count.value++; // 現在 count 是一個 ref，需要 .value
};

const changeName = () => {
  name.value = 'Vue 3'; // name 也是一個 ref
};
</script>

<template>
  <p>Count: {{ count }}</p>
  <p>Name: {{ name }}</p>
  <button @click="increment">Increment</button>
  <button @click="changeName">Change Name</button>
</template>
```
> 透過 `toRefs`，你既能享受到 `reactive` 的深層響應，又能避免解構失去響應性的問題，簡直是兩全其美！

## 小技巧與注意事項：

*   **判斷響應式數據**：Vue 提供了一些工具函式來判斷一個變數是否是響應式數據：`isRef`、`isReactive`、`isProxy`。在除錯時非常有用。
*   **一致性**：在團隊協作中，建議統一 `ref` 和 `reactive` 的使用規範，避免程式碼風格混亂。

## 本篇自我挑戰

-   **今日挑戰**：嘗試用 `ref` 和 `reactive` 分別實現一個簡單的表單（例如包含姓名、年齡的表單），觀察它們在數據綁定和更新時的行為差異。特別注意 `reactive` 在解構時的響應性問題。
-   **反思**：在什麼情況下，你會堅定地選擇 `ref`，又在什麼情況下，你會考慮 `reactive` 並搭配 `toRefs` 使用？

## 總結

今天我們深入探討了 Vue 響應式數據的兩位核心成員：`ref` 和 `reactive`。它們各有優缺，適用於不同的場景。`ref` 像個萬能的「魔法盒子」，靈活且安全；`reactive` 則是物件專屬的「魔法容器」，提供深層響應，但需要小心解構和替換的陷阱。

掌握它們的核心差異，並學會搭配 `toRefs` 使用，你就能在 Vue 3 的世界裡，更自如地管理你的數據，寫出更健壯、更優雅的應用程式！

本日關鍵字回顧

-   **`ref`**: 包裝任何型別數據，需 `.value` 訪問，模板自動解包。
-   **`reactive`**: 包裝物件型別數據，深層響應，解構易失響應性。
-   **`.value`**: 訪問 `ref` 包裝數據的屬性。
-   **`Proxy`**: `reactive` 內部實現深層響應的機制。
-   **`toRefs`**: 將 `reactive` 物件的屬性轉換為 `ref`，解決解構問題。
-   **響應式**: 數據變化時自動更新畫面。

明天，我們將繼續探索 Vue 的響應式魔法，深入了解 `computed` 和 `watch` 這兩個強大的工具，看看它們如何在數據變化時，為我們提供更精細的控制！
