# 【Day 9】`computed` vs `watch`：Vue 響應式魔法的兩把利刃，何時出鞘？

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！前兩天我們深入了解了 `ref` 和 `reactive` 這對響應式數據的雙生子，它們是讓你的數據「活」起來的基石。今天，我們要介紹兩位更進階的響應式魔法工具：**`computed`** 和 **`watch`**！

它們就像 Vue 響應式魔法的兩把利刃，都能在數據變化時做出反應，但它們的「出招方式」和「適用場景」卻大相徑庭。如果說 `ref` 和 `reactive` 是你的「數據容器」，那麼 `computed` 和 `watch` 就是你的「數據處理器」或「數據觀察者」。搞清楚它們的脾氣，才能讓你的程式碼既優雅又高效，避免「誤用」的尷尬！

## `computed`：聰明的「計算機」，結果會自動快取

想像 `computed` 就像一個超級聰明的「計算機」或「會計師」。它會根據你給它的原始數據（響應式數據），自動幫你計算出一個新的結果。最棒的是，它非常「懶惰」且「聰明」：**只有當它所依賴的原始數據發生變化時，它才會重新計算一次**。如果原始數據沒變，它就會直接給你上次計算好的「快取」結果，效率超高！

### 概念與原理

`computed` 是一個「計算屬性」，它的值是根據其他響應式數據「計算」出來的。它本質上是一個函式，但使用起來就像一個普通的響應式屬性。

```javascript
<script setup>
import { ref, computed } from 'vue';

const price = ref(100);
const quantity = ref(2);

// computed 屬性：總價
const totalPrice = computed(() => {
  console.log('計算總價...'); // 只有在 price 或 quantity 變化時才會執行
  return price.value * quantity.value;
});

// computed 屬性：判斷是否為 VIP (假設總價超過 500 才是 VIP)
const isVIP = computed(() => {
  return totalPrice.value > 500 ? '是' : '否';
});

const incrementQuantity = () => {
  quantity.value++;
};

const changePrice = () => {
  price.value = 250;
};
</script>

<template>
  <p>單價: {{ price }}</p>
  <p>數量: {{ quantity }}</p>
  <p>總價: {{ totalPrice }}</p> <!-- 這裡會觸發 totalPrice 的計算 -->
  <p>是否為 VIP: {{ isVIP }}</p>
  <button @click="incrementQuantity">增加數量</button>
  <button @click="changePrice">改變單價</button>
</template>
```

### `computed` 的優點與適用場景

*   **快取效能**：這是 `computed` 最核心的優勢。它會記住上次的計算結果，只有當依賴的數據改變時才重新計算，避免不必要的重複運算，大大提升效能。
*   **語法簡潔**：讓模板保持乾淨。複雜的邏輯計算放在 `computed` 裡，模板只負責顯示結果。
*   **數據衍生**：適合從現有數據衍生出新數據的場景，例如：
    *   購物車總價、過濾後的列表、排序後的數據。
    *   將多個數據組合成一個新的顯示格式（如全名、格式化日期）。
    *   根據某個條件判斷一個布林值（如 `isFormValid`）。

## `watch`：忠實的「監聽者」，數據一變就行動

`watch` 就像一個忠心耿耿的「監聽者」或「偵探」。它會緊盯著你指定的響應式數據，一旦這些數據有任何風吹草動（變化），它就會立刻執行你給它的任務（一個回調函式）。`watch` 不會快取結果，它的主要目的是執行「副作用」。

### 概念與原理

`watch` 用來「監聽」一個或多個響應式數據源，並在數據源變化時執行一個「副作用」函式。這個副作用可以是發送 API 請求、操作 DOM、執行異步操作、觸發動畫等。

```javascript
<script setup>
import { ref, watch } from 'vue';

const searchTerm = ref('');
const searchResult = ref([]);

// 監聽 searchTerm 的變化
watch(searchTerm, async (newValue, oldValue) => {
  console.log(`搜索詞從 "${oldValue}" 變為 "${newValue}"`);
  if (newValue.length > 2) {
    // 模擬 API 請求
    console.log(`正在搜索 "${newValue}"...`);
    searchResult.value = await simulateApiCall(newValue);
  } else {
    searchResult.value = [];
  }
});

// 模擬一個非同步 API 請求
const simulateApiCall = (term) => {
  return new Promise(resolve => {
    setTimeout(() => {
      const results = [`Result for ${term} 1`, `Result for ${term} 2`];
      resolve(results);
    }, 500);
  });
};

// 監聽多個數據源
const firstName = ref('John');
const lastName = ref('Doe');

watch([firstName, lastName], ([newFirstName, newLastName], [oldFirstName, oldLastName]) => {
  console.log(`姓名從 ${oldFirstName} ${oldLastName} 變為 ${newFirstName} ${newLastName}`);
});

// 監聽物件的深層變化 (需要 deep: true)
const userProfile = ref({
  name: 'Alice',
  settings: {
    theme: 'dark'
  }
});

watch(userProfile, (newValue) => {
  console.log('用戶設定有變化 (深層監聽):', newValue.settings.theme);
}, { deep: true }); // 啟用深層監聽

// 立即執行一次 (immediate: true)
const count = ref(0);
watch(count, (newValue) => {
  console.log('Count 變化或初始化:', newValue);
}, { immediate: true }); // 組件掛載時會立即執行一次
</script>

<template>
  <input v-model="searchTerm" placeholder="輸入搜索詞 (至少3個字)">
  <ul>
    <li v-for="result in searchResult" :key="result">{{ result }}</li>
  </ul>

  <hr>
  <input v-model="firstName" placeholder="First Name">
  <input v-model="lastName" placeholder="Last Name">

  <hr>
  <button @click="userProfile.settings.theme = userProfile.settings.theme === 'dark' ? 'light' : 'dark'">
    切換主題 (深層監聽)
  </button>
</template>
```

### `watch` 的優點與適用場景

*   **執行副作用**：這是 `watch` 的主要目的。當數據變化時，你需要執行一些與 UI 無關，或需要非同步操作的邏輯，例如：
    *   發送 API 請求（如搜索框輸入後延遲發送請求）。
    *   操作 DOM（如滾動到特定位置）。
    *   執行異步操作（如數據驗證、動畫）。
    *   監聽路由變化、props 變化等。
*   **精確控制**：
    *   可以監聽單個或多個數據源。
    *   可以設定 `immediate: true` 讓監聽器在組件初始化時立即執行一次。
    *   可以設定 `deep: true` 深度監聽物件或陣列內部屬性的變化。

## `computed` vs `watch`：權衡與選擇，避免誤用！

這兩把利刃雖然都能響應數據變化，但它們的「目的」和「行為」截然不同。搞混了，輕則程式碼冗餘，重則導致效能問題或難以追蹤的 Bug！

### 核心差異總結

| 特性     | `computed`                               | `watch`                                  |
| :------- | :--------------------------------------- | :--------------------------------------- |
| **目的** | **計算**並**返回一個新值**（有快取）     | **監聽**數據變化，並**執行副作用**（無快取） |
| **返回值** | **必須有返回值**                         | **沒有返回值要求**                       |
| **惰性執行** | 默認**惰性執行**（只有被使用時才計算）   | 默認**非惰性**（數據變化就執行，可設 `immediate`） |
| **副作用** | **不應有副作用**（應是純粹的計算）       | **主要用於執行副作用**                   |
| **適用場景** | 數據衍生、模板複雜邏輯、過濾排序         | 異步操作、DOM 操作、監聽外部變化         |

### 誤用避免：別把「計算」當「監聽」！

*   **不要用 `watch` 來替代 `computed`**：
    *   如果你只是想根據現有響應式數據計算出一個新的響應式數據，並且這個新數據會被模板或其他 `computed` 屬性使用，那麼請堅定地選擇 `computed`！它有快取，效能更好，程式碼也更簡潔。
    *   **錯誤範例**：你可能會想用 `watch` 來監聽 `price` 和 `quantity` 的變化，然後手動更新 `totalPrice`。這不僅程式碼更長，也失去了 `computed` 的快取優勢。
*   **不要在 `computed` 中執行副作用**：
    *   `computed` 應該是「純粹」的計算，不應該有任何「副作用」（例如發送 API 請求、修改其他響應式數據、操作 DOM）。如果你這樣做了，不僅會讓程式碼難以理解和測試，還可能導致意想不到的行為。
    *   **錯誤範例**：在 `computed` 裡發送 API 請求，這會導致每次 `computed` 重新計算時都發送請求，而不是在你需要的時候。

## 監聽不同數據源：`ref`、`reactive`、`getter` 的寫法差異

`watch` 的第一個參數是你要監聽的「數據源」。根據數據源的類型，寫法會有些不同：

1.  **監聽 `ref`**：直接傳入 `ref` 變數即可。
    ```javascript
    const myRef = ref(0);
    watch(myRef, (newValue, oldValue) => {
      console.log('myRef 變了:', newValue);
    });
    ```

2.  **監聽 `reactive` 物件**：直接傳入 `reactive` 物件即可，會自動深度監聽其所有屬性。
    ```javascript
    const myReactiveObj = reactive({ a: 1, b: { c: 2 } });
    watch(myReactiveObj, (newValue, oldValue) => {
      console.log('myReactiveObj 變了:', newValue);
    });
    // 注意：如果你直接替換 myReactiveObj = { ... }，則監聽會失效！
    // 因為 reactive 監聽的是物件的引用，替換引用會導致 Proxy 失效。
    // 解決方法是使用 Object.assign(myReactiveObj, { ... }) 或修改其內部屬性。
    ```

3.  **監聽 `reactive` 物件的特定屬性**：需要使用一個 getter 函式來返回該屬性。
    ```javascript
    const myReactiveObj = reactive({ a: 1, b: { c: 2 } });
    watch(() => myReactiveObj.a, (newValue, oldValue) => {
      console.log('myReactiveObj.a 變了:', newValue);
    });
    // 如果要監聽物件屬性的深層變化，且該屬性本身是物件，則需要加上 { deep: true }
    watch(() => myReactiveObj.b, (newValue, oldValue) => {
      console.log('myReactiveObj.b 變了:', newValue);
    }, { deep: true });
    ```

4.  **監聽多個數據源**：傳入一個陣列，陣列中可以是 `ref` 或 getter 函式。
    ```javascript
    const ref1 = ref(0);
    const reactiveObj = reactive({ prop: 'hello' });
    watch([ref1, () => reactiveObj.prop], ([newRef1, newProp], [oldRef1, oldProp]) => {
      console.log('多個數據源變了:', newRef1, newProp);
    });
    ```

## 啟動時機與生命週期：誰先誰後？

`computed`、`watch` 和 `watchEffect` 在組件初始化時的觸發時機有所不同，這與組件的生命週期息息相關。

*   **`computed`**：
    *   **惰性執行**：`computed` 默認是「懶惰」的。它只會在它的值**第一次被訪問**時（例如在模板中顯示，或被另一個 `computed`/`watch` 依賴時）才會執行計算。在此之前，它不會執行。
    *   **與生命週期關係**：它不直接綁定到某個生命週期鉤子，但它的首次計算通常發生在組件掛載過程中，當模板被渲染並需要其值時。

*   **`watch`**：
    *   **默認非立即執行**：`watch` 默認不會在組件初始化時立即執行。它只會在**監聽的數據源發生變化時**才觸發回調函式。
    *   **`immediate: true`**：如果你希望 `watch` 在組件初始化時就立即執行一次，可以設定 `immediate: true` 選項。此時，它的回調函式會在組件的 `setup()` 函式執行完畢後，但在 `onMounted` 鉤子之前觸發。
    *   **與生命週期關係**：
        *   **無 `immediate`**：首次觸發在 `onMounted` 之後（當數據源首次變化時）。
        *   **有 `immediate`**：首次觸發在 `setup()` 執行後，`onMounted` 之前。

*   **`watchEffect`**：
    *   **立即執行**：`watchEffect` 總是會在組件初始化時**立即執行一次**。它會自動追蹤回調函式中使用的所有響應式依賴，並在這些依賴變化時重新執行。
    *   **與生命週期關係**：首次觸發在 `setup()` 執行後，`onMounted` 之前。它非常適合那些需要在組件掛載前就執行一次，並且後續依賴變化時也需要執行的副作用。

**簡單來說：**
*   **`computed`**：你問它才算，而且算過會記住。
*   **`watch`**：你說要監聽誰，誰變了它才動，除非你特別交代它「一開始就動 (`immediate: true`)」。
*   **`watchEffect`**：它會自己看你用了誰，然後一開始就動，之後誰變了它也動。

## 小技巧與注意事項：

*   **`watch` 沒觸發？可能是監聽對象的問題！**
    *   **監聽 `reactive` 物件的替換**：如果你直接替換掉 `reactive` 物件的引用（例如 `myReactiveObj = {}`），而不是修改其內部屬性，那麼直接監聽 `myReactiveObj` 的 `watch` 會失效。因為 `watch` 監聽的是該物件的引用，引用變了，它就跟丟了。解決方法是修改其內部屬性，或使用 `Object.assign()` 來更新。
    *   **監聽 `reactive` 物件的原始型別屬性**：如果你想監聽 `reactive` 物件的某個原始型別屬性（例如 `myReactiveObj.count`），直接 `watch(myReactiveObj.count, ...)` 是無效的。你需要使用一個 getter 函式：`watch(() => myReactiveObj.count, ...)`。這是因為 `watch` 需要一個響應式引用作為源，而 `myReactiveObj.count` 是一個原始型別的值，不是響應式引用。

*   **`watch` 觸發了，但 DOM 還沒更新？請找 `nextTick`！**
    *   `watch` 的回調函式通常在數據變化後立即執行，但此時 Vue 可能還沒有來得及更新 DOM。如果你在 `watch` 回調中需要訪問或操作**更新後的 DOM**，你必須將這些操作放在 `nextTick` 回調中。這就像是：「等 Vue 把畫面畫好了，你再動手！」
    ```javascript
    watch(someData, (newValue) => {
      // 數據已經變了，但 DOM 可能還沒更新
      nextTick(() => {
        // 在這裡訪問的 DOM 才是更新後的狀態
        console.log('DOM 已更新，可以操作了', document.getElementById('my-element').textContent);
      });
    });
    ```

*   **`watchEffect`：更簡潔的副作用監聽**
    *   除了 `watch`，Vue 還提供了 `watchEffect`。它會自動追蹤回調函式中使用的所有響應式依賴，並在這些依賴變化時重新執行。適合那些不需要明確指定監聽源，且副作用邏輯相對簡單的場景。
    *   **`watch` vs `watchEffect`**：
        *   `watch`：明確指定監聽源，可以訪問 `newValue` 和 `oldValue`，可以延遲執行。
        *   `watchEffect`：自動追蹤依賴，無法訪問 `oldValue`，立即執行。
*   **清除副作用**：在 `watch` 中執行異步操作時，要特別注意「競態條件」（Race Condition）。例如，用戶快速輸入搜索詞，導致多個 API 請求同時發出，但返回順序不確定。這時候，你需要在新的請求發出前，取消或忽略舊的請求。這通常通過返回一個清理函式來實現。

## 本篇自我挑戰

-   **今日挑戰**：設計一個搜索功能：一個輸入框，下方即時顯示搜索結果。
    *   嘗試用 `computed` 來實現一個「過濾列表」的功能（假設數據已經在前端）。
    *   再嘗試用 `watch` 來實現一個「延遲搜索」的功能（模擬發送 API 請求）。
    *   比較這兩種方式在程式碼結構和行為上的差異。
-   **反思**：當你需要根據用戶輸入，從後端獲取數據並更新 UI 時，你會選擇 `computed` 還是 `watch`？為什麼？

## 總結

今天我們深入探討了 Vue 響應式魔法的兩把利刃：`computed` 和 `watch`。`computed` 是一個聰明的「計算機」，擅長從現有數據衍生新數據並自動快取；而 `watch` 則是一個忠實的「監聽者」，專注於在數據變化時執行各種副作用。

理解它們的核心差異，並在正確的場景選擇正確的工具，是寫出高效、可維護 Vue 應用程式的關鍵。避免將它們混淆使用，才能讓你的程式碼更加清晰、健壯。

本日關鍵字回顧

-   **`computed`**: 計算屬性，有快取，用於數據衍生，不應有副作用。
-   **`watch`**: 監聽器，無快取，用於執行副作用（如 API 請求、DOM 操作）。
-   **快取 (Caching)**: `computed` 的效能優勢。
-   **副作用 (Side Effect)**: `watch` 的主要應用場景。
-   **惰性執行 (Lazy Evaluation)**: `computed` 的特性。
-   **`watchEffect`**: 自動追蹤依賴的監聽器。
-   **競態條件 (Race Condition)**: `watch` 異步操作中需注意的問題。
-   **`nextTick`**: 確保在 DOM 更新週期後執行操作。

明天，我們將把這些響應式魔法工具打包，學習如何打造自己的 `Composable` (自訂 Hook)，讓你的程式碼複用性更上一層樓！