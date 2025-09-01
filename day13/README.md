# 【Day 13】與世界對話：API 串接與非同步狀態管理

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

到目前為止，我們的應用程式就像一個自給自足的內陸國。我們有房間（路由）、有中央儲藏室（Pinia），但所有的傢俱和物品（資料）都是我們自己預先放在裡面的。這在真實世界中是遠遠不夠的。

一個動態的應用程式，需要能與外界溝通、從遠方獲取資訊。這個「外界」，通常就是後端伺服器提供的 **API (Application Programming Interface)**。

今天，我們將學習如何讓我們的 Vue 應用程式「走出去」，使用廣受歡迎的 **Axios** 函式庫來發送網路請求 (HTTP Request)，並更重要的是，學習如何優雅地在 Pinia 中管理這些**非同步操作的狀態**（載入中、成功、失敗）。

## 為什麼需要 Axios？

你可能會問：「瀏覽器不是內建了 `fetch` API 嗎？為什麼我還需要另一個函式庫？」

你說的沒錯，`fetch` 很好用。但 Axios 在真實專案開發中，提供了一些讓開發者生活更美好的便利功能：

- **更簡潔的 API**：`axios.get()` 的語法通常比 `fetch` 更直觀。
- **自動轉換 JSON**：Axios 會自動將收到的 JSON 字串轉換成 JavaScript 物件，而 `fetch` 需要手動呼叫 `.json()`。
- **更好的錯誤處理**：`fetch` 只有在網路層級出錯時才會 reject，對於像 404 或 500 這樣的 HTTP 錯誤，它仍然會 resolve，你需要自己檢查 `response.ok`。而 Axios 會直接將這些錯誤視為 Promise 的 rejection，讓 `catch` 區塊的邏輯更統一。
- **請求與回應攔截器 (Interceptors)**：這是 Axios 的王牌功能。你可以在發送請求前或收到回應後，統一對它們進行處理，例如自動加上 Token 或處理全域的錯誤提示。

## 在 Pinia Action 中串接 API

管理非同步狀態的最佳實踐，就是將所有相關的邏輯都封裝在 Pinia 的 `action` 裡面。組件只需要「觸發」這個 action，然後監聽 store 中狀態的變化即可，完全不用關心 API 是如何被呼叫的。

一個典型的非同步 action 包含三個狀態：
1.  `data`：成功時，用來儲存 API 回傳的資料。
2.  `isLoading`：一個布林值，表示請求是否正在進行中。
3.  `error`：請求失敗時，用來儲存錯誤訊息。

讓我們來看一個獲取使用者列表的例子。

**`stores/userStore.js`**
```javascript
import { defineStore } from 'pinia';
import axios from 'axios';

export const useUserStore = defineStore('user', {
  state: () => ({
    users: [],
    isLoading: false,
    error: null,
  }),
  actions: {
    async fetchUsers() {
      // 1. 開始請求，進入載入狀態
      this.isLoading = true;
      this.error = null; // 清除之前的錯誤

      try {
        // 2. 發送 API 請求
        const response = await axios.get('https://jsonplaceholder.typicode.com/users');
        
        // 3. 請求成功，更新資料
        this.users = response.data;

      } catch (err) {
        // 4. 請求失敗，記錄錯誤訊息
        this.error = '無法獲取使用者資料：' + err.message;
        console.error(err);

      } finally {
        // 5. 無論成功或失敗，最終都結束載入狀態
        this.isLoading = false;
      }
    },
  },
});
```
> 這個 `try...catch...finally` 的結構是處理非同步操作的黃金模式，務必牢記！

## 在組件中呈現非同步狀態

一旦 Store 的邏輯建立好，組件的任務就變得非常單純：

1.  在適當的時機（例如 `onMounted`）呼叫 `fetchUsers` action。
2.  根據 `isLoading` 和 `error` 的狀態，顯示不同的 UI。

**`UserList.vue`**
```html
<script setup>
import { onMounted } from 'vue';
import { useUserStore } from '../stores/userStore';

const userStore = useUserStore();

// 在組件掛載後，觸發 action 來獲取資料
onMounted(() => {
  userStore.fetchUsers();
});
</script>

<template>
  <div>
    <h1>使用者列表</h1>

    <!-- 載入中狀態 -->
    <div v-if="userStore.isLoading">載入中...</div>

    <!-- 錯誤狀態 -->
    <div v-else-if="userStore.error" class="error">{{ userStore.error }}</div>

    <!-- 成功狀態 -->
    <ul v-else>
      <li v-for="user in userStore.users" :key="user.id">
        {{ user.name }} ({{ user.email }})
      </li>
    </ul>
  </div>
</template>
```
> 這種模式讓我們的組件職責非常清晰。組件只負責「呈現 UI」和「觸發事件」，而所有複雜的商業邏輯和狀態管理，都交給了 Pinia Store。

## 本篇自我挑戰

- **挑戰一：尋寶遊戲**
  網路上有許多免費的公開 API，例如 [JSONPlaceholder](https://jsonplaceholder.typicode.com/) 或 [The Rick and Morty API](https://rickandmortyapi.com/documentation)。請任選一個，建立一個對應的 Pinia store，並在組件中獲取並展示它的資料。

- **挑戰二：下拉刷新**
  如果想在 `UserList.vue` 中加入一個「重新整理」的按鈕，點擊後可以重新獲取最新的使用者列表。你會如何修改組件和 store 的 action 來實現這個功能？

## 總結

今天，我們為應用程式裝上了連通世界的「網路天線」。我們掌握了如何將非同步的 API 請求，優雅地整合到我們的狀態管理流程中。

- **Axios**：一個強大且易用的 HTTP 客戶端，簡化了 API 請求的處理。
- **非同步 Action**：在 Pinia 的 action 中使用 `async/await` 來處理 API 請求是標準做法。
- **管理載入狀態**：使用 `isLoading` 這樣的布林值來追蹤請求是否在進行中，對於提升使用者體驗至關重要。
- **錯誤處理**：`try...catch` 區塊讓我們可以捕捉到 API 請求失敗時的錯誤，並在 UI 上給予使用者適當的回饋。
- **職責分離**：組件負責 UI 和觸發事件，Store 負責商業邏輯和狀態管理，這讓我們的程式碼更清晰、更容易維護。

我們已經能獲取資料了，但使用者也需要能「輸入」資料。明天，我們將探討在 Web 開發中最常見也最複雜的任務之一：**表單處理與驗證**。
