# 【Day 11】前端世界的導航系統：Vue Router 路由管理

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

在前面的章節中，我們學會了如何建立組件、讓它們溝通、甚至用 `Composable` 函式來封裝和複用邏輯。我們的應用程式已經具備了強大的「內在」。但一個現代化的網站，很少只有一個頁面。

我們需要首頁、關於我們、產品列表、使用者個人資料頁...等等。如果每次切換頁面都要向伺服器重新請求、載入整個網頁，那體驗就太糟糕了。

這就是 **客戶端路由 (Client-Side Routing)** 發揮作用的地方。今天，我們將學習 Vue 的官方路由管理器：**Vue Router**，它能讓我們打造出如絲般滑順的 **單頁式應用 (Single-Page Application, SPA)**。

## 什麼是單頁式應用 (SPA)？

傳統網站是「多頁式應用 (MPA)」，你點一個連結，瀏覽器就向伺服器請求一個全新的 HTML 檔案，然後載入、渲染，整個頁面會「刷新」。

而 SPA 則更像一個桌面應用程式。它只在初次載入時，從伺服器獲取一個主要的 HTML 檔案及所有必要的資源 (JS/CSS)。之後，當使用者點擊導覽連結時，它**不會重新載入整個頁面**，而是透過 JavaScript 動態地攔截導覽，並在不刷新的情況下，切換顯示對應的「視圖 (View)」或「組件」。

這帶來的好處是：
- **更快的反應速度**：使用者體驗極佳，感覺就像在本機操作一樣。
- **減輕伺服器負擔**：後端只需專注於提供 API 資料，不用再管頁面渲染。

## 深入 Vue Router 的配置

雖然我們說有核心三劍客，但真正的威力都藏在細節裡。讓我們先深入 `router/index.js`，看看這份「導航地圖」還能做些什麼。

### 1. `createRouter` 的核心選項

`createRouter` 是我們建立路由實例的地方，它接收一個選項物件。

```javascript
import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router';

const router = createRouter({
  // history: 控制路由模式，最常用的是兩種
  // 1. createWebHistory(): HTML5 模式，URL 好看，但需要後端配合處理 404
  // 2. createWebHashHistory(): Hash 模式，URL 裡有個 #，對 SEO 不友善，但不需要後端配置
  history: createWebHistory(),

  // routes: 路由規則的陣列，這是核心中的核心
  routes: [
    // ... 路由規則
  ],

  // scrollBehavior: 控制頁面切換時的滾動行為
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      // 如果有儲存的捲動位置（例如按上一頁），就回到那個位置
      return savedPosition;
    } else {
      // 否則，一律滾動到頁面頂部
      return { top: 0, behavior: 'smooth' };
    }
  },
});
```

### 2. 解構單一路由規則 (RouteRecord)

`routes` 陣列中的每一個物件，都是一條路由規則，它也有許多強大的屬性：

```javascript
{
  // path: (必須) URL 路徑，支援動態參數，例如 /users/:id
  path: '/user/:id',

  // component: (必須) 對應要渲染的組件
  component: () => import('../views/User.vue'), // 推薦使用動態載入，優化首頁速度

  // name: 路由的唯一名稱，方便在程式中進行導航，例如 router.push({ name: 'User', params: { id: 123 }})
  name: 'User',

  // redirect: 重新導向，可以是一個路徑字串，或是一個函式
  // redirect: '/team/:id',
  redirect: to => ({ path: '/team', query: { id: to.params.id } }),

  // alias: 路由的「別名」，/profile 會和 /user/:id 渲染同一個組件，但 URL 保持 /profile
  alias: '/profile/:id',

  // props: 將路由參數解耦，直接當作 props 傳入組件，讓組件更獨立
  // true -> 將 route.params 直接當作 props 傳入
  // 物件 -> 傳入靜態的 props
  // 函式 -> 自訂傳入的 props
  props: route => ({ userId: route.params.id, query: route.query.q }),

  // meta: 元資料，可以放任何自訂的資訊，最常用來做權限驗證
  meta: {
    requiresAuth: true, // 這個頁面需要登入
    title: '使用者中心'
  },

  // children: 巢狀路由，用來實現複雜的頁面佈局
  children: [
    {
      // 當 URL 是 /user/:id/profile 時，UserProfile 會被渲染在 User 組件的 <router-view> 中
      path: 'profile', 
      component: UserProfile,
    },
    {
      path: 'posts',
      component: UserPosts,
    },
  ],
}
```
> **小提示**：在 `children` 陣列中的 `path` **不可以**用 `/` 開頭，它會被自動接在父層路徑的後面。

## 導航守衛 (Navigation Guards)：路由的守門員

Vue Router 提供了一套「導航守衛」機制，它就像是每個路由路口上的保全人員，讓你在使用者**進入**、**離開**或**更新**路由時，可以執行一些邏輯。這對於權限檢查、資料預先載入、或防止使用者意外離開未儲存的頁面等場景，至關重要。

導航守衛有三個關鍵參數：`to`, `from`, `next`。
- `to`: 即將要進入的目標路由物件。
- `from`: 當前正要離開的路由物件。
- `next`: 一個**必須被呼叫**的函式，用來解析這個守衛。
  - `next()`: 正常放行，進入下一個守衛。
  - `next(false)`: 中斷當前的導航。
  - `next('/')` 或 `next({ name: 'Login' })`: 重導向到一個不同的位址。

### 全域前置守衛 (`router.beforeEach`)

這是最常用的一個守衛，它在**每一次**路由切換前都會被觸發。最適合用來做全站的登入權限驗證。

**`router/index.js`**
```javascript
router.beforeEach((to, from, next) => {
  const isLoggedIn = !!localStorage.getItem('token'); // 簡易的登入判斷

  // 檢查目標路由是否需要登入
  if (to.meta.requiresAuth && !isLoggedIn) {
    // 如果需要登入但使用者未登入，就導向登入頁
    next({ name: 'Login', query: { redirect: to.fullPath } });
  } else {
    // 否則，正常放行
    next();
  }
});
```

### 組件內守衛 (In-Component Guards)

有時候，你只想在特定組件中處理路由邏輯。例如，當使用者正在編輯表單但尚未儲存時，阻止他離開。

**`EditForm.vue`**
```html
<script setup>
import { ref } from 'vue';
import { onBeforeRouteLeave } from 'vue-router';

const isFormDirty = ref(true); // 假設表單已被修改

// 在使用者嘗試離開此路由時觸發
onBeforeRouteLeave((to, from) => {
  if (isFormDirty.value) {
    const answer = window.confirm('您有未儲存的變更，確定要離開嗎？');
    if (!answer) {
      return false; // 如果使用者選擇「取消」，則中斷導航
    }
  }
});
</script>
```
除了 `onBeforeRouteLeave`，還有 `onBeforeRouteUpdate`，它在當前路由改變，但該組件仍然被複用時呼叫（例如，從 `/users/1` 導航到 `/users/2`）。

## 本篇自我挑戰

- **思考一：設計部落格路由**
  請試著設計一個簡單部落格的路由表。它需要包含：
  1.  一個顯示所有文章列表的首頁 (`/`)。
  2.  一個顯示單篇文章內容的頁面 (`/posts/:postId`)。
  3.  一個「關於我」的頁面 (`/about`)。

- **思考二：`useRoute` vs `useRouter`**
  Vue Router 提供了兩個很像的 Composable 函式：`useRoute()` 和 `useRouter()`。它們有什麼不同？分別應該在什麼情境下使用？ (提示：一個是唯讀的資訊，一個是可執行的方法)

## 總結

今天我們學會了如何使用 Vue Router 來為我們的應用程式增加「導航」功能，正式從單一組件的練習，邁向了多視圖的「單頁式應用」。

- **SPA (Single-Page Application)**：只載入一次，後續靠 JavaScript 動態切換內容，提供流暢的使用者體驗。
- **`routes`**：定義 URL 路徑與組件對應關係的「導航地圖」。
- **`<router-link>`**：取代 `<a>` 標籤，用來產生無刷新的導航連結。
- **`<router-view>`**：用來顯示當前路由對應組件的「渲染出口」。
- **動態路由**：使用 `:param` 的形式來匹配一類模式的 URL。
- **巢狀路由**：透過 `children` 屬性，實現複雜的頁面佈局。

我們的應用程式現在擁有了不同的「房間」(頁面)，但一個新的問題出現了：如果住在不同房間的人需要共享一個物品（資料），該怎麼辦？明天，我們將學習新一代的 Vue 狀態管理器 **Pinia**，來解決跨組件的資料共享問題！