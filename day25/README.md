# 【Day 25】給測試一個乾淨的世界：Mocking API 請求

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

到目前為止，我們的測試都運行在一個「無菌室」裡。元件和函式之間的互動都在我們的完全掌控之中。但真實世界的應用，總是需要和外界溝通的，最常見的就是透過 **API 請求**來獲取或提交資料。

想像一下，如果我們在自動化測試中，真的去呼叫後端的 API，會發生什麼事？

1.  **速度慢**：網路請求動輒數百毫秒，成千上百個測試跑起來會等到天荒地老。
2.  **不穩定**：後端伺服器可能剛好在維護、網路可能不通。任何測試環境之外的因素，都可能導致你的測試無故失敗。
3.  **狀態不可控**：測試依賴資料庫中的特定資料。如果有人把那筆資料刪了，測試就壞了。你沒辦法保證每次測試都在同樣的資料基礎上運行。
4.  **有風險/成本**：你不會希望你的測試在資料庫裡建立一萬筆假使用者，或是不小心觸發了寄送真實 Email 的功能吧？

測試應該是**快速、穩定、可預測**的。為此，我們需要將測試與外部世界隔離開來。這就要用到一個核心測試技巧：**Mocking (模擬)**。

**打個比方**：當電影要拍一段在美國白宮橢圓辦公室的場景時，劇組不會真的跑去白宮。他們會搭建一個精確的「**模擬場景 (Mock Set)**」。這個場景對攝影機來說跟真的一樣，但它完全在劇組的掌控之下，可以隨意打光、移動牆壁。**Mocking API** 就是同樣的道理：我們打造一個假的 API，它對我們的元件來說跟真的一樣，但它的回應完全由我們在測試中掌控。

## 1. 什麼是 Mocking？

在測試中，Mocking 指的是「**用一個我們控制的『假』物件，來取代一個真實的外部依賴**」。

對於 API 請求，我們的目標就是：找到發出網路請求的那段程式碼 (例如 `axios.get`)，然後把它換成一個我們預先寫好的假函式，這個假函式會立刻回傳我們指定的假資料，完全不會有任何網路流量產生。

Vitest 提供了強大的內建 Mocking 功能，讓我們能輕易地替換掉整個模組。

## 2. 使用 `vi.mock` 模擬模組

`vi.mock` 是 Vitest 的一把瑞士刀，能讓我們攔截對某個模組的 `import`，並將其替換為我們定義的 Mock 版本。

**測試情境**：一個 `UserPost.vue` 元件，它在掛載時會呼叫 API 來獲取文章資料並顯示。

**我們的 API 服務 (`postService.js`)**
```javascript
// src/services/postService.js
import axios from 'axios';

export const fetchPost = async (id) => {
  const response = await axios.get(`https://api.example.com/posts/${id}`);
  return response.data;
};
```

**我們的元件 (`UserPost.vue`)**
```vue
// src/components/UserPost.vue
<template>
  <div v-if="error">{{ error }}</div>
  <div v-else-if="post">
    <h1>{{ post.title }}</h1>
    <p>{{ post.body }}</p>
  </div>
  <div v-else>Loading post...</div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { fetchPost } from '@/services/postService';

const post = ref(null);
const error = ref(null);

onMounted(async () => {
  try {
    post.value = await fetchPost(1);
  } catch (e) {
    error.value = 'Failed to fetch post.';
  }
});
</script>
```

**我們的測試 (`UserPost.spec.js`)**

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/vue';
import UserPost from './UserPost.vue';

// 關鍵！我們需要從真實模組 import，但 Vitest 會幫我們把它換成 Mock
import { fetchPost } from '@/services/postService';

// vi.mock 必須寫在頂層作用域，Vitest 會在 import 之前自動提升 (hoist) 它
vi.mock('@/services/postService', () => ({
  // 我們告訴 Vitest，當有人 import fetchPost 時，給他這個假的 vi.fn()
  fetchPost: vi.fn(),
}));

describe('UserPost.vue', () => {
  it('fetches and displays a post on success', async () => {
    const mockPost = {
      title: 'Hello, Vue Testing',
      body: 'This is a mocked post.',
    };

    // 設定這次測試中，當 fetchPost 被呼叫時，它應該「假裝」成功並回傳 mockPost
    fetchPost.mockResolvedValue(mockPost);

    render(UserPost);

    // 初始狀態應該是 Loading
    expect(screen.getByText('Loading post...')).toBeInTheDocument();

    // 等待 API 呼叫完成後，斷言文章內容是否被正確渲染
    expect(await screen.findByText(mockPost.title)).toBeInTheDocument();
    expect(screen.getByText(mockPost.body)).toBeInTheDocument();

    // 我們還可以檢查 Mock 函式是否被正確呼叫
    expect(fetchPost).toHaveBeenCalledTimes(1);
    expect(fetchPost).toHaveBeenCalledWith(1);
  });

  it('displays an error message on failure', async () => {
    // 這次，我們設定 fetchPost 應該「假裝」失敗
    fetchPost.mockRejectedValue(new Error('API Error'));

    render(UserPost);

    // 等待 API 呼叫失敗後，斷言錯誤訊息是否被正確渲染
    expect(await screen.findByText('Failed to fetch post.')).toBeInTheDocument();
  });
});
```

> **Mocking 工具箱**：
> - `vi.mock('path', factory)`: 攔截模組導入，用 `factory` 回傳的內容取而代之。
> - `vi.fn()`: 建立一個空白的 Mock 函式，它像一個間諜，能記錄自己被如何呼叫。
> - `.mockResolvedValue(value)`: 設定 Mock 函式非同步回傳一個成功的值。
> - `.mockRejectedValue(error)`: 設定 Mock 函式非同步回傳一個失敗的錯誤。
> - `.toHaveBeenCalledWith(...args)`: 斷言 Mock 函式是否曾用指定的參數被呼叫。

## 3. 更優雅的選擇：Mock Service Worker (MSW)

`vi.mock` 非常強大，但它有一個缺點：你的測試程式碼需要明確地知道要 Mock 哪個模組 (`@/services/postService`)。這是一種耦合。

有沒有一種方法，讓我們的元件和測試程式碼**完全不知道 Mock 的存在**，就像在跟真實 API 互動一樣？

答案是 **Mock Service Worker (MSW)**。

**MSW 的比喻**：如果說 `vi.mock` 是在片場把「演員 A」換成「替身演員 B」，那麼 MSW 就是在攝影棚和真實世界之間，設立了一個「**數位特效工作室**」。

你的元件（演員）正常地發出一個網路請求（表演），這個請求在離開攝影棚的瞬間，被 MSW 攔截下來，然後 MSW 回傳一個以假亂真的數位特效畫面（Mock 回應）。演員自始至終都以為自己在跟真實世界互動。

**MSW 的優點**：
-   **零侵入**：你的元件程式碼完全不用改，測試程式碼也變得極度乾淨。
-   **網路層攔截**：它運作在網路層，所以無論你用 `axios`, `fetch` 還是其他庫，都能攔截。
-   **可重用**：可以在單元測試、整合測試、E2E 測試，甚至開發環境中使用。

MSW 的設定稍微複雜，需要定義 `handlers` 和設定 `server`，這超出了我們今天的範圍。但你必須知道，當你的專案規模變大，API 數量變多時，MSW 是管理 API Mock 的黃金標準。

## 本篇自我挑戰

1.  **練習 `vi.mock`**：挑選一個你的專案中有進行 API 請求的元件。
2.  **撰寫成功情境**：使用 `vi.mock` 和 `mockResolvedValue`，模擬 API 成功回傳資料，並斷言資料是否正確顯示在畫面上。
3.  **撰寫失敗情境**：使用 `mockRejectedValue`，模擬 API 請求失敗，並斷言畫面上是否出現了對應的錯誤提示訊息。

## 總結

今天，我們學會了如何為測試建立一個乾淨、可控的環境，將不穩定的外部 API 請求拒之門外。

-   我們理解了**為什麼**在測試中要避免真實的 API 請求（速度、穩定性、可預測性）。
-   我們掌握了使用 Vitest 內建的 **`vi.mock`** 來替換整個模組，並用 `.mockResolvedValue` / `.mockRejectedValue` 來控制其回傳值。
-   我們認識了更先進的網路層 Mock 工具 **Mock Service Worker (MSW)**，它是大型專案的理想選擇。

掌握 Mocking，是從「只能測試純函式」到「能測試真實世界應用」的關鍵一步。它讓我們的測試套件變得真正強大而可靠。

明天，我們將退一步，從宏觀的角度來看看不同測試類型之間的區別與取捨：**單元測試 vs. E2E 測試**。

## 本日關鍵字回顧

-   `Mocking (模擬)`
-   `依賴 (Dependency)`
-   `vi.mock`
-   `vi.fn()`
-   `mockResolvedValue`
-   `mockRejectedValue`
-   `toHaveBeenCalledWith`
-   `Mock Service Worker (MSW)`
-   `網路層攔截 (Network-level Interception)`
