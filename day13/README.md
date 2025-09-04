# 【Day 13】API 串接與狀態管理：優雅地處理數據的「來去」

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 的魔法師們！前幾天我們學習了如何管理組件內部的數據，如何讓數據響應式，如何管理全局狀態。但現實世界中的應用程式，數據往往不是憑空而來，而是需要從遠方的「數據倉庫」（後端 API）中獲取。今天，我們就要學習如何讓你的 Vue 應用程式與後端進行「對話」，優雅地進行 **API 串接**，並將獲取的數據與狀態管理結合起來，處理數據的「來去」！

想像你的應用程式是一個餐廳，後端 API 就是食材供應商。你需要向供應商下訂單（發送請求），等待食材送達（接收響應），期間可能會有等待（Loading 狀態），也可能遇到食材短缺或送錯貨（Error 狀態）。如何高效、穩定地處理這些環節，是打造一個健壯應用程式的關鍵。

## 為什麼需要 API 串接？

現代前端應用程式大多採用前後端分離的架構。前端負責 UI 呈現和使用者互動，後端負責數據儲存和業務邏輯。兩者之間通過 API (Application Programming Interface) 進行數據交換。

*   **獲取數據**：從伺服器獲取用戶資料、商品列表、文章內容等。
*   **提交數據**：將用戶輸入的表單數據、訂單資訊等發送到伺服器。
*   **更新數據**：修改伺服器上的現有數據。
*   **刪除數據**：從伺服器刪除數據。

## 常見的 API 串接方式：`fetch` vs `Axios`

在 JavaScript 中，有兩種主流的方式來發送 HTTP 請求：

### 1. 原生 `fetch` API

`fetch` 是瀏覽器內建的 API，它返回一個 Promise，使用起來相對簡潔。

```javascript
// 使用 fetch 獲取數據
const fetchDataWithFetch = async () => {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/todos/1');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    console.log('Fetch Data:', data);
  } catch (error) {
    console.error('Fetch Error:', error);
  }
};

// 使用 fetch 提交數據 (POST)
const postDataWithFetch = async () => {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        title: 'foo',
        body: 'bar',
        userId: 1,
      }),
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    console.log('Fetch POST Data:', data);
  } catch (error) {
    console.error('Fetch POST Error:', error);
  }
};
```

### 2. 第三方函式庫 `Axios` (推薦)

`Axios` 是一個基於 Promise 的 HTTP 客戶端，可以在瀏覽器和 Node.js 中使用。它提供了更豐富的功能和更友好的 API，是前端開發中非常流行的選擇。

**安裝 Axios**：

```bash
npm install axios
# 或者
yarn add axios
```

**使用 Axios**：

```javascript
// 使用 Axios 獲取數據
import axios from 'axios';

const fetchDataWithAxios = async () => {
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/todos/1');
    console.log('Axios Data:', response.data); // Axios 會自動解析 JSON，數據在 response.data 中
  } catch (error) {
    console.error('Axios Error:', error);
    // Axios 的錯誤處理更方便，可以直接訪問 error.response, error.request, error.message
  }
};

// 使用 Axios 提交數據 (POST)
const postDataWithAxios = async () => {
  try {
    const response = await axios.post('https://jsonplaceholder.typicode.com/posts', {
      title: 'foo',
      body: 'bar',
      userId: 1,
    });
    console.log('Axios POST Data:', response.data);
  } catch (error) {
    console.error('Axios POST Error:', error);
  }
};
```

**為什麼推薦 Axios？**

*   **自動 JSON 解析**：`Axios` 會自動將響應數據解析為 JSON 物件，無需手動 `response.json()`。
*   **統一的錯誤處理**：提供了更詳細的錯誤資訊，方便除錯。
*   **請求/響應攔截器**：可以在請求發送前或響應接收後進行統一處理（例如添加 Token、錯誤日誌）。
*   **取消請求**：支援取消正在進行的請求。
*   **進度追蹤**：支援上傳/下載進度追蹤。

## 優雅地處理 Loading 與 Error 狀態：讓用戶知道發生了什麼

在發送 API 請求時，數據的獲取是一個非同步過程。這期間，用戶可能會看到空白頁面，或者操作沒有反應。因此，優雅地處理「載入中 (Loading)」和「錯誤 (Error)」狀態至關重要，這能大大提升使用者體驗。

通常，我們會結合 Pinia (或組件自身的響應式狀態) 來管理這些狀態。

```html
<!-- src/components/UserFetcher.vue -->
<script setup>
import { ref } from 'vue';
import axios from 'axios';

const user = ref(null);
const isLoading = ref(false);
const error = ref(null);

const fetchUser = async () => {
  isLoading.value = true; // 開始載入，設定為 true
  error.value = null; // 清除之前的錯誤
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/users/1');
    user.value = response.data;
  } catch (err) {
    error.value = '載入用戶數據失敗：' + err.message; // 捕獲錯誤
  } finally {
    isLoading.value = false; // 載入結束，設定為 false
  }
};

// 組件掛載時自動獲取數據
import { onMounted } from 'vue';
onMounted(fetchUser);
</script>

<template>
  <div>
    <h1>用戶數據</h1>
    <button @click="fetchUser" :disabled="isLoading">重新載入</button>

    <div v-if="isLoading">
      <p>載入中，請稍候...</p>
    </div>

    <div v-else-if="error">
      <p style="color: red;">{{ error }}</p>
    </div>

    <div v-else-if="user">
      <p>姓名: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
    </div>

    <div v-else>
      <p>沒有數據</p>
    </div>
  </div>
</template>
```

### 結合 Pinia 進行狀態管理 (推薦)

將 Loading 和 Error 狀態集中到 Pinia Store 中管理，可以讓多個組件共享這些狀態，避免重複邏輯。

```javascript
// src/stores/user.js
import { defineStore } from 'pinia';
import axios from 'axios';

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    isLoading: false,
    error: null,
  }),
  actions: {
    async fetchUser() {
      this.isLoading = true;
      this.error = null;
      try {
        const response = await axios.get('https://jsonplaceholder.typicode.com/users/1');
        this.user = response.data;
      } catch (err) {
        this.error = '載入用戶數據失敗：' + err.message;
      } finally {
        this.isLoading = false;
      }
    },
  },
});
```

在組件中使用：

```html
<!-- src/components/UserDisplay.vue -->
<script setup>
import { useUserStore } from '../stores/user';
import { storeToRefs } from 'pinia';
import { onMounted } from 'vue';

const userStore = useUserStore();
const { user, isLoading, error } = storeToRefs(userStore);

onMounted(() => {
  userStore.fetchUser(); // 組件掛載時觸發獲取數據的 action
});
</script>

<template>
  <div>
    <h1>用戶數據 (來自 Pinia)</h1>
    <button @click="userStore.fetchUser()" :disabled="isLoading">重新載入</button>

    <div v-if="isLoading">
      <p>載入中，請稍候...</p>
    </div>

    <div v-else-if="error">
      <p style="color: red;">{{ error }}</p>
    </div>

    <div v-else-if="user">
      <p>姓名: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
    </div>

    <div v-else>
      <p>沒有數據</p>
    </div>
  </div>
</template>
```

## 小技巧與注意事項：

*   **錯誤處理**：錯誤處理就像是給你的 API 請求買保險！別以為請求一定會成功，網路不穩、伺服器心情不好... 什麼都可能發生。用 `try...catch` 把錯誤「抓」起來，然後告訴用戶發生了什麼事，或是偷偷記下來（console.error），別讓你的應用程式「裸奔」！
*   **請求攔截器 (Axios Interceptors)**：攔截器就像是 API 請求的「海關」和「安檢」。請求發出去前，先檢查一下有沒有帶「通行證」（Token）；回應回來後，先看看是不是「違禁品」（錯誤碼），然後再決定要不要放行。這樣你就不需要在每個請求裡重複寫這些邏輯，省心又省力！
*   **取消請求**：想像用戶是個急性子，點了又點、滑了又滑。如果你不取消舊請求，就像你點了十份外賣，結果只想要一份。用「取消請求」，就像打電話給外賣小哥說：「不好意思，剛才點錯了，那份不要了！」避免伺服器和瀏覽器「過勞死」，也避免數據「亂入」。
*   **數據預載入**：數據預載入就像是「提前備菜」。用戶還沒進餐廳（頁面），你就先把菜準備好。這樣等用戶一坐下，菜馬上就能上桌，不會看到空蕩蕩的桌面（頁面閃爍），體驗超棒！
*   **集中式 API 服務**：
    *   別讓你的 API 請求「散兵遊勇」！把所有跟 API 溝通的邏輯都集中到一個「司令部」（`api.js`），統一管理。這樣，無論是改基地 URL、加 Token，還是處理錯誤，都只需要改一個地方，就像有了「中央控制室」，效率高到爆！
    ```javascript
    // src/services/api.js
    import axios from 'axios';

    const api = axios.create({
      baseURL: 'https://api.example.com',
      timeout: 10000, // 請求超時時間
      headers: { 'Content-Type': 'application/json' }
    });

    // 請求攔截器：例如添加認證 Token
    api.interceptors.request.use(
      config => {
        const token = localStorage.getItem('authToken');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      error => {
        return Promise.reject(error);
      }
    );

    // 響應攔截器：例如統一處理錯誤響應
    api.interceptors.response.use(
      response => response,
      error => {
        if (error.response) {
          // 伺服器返回錯誤狀態碼
          console.error('API Error:', error.response.status, error.response.data);
          // 根據狀態碼進行處理，例如 401 跳轉登入頁
          if (error.response.status === 401) {
            // router.push('/login');
          }
        } else if (error.request) {
          // 請求已發出但沒有收到響應
          console.error('No response received:', error.request);
        } else {
          // 其他錯誤
          console.error('Error:', error.message);
        }
        return Promise.reject(error);
      }
    );

    export default api;
    ```
    然後在你的 Store 或組件中這樣使用：
    ```javascript
    // import api from '../services/api';
    // const response = await api.get('/users');
    ```
*   **數據獲取模式 (Data Fetching Patterns)**：
    *   數據怎麼「上菜」？有兩種常見模式：
        *   **「一開門就上菜」(`onMounted`)**：組件一掛載就發請求，適合頁面首次載入的數據。
        *   **「看客人臉色上菜」(`watch`)**：當用戶的「點菜單」（搜索詞、篩選器）變了，就重新上菜。記得給「點菜單」加個「防抖」（debounce），別讓廚師（伺服器）忙不過來！
    ```javascript
    // 範例：監聽搜索關鍵字變化來獲取數據
    import { ref, watch } from 'vue';
    // ...
    const searchTerm = ref('');
    watch(searchTerm, (newTerm) => {
      if (newTerm.length > 2) {
        // 觸發 API 請求
        // fetchUsers(newTerm);
      }
    }, { debounce: 300 }); // 可以考慮防抖 (debounce) 來減少請求頻率
    ```
*   **`Composable` 封裝 API 邏輯**：
    *   把 API 請求、Loading、Error 這些「三件套」打包成一個「萬用工具」（例如 `useFetch` 或 `useApi`）。這樣，無論哪個組件需要「取貨」，直接拿這個工具去用就行，不用每次都重新組裝，省時省力，還能保證品質！
    ```javascript
    // src/composables/useFetch.js (簡化範例)
    import { ref } 'vue';
    import axios from 'axios';

    export function useFetch(url) {
      const data = ref(null);
      const error = ref(null);
      const isLoading = ref(false);

      const fetchData = async () => {
        isLoading.value = true;
        error.value = null;
        try {
          const response = await axios.get(url);
          data.value = response.data;
        } catch (err) {
          error.value = err;
        } finally {
          isLoading.value = false;
        }
      };

      return { data, error, isLoading, fetchData };
    }
    ```
    然後在組件中使用：
    ```javascript
    // import { useFetch } from '../composables/useFetch';
    // const { data, error, isLoading, fetchData } = useFetch('https://api.example.com/posts');
    // onMounted(fetchData);
    ```
*   **樂觀更新 (Optimistic Updates)**：
    *   樂觀更新就像是「先斬後奏」！用戶點了個讚，你先讓讚的數字馬上變，讓用戶感覺超快。然後再偷偷去跟伺服器說：「我點讚了喔！」如果伺服器說不行，再把數字改回來。這招能讓你的應用程式「飛」起來，但記得要準備好「後悔藥」（回滾邏輯）喔！

## 本篇自我挑戰

-   **今日挑戰**：選擇一個你感興趣的公開 API（例如：[JSONPlaceholder](https://jsonplaceholder.typicode.com/)、[PokéAPI](https://pokeapi.co/)），嘗試在你的 Vue 應用程式中，使用 `Axios` 獲取數據，並將 `isLoading`、`error` 和獲取到的數據儲存到一個 Pinia Store 中。在組件中顯示這些狀態，並添加一個「重新載入」按鈕。
-   **反思**：將 API 請求的 Loading 和 Error 狀態集中管理，對於大型應用程式的開發和維護有什麼好處？

## 總結

今天我們學習了如何在 Vue 應用程式中進行 API 串接，並重點探討了如何優雅地處理數據的載入 (Loading) 和錯誤 (Error) 狀態。我們比較了 `fetch` 和 `Axios`，並推薦使用 `Axios` 來簡化請求處理。結合 Pinia 進行狀態管理，可以讓你的數據流更加清晰、可預測，大大提升使用者體驗和開發效率。

本日關鍵字回顧

-   **API 串接**: 前端與後端數據交換。
-   **`fetch`**: 瀏覽器原生 API。
-   **`Axios`**: 基於 Promise 的 HTTP 客戶端，功能更強大。
-   **Loading 狀態**: 數據載入中。
-   **Error 狀態**: 數據載入失敗。
-   **`try...catch...finally`**: 錯誤處理機制。
-   **請求攔截器**: `Axios` 的高級功能。
-   **取消請求**: 避免不必要的請求和競態條件。
-   **Pinia**: 狀態管理庫。
-   **集中式 API 服務**: 封裝 Axios 實例和請求邏輯。
-   **數據獲取模式**: `onMounted`, `watch` 等觸發請求的時機。
-   **`Composable` 封裝**: 將 API 邏輯抽離成可複用函式。
-   **樂觀更新**: 提升使用者體驗的 UI 技巧。

明天，我們將進入另一個實用主題——表單處理與驗證，學習如何讓你的表單既美觀又健壯！
