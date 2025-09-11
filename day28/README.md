# 【Day 28】縮小你的首屏負擔：Code Splitting 與 Lazy Loading

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

昨天，我們優化了應用的「**執行期 (Runtime)**」效能，讓它在載入後能流暢地運作。但還有一個同樣重要的戰場：「**載入期 (Load-time)**」效能。也就是說，使用者在輸入網址、按下 Enter 後，需要等待多久才能看到有意義的內容？

預設情況下，像 Vite 或 Webpack 這類的打包工具 (Bundler)，會將你專案中所有的 JavaScript 程式碼，打包成一個**巨大**的 `bundle.js` 檔案。隨著你的應用功能越來越多、頁面越來越複雜，這個檔案的體積也會像滾雪球一樣越變越大。

這會導致一個災難性的後果：使用者必須先下載完這整個幾 MB 大小的檔案，你的 Vue 應用才能開始渲染。這中間的漫長等待，就是我們常說的「**白畫面時間 (White Screen of Death)**」，也是使用者流失的主要原因之一。

**打個比方**：這就像搬家時，你把所有家當——床、沙發、書、廚具、衣服——全都塞進一個巨大無比的箱子裡。在你的新家，你必須等這個超級大箱子運到之後，才能開始拿出東西、佈置房間。在那之前，你只能待在空無一物的房子裡乾等。

這顯然不合理，對吧？一個更聰明的做法是，把家當分門別類地裝進不同的小箱子（「廚房」、「臥室」、「急救箱」）。先把最重要的「急救箱」送達，你至少能開始基本的生活。其他的箱子，可以等你需要時再陸續送到。

這個「**分箱打包**」的策略，在前端世界裡就叫做 **Code Splitting (程式碼分割)**，而「**需要時再送到**」的戰術，就叫做 **Lazy Loading (延遲載入)**。

## 1. 路由層級的 Lazy Loading

**這是最常用，也是最有效的程式碼分割方式。**

-   **核心思想**：使用者在訪問 `A` 頁面時，根本不需要 `B`、`C`、`D` 頁面的程式碼。我們應該只載入當前路由對應的程式碼。當使用者導航到新路由時，再即時去下載該路由的程式碼。

-   **比喻**：這就像你看 Netflix。你不會在點開一部電影時，就把整個 Netflix 影片庫都下載到你的電腦上。你只會「串流 (stream)」你正在看的那一部。

-   **如何實現**：在 `vue-router` 的設定中，將靜態的 `import`，改為動態的 `import()` 函式。

    **修改前 (所有程式碼打包在一起):**
    ```javascript
    // src/router/index.js
    import HomeView from '../views/HomeView.vue';
    import AboutView from '../views/AboutView.vue';
    import PostView from '../views/PostView.vue';

    const routes = [
      { path: '/', name: 'home', component: HomeView },
      { path: '/about', name: 'about', component: AboutView },
      { path: '/post', name: 'post', component: PostView },
    ];
    ```

    **修改後 (Lazy Loading):**
    ```javascript
    // src/router/index.js
    const routes = [
      {
        path: '/',
        name: 'home',
        component: () => import('../views/HomeView.vue') // 動態 import
      },
      {
        path: '/about',
        name: 'about',
        // Webpack/Vite 看到這個語法，就會自動將 AboutView.vue 分割成一個獨立的 chunk 檔案
        component: () => import('../views/AboutView.vue')
      },
      {
        path: '/post',
        name: 'post',
        component: () => import('../views/PostView.vue')
      }
    ];
    ```

就這麼簡單！當打包工具看到 `import()` 這個函式語法時，它就會自動將這個模組及其依賴，打包成一個獨立的 JavaScript 檔案 (chunk)。這個檔案只會在使用者第一次訪問該路由時，才被非同步地從伺服器下載。

## 2. 元件層級的 Lazy Loading

有時候，程式碼分割的粒度需要更細。在同一個頁面中，可能也存在一些「大而重」，但並非立即需要的元件。

-   **常見場景**：
    -   一個複雜的**彈出視窗 (Modal)**，只有在使用者點擊按鈕時才需要顯示。
    -   一個使用了大型圖表庫 (如 D3.js, Chart.js) 的**圖表元件**。
    -   一個位於頁面很下方的「**留言區**」，使用者需要滾動很久才能看到。

-   **解決方案**：使用 Vue 提供的 `defineAsyncComponent` 函式。

-   **比喻**：這就像一個新聞網站，文章的標題和內文會立刻載入，但頁面下方的互動式地圖或影片播放器，可以等到使用者滾動到那個位置時，再開始載入。

-   **如何實現**：用 `defineAsyncComponent` 包覆一個動態 `import()`。

    ```vue
    <template>
      <button @click="show = true">打開重量級 Modal</button>

      <!-- HeavyModal 的程式碼，只會在 show 變為 true 的那一刻才開始下載 -->
      <HeavyModal v-if="show" @close="show = false" />
    </template>

    <script setup>
    import { ref, defineAsyncComponent } from 'vue';

    const show = ref(false);

    // 使用 defineAsyncComponent 來延遲載入元件
    const HeavyModal = defineAsyncComponent(() =>
      import('@/components/HeavyModal.vue')
    );
    </script>
    ```

`defineAsyncComponent` 還可以接受一個物件，讓我們可以定義載入時的 `loadingComponent` 和載入失敗時的 `errorComponent`，提供更完善的使用者體驗。

## 3. 驗證你的成果

設定完 Lazy Loading 後，你該如何確認它真的生效了？

1.  執行 `npm run build`，觀察 `dist/assets` 資料夾。你會發現，除了主要的 `index-xxxx.js` 檔案外，還多出了很多以數字或元件名命名的 `js` 小檔案。這些就是被分割出來的 chunks。
2.  打開瀏覽器的開發者工具，切換到「**Network (網路)**」頁籤。
3.  重新整理你的網站。你會看到初始載入的 `js` 檔案體積變小了。
4.  當你導航到一個被 Lazy Load 的新路由時，你會在 Network 頁籤中看到一個新的 `js` 檔案被即時下載。

## 本篇自我挑戰

1.  **改造你的路由**：將你專案中 `vue-router` 的所有路由，全部改寫成 Lazy Loading 的形式。打開 Network 頁籤，比較一下修改前後，初始載入的檔案大小和數量有何變化。
2.  **尋找懶加載目標**：檢查你的專案，找出一個適合被 Lazy Load 的元件（例如：一個不常用的、複雜的、或隱藏在 `v-if` 後面的元件），並使用 `defineAsyncComponent` 對它進行改造。

## 總結

今天，我們學會了如何解決前端應用「過於肥胖」的問題，為使用者提供更快的首屏體驗。

-   **Code Splitting (程式碼分割)**：是一種將巨大程式碼包，拆分成多個小塊的「策略」。
-   **Lazy Loading (延遲載入)**：是一種只在需要時，才去下載這些小塊程式碼的「戰術」。
-   **路由層級的 Lazy Loading**：最重要、效益最高的分割方式，透過在 `vue-router` 中使用 `() => import(...)` 實現。
-   **元件層級的 Lazy Loading**：更細粒度的優化，透過 `defineAsyncComponent` 處理頁面中非必要的重量級元件。

掌握 Code Splitting，是所有現代前端工程師的必備技能。它能從根本上改善你的應用載入效能，給使用者留下最好的第一印象。

明天，我們將探討一個軟體工程中的永恆話題：**專案重構與可維護性**。

## 本日關鍵字回顧

-   `載入期效能 (Load-time Performance)`
-   `程式碼分割 (Code Splitting)`
-   `延遲載入 (Lazy Loading)`
-   `打包工具 (Bundler)`
-   `Chunk`
-   `動態 Import: import()`
-   `defineAsyncComponent`
-   `首屏時間 (First Paint)`
