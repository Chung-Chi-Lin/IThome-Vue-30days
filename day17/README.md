# 【Day 17】TypeScript 的讀心術：`ref` 與 `reactive` 的型別自動推論

## 聯繫我
如果有任何問題或建議，歡迎隨時聯繫我：

- [GitHub](https://github.com/Chung-Chi-Lin)
- [Email](mailto:z0925955648@gmail.com)

## 前言

各位 Vue 偵探們！昨天我們學會了如何為組件的 Props 和 Emits 製作精密的「身分證」和「加密對講機」。你可能會覺得：「天啊，難道我以後寫每個變數都要手動加上型別嗎？也太累了吧！」

別擔心！TypeScript 這位架構師，其實也是一位非常聰明的「**偵探福爾摩斯**」。很多時候，你不需要事事都告訴他，他會自己觀察線索，然後推斷出事情的真相。這就是我們今天要聊的——**型別推論 (Type Inference)**！

型別推論是 TS 最貼心的功能之一。它讓你在享受型別安全的好處時，又不必犧牲太多的開發效率。讓我們來看看這位偵探是如何為 Vue 的 `ref` 和 `reactive` 進行「讀心」的。

## `ref` 的型別推論：一目了然的線索

當你使用 `ref` 並給它一個初始值時，TS 偵探就會立刻根據這個「第一現場的線索」來判斷它的型別。

```typescript
import { ref } from 'vue';

// 線索：初始值是 0 (一個數字)
// 偵探推斷：OK，這個 `count` 變數以後就專門放數字了。
const count = ref(0);
// 此時 `count` 的型別被推斷為 Ref<number>

// 嘗試賦予一個字串值
// count.value = 'hello'; // TS 立刻抗議：喂！說好了放數字的！(Type 'string' is not assignable to type 'number'.)

// 線索：初始值是一個物件
// 偵探推斷：嗯，這個 `user` 變數的規格是 { name: string }。
const user = ref({ name: 'Watson' });
// `user` 的型別被推斷為 Ref<{ name: string; }>

// user.value.name = 'Holmes'; // 合法
// user.value.age = 30; // TS 抗議：一開始沒說有 age 這個屬性啊！(Property 'age' does not exist on type '{ name: string; }'.)
```

### 當線索不足時：你需要提供額外情報

有時候，初始線索太模糊，偵探也沒辦法。例如，一個變數一開始是 `null`，但之後會被賦予一個物件。

```typescript
import { ref } from 'vue';

interface User {
  id: number;
  name: string;
}

// 線索：初始值是 null
// 偵探推斷：OK，這個 `userData` 永遠都只能是 null。
const userData = ref(null);
// `userData` 的型別被推斷為 Ref<null>

// 之後從 API 獲取了用戶資料
// userData.value = { id: 1, name: 'Moriarty' }; // TS 大力反對：你騙我！說好是 null 的！(Type 'User' is not assignable to type 'null'.)
```

這時候，你就必須主動提供情報給偵探，告訴他你的「完整計畫」。我們使用「**聯合型別 (Union Types)**」來解決這個問題。

```typescript
// 我們明確告訴 TS：這個 ref 要嘛是 User 型別，要嘛是 null
const userData = ref<User | null>(null);

// 這樣一來，之後的賦值就完全合法了
userData.value = { id: 1, name: 'Moriarty' }; // 偵探點點頭：嗯，在計畫之中。
```

## `reactive` 的型別推論：整個犯罪現場都在掌控之中

`reactive` 用於處理物件，TS 偵探對它的推論更加全面。它會直接鎖定整個物件的「結構」。

```typescript
import { reactive } from 'vue';

// 線索：一個包含 count 和 name 的物件
// 偵探推斷：記下了，這個 state 的形狀就是 { count: number; name: string; }
const state = reactive({ count: 0, name: 'Moriarty' });

// state.count = 1; // 合法
// state.name = 'Holmes'; // 合法

// state.count = 'one'; // TS 警告：型別錯誤！
// state.address = '221B Baker Street'; // TS 警告：一開始沒有這個屬性！
```

## 警惕「any」陷阱：當偵探放棄辦案

如果你在宣告 `ref` 時不提供任何初始值，會發生什麼事？

```typescript
const something = ref(); // 沒有初始值
// `something` 的型別被推斷為 Ref<any>
```

`any` 型別等於是告訴 TS 偵探：「**這個案子你別管了，我自己來。**」 這會讓 `something` 變回純 JavaScript 的行為，你可以對它做任何事，TS 完全不會報錯。這也意味著，你失去了 TypeScript 帶來的所有保護！

所以，請盡量避免讓 `any` 出現。要嘛給個初始值，要嘛手動指定型別。

## 總結

今天我們見識了 TypeScript 強大的「讀心術」——型別推論。

-   **型別推論 (Type Inference)**：TS 會根據變數的初始值，自動推斷出它的型別，讓我們不必手動編寫所有型別註解。
-   **簡單型別**：對於 `ref(0)` 或 `ref('hello')`，TS 的推論通常是準確且足夠的。
-   **複雜或可變型別**：當一個 `ref` 的初始值無法完全表達它未來的可能性時（如 `ref(null)`），我們需要使用**聯合型別**（`ref<User | null>(null)`）來手動提供更完整的「情報」。
-   **避免 `any`**：不要創建沒有初始值且沒有指定型別的 `ref`，這會讓型別檢查失效。

善用型別推論，可以讓我們在安全與效率之間找到完美的平衡點。

## 本篇自我挑戰

-   **今日挑戰**：檢查一下你昨天的程式碼。有沒有哪些地方的型別註解其實是可以省略的（因為 TS 可以自動推斷）？有沒有哪個 `ref` 的初始值是 `null` 或 `[]`，但之後會被賦予更複雜的資料，導致你需要手動添加型別？
-   **反思**：你認為在哪些情況下，即使 TS 能夠成功推斷，你仍然會選擇手動編寫型別？（提示：為了程式碼的可讀性或明確性）

本日關鍵字回顧

-   **型別推論 (Type Inference)**: TS 自動從程式碼上下文推斷型別的能力。
-   **聯合型別 (Union Types)**: 使用 `|` 符號來表示一個變數可以是多種型別之一，如 `string | number`。
-   **`Ref<T>`**: `ref` 物件的泛型型別，`T` 代表它內部 `.value` 的型別。
-   **`any`**: 「放棄治療」型別，會繞過所有型別檢查，應極力避免。

明天，我們將學以致用，回到 Day 10 的 `Composable`，為我們自訂的 Hook 加上完整的型別定義，打造出真正可複用、可信賴的「武功祕笈」！
