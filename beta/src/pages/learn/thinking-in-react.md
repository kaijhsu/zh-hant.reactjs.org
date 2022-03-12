---
title: 用 React 思考
---

<Intro>

React 可以改變你對你所看到的設計和建立的應用程式的思考方式。以前你可能看到的是一片森林，在使用 React 之後，你會欣賞每一個樹木。React 使在設計系統和 UI 狀態中的思考變得更容易。在這份教學中，我們會帶領你走過一遍用 React 來建立一個可搜尋的產品資料表格的思考過程。

</Intro>

## 從視覺稿開始 {/*start-with-the-mockup*/}

想像一下我們已經有個 JSON API 和一個設計師給我們的產品視覺稿。

JSON API 則會回傳一些看起來像這樣的資料：

```json
[
  { category: "Fruits", price: "$1", stocked: true, name: "Apple" },
  { category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit" },
  { category: "Fruits", price: "$2", stocked: false, name: "Passionfruit" },
  { category: "Vegetables", price: "$2", stocked: true, name: "Spinach" },
  { category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin" },
  { category: "Vegetables", price: "$1", stocked: true, name: "Peas" }
]
```

這個視覺稿看起來像這樣：

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

要在 React 內實作一個 UI，你通常會遵循同樣的五個步驟。

## 第一步：將 UI 拆解成 component 層級 {/*step-1-break-the-ui-into-a-component-hierarchy*/}

首先，你要做的是將視覺稿中每一個 component 和 subcomponent 都圈起來，並且幫它們命名。如果你是跟設計師合作的話，他們可能已經透過設計工具幫你做好這一步了，所以跟他們聊聊吧！

根據你的背景，你可以考慮以不同的方式將設計拆分為 component：

* **Programming**--使用同樣的技術來決定是否要建立一個 function 或 object。其中一項技術是[單一職責原則](https://en.wikipedia.org/wiki/Single_responsibility_principle)，那就是一個 component 理想上只做一件事。如果它最終成長，那它應該被分解成更小的 subcomponent。
* **CSS**--考慮你會為什麼製作 class selector。（但是，component 的粒度要小一些。）
* **Design**--考慮你想怎麼組織你的設計的圖層。

如果你的 JSON 結構是良好的，你會經常發現它自然的映射到你的 UI component 結構。這是因為 UI 和資料模型通常具有一樣的資訊架構--也就是相同的形狀。將你的 UI 拆分成 component，每個 component 都可以與你一部分資料模型做匹配。

在這個畫面上有五個 component 組成：

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1. `FilterableProductTable` (grey) 包含整個應用程式。
2. `SearchBar`（blue）接收使用者的輸入。
3. `ProductTable`（lavender）根據使用者的輸入顯示和過濾列表。
4. `ProductCategoryRow`（green）為每個類別顯示標題。
5. `ProductRow`（yellow）為每個產品顯示一行。

</CodeDiagram>

</FullWidth>

如果你看到 `ProductTable`（lavender），會發現表格的標題列（內含「Name」和「Price」標籤 ）並非獨立的 component。要不要把它們變成 component 這個議題完全是個人的喜好，正反意見都有。對於這個範例，它是 `ProductTable` 的一部份，因為它出現在 `ProductTable` 的列表內。然而，如果這個標題變得更複雜的話（例如，加入排序功能），那麼建立一個 `ProductTableHeader` component 是非常合理的。

既然我們已經找出視覺稿中的 component 了，讓我們來安排它們的層級。在視覺稿中，在另一個 component 中出現的 component 就應該是 child：

* `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
        * `ProductCategoryRow`
        * `ProductRow`

## 第二步：在 React 中建立一個靜態版本 {/*step-2-build-a-static-version-in-react*/}

現在你已經有了 component 的層級，是時候來實作你的應用程式了。最簡單的方式是為你的應用程式建立一個接收資料模型、render UI 且沒有互動性的版本...！通常先建立靜態版本然後再單獨加上互動性通常更加容易。建立一個靜態版本需要打很多字，但不需要想很多，而加上互動性則相反，需要做很多的思考，很少的打字。

為你的應用程式建立一個 render 資料模型的版本，你會想要建立可以重複使用其他 [component](/learn/your-first-component) 的 component，並使用 [props]((/learn/passing-props-to-a-component)) 傳遞資料。Props 是將資料從 parent 傳給 child 的方式。(如果你對於 [state](/learn/state-a-components-memory) 的概念很熟悉的話，請不要使用 state 來建立靜態版本。State 是保留給互動性的，也就是會隨時間改變的資料。既然我們目前要做的是這應用程式的靜態版本，你就不需要。)
你可以「由上而下」從高層次的 component（像是 `FilterableProductTable`）開始建構，或是「由下至上」從較低的 component（像是 `ProductRow`）開始建構。在簡單的範例中，通常由上至下較容易，而在大型專案中，由下至上會較容易。

<Sandpack>

```jsx App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

（如果這些程式碼看起來很嚇人，請先看一下[快速入門]((/learn/))的內容！）

在建立你的 component 之後，你將有一個可重複使用的 component 來 render 你的資料模型。因為這是一個靜態應用程式，這些 component 將只回傳 JSX。處於層次結構頂端的 component（`FilterableProductTable`）將把你的資料模型作為一個 prop。這也稱作為_單向資料流_，因為資料從頂層的 component 流向到樹底的 component。

<Gotcha>

在這個時間點上，你不應該使用任何 state 的值。這是為下一步準備的！

</Gotcha>

## Step 3: 找出最少但完整的 UI State 的代表 {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

為了將你的 UI 變成有互動性，你需要讓使用者改變你的底層模型。為此，你將需要使用 *state*。

你首先需要思考你的應用程式最少需要哪些可變的 state。結構化 state 最重要的原則是保持它 [DRY (Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself))。請找出你的應用程式所需的最少的呈現方式，並按需求計算其他一切。例如，你要建立一個購物清單，你可以將這些項目作為 array 儲存到 state。如果你也想同時顯示購物清單內的數量，不要把項目數量作為另一個 state 來儲存--而是讀取 array 的長度。

現在思考這個範例應用程式中的所有資料：

1. 原本的產品列表
2. 使用者輸入的搜尋關鍵字
3. Checkbox 的值
4. 篩選過後的產品列表

其中哪些是 state？找出那些不是的：

* 隨著時間的推移，它是否**保持不變**？如果是，那它不是 state。
* 它是**通過 prop 從 parent 傳來的**嗎？如果是，那它不是 state。
* 你可以基於現有 component 內存在的 state 或 prop **計算它**嗎？如果可以，它*絕對*不是 state！

剩下的可能是 state。

讓我們再一一的介紹一次：

1. 產品原始列表是**作為 prop 傳入的，所以它不是 state**。
2. 搜尋文字似乎是 state，因為它會隨著時間變化，而且無法從任何其他地方計算出來。
3. Checkbox 似乎是 state，因為它會隨著時間變化，而且無法從任何其他地方計算出來。
4. 篩選過的產品列表**不是狀態**，因為它可以透過原始產品列表並根據搜尋文字和 checkbox 的值**被計算**出來的。

這代表著只有搜尋文字和 checkbox 的值是 state！做得好！

<DeepDive title="Props vs State">

There are two types of "model" data in React: props and state. The two are very different:

* [**Props** are like arguments you pass](/learn/passing-props-to-a-component) to a function. They let a parent component pass data to a child component and customize its appearance. For example, a `Form` can pass a `color` prop to a `Button`.
* [**State** is like a component’s memory.](/learn/state-a-components-memory) It lets a component keep track of some information and change it in response to interactions. For example, a `Button` might keep track of `isHovered` state.

Props and state are different, but they work together. A parent component will often keep some information in state (so that it can change it), and *pass it down* to child components as their props. It's okay if the difference still feels fuzzy on the first read. It takes a bit of practice for it to really stick!

</DeepDive>

## Step 4: Identify where your state should live {/*step-4-identify-where-your-state-should-live*/}

After identifying your app’s minimal state data, you need to identify which component is responsible for changing this state, or *owns* the state. Remember: React uses one-way data flow, passing data down the component hierarchy from parent to child component. It may not be immediately clear which component should own what state. This can be challenging if you’re new to this concept, but you can figure it out by following these steps!

For each piece of state in your application:

1. Identify *every* component that renders something based on that state.
2. Find their closest common parent component--a component above them all in the hierarchy.
3. Decide where the state should live:
    1. Often, you can put the state directly into their common parent.
    2. You can also put the state into some component above their common parent.
    3. If you can't find a component where it makes sense to own the state, create a new component solely for holding the state and add it somewhere in the hierarchy above the common parent component.

In the previous step, you found two pieces of state in this application: the search input text, and the value of the checkbox. In this example, they always appear together, so it is easier to think of them as a single piece of state.

Now let's run through our strategy for this state:

1. **Identify components that use state:**
    * `ProductTable` needs to filter the product list based on that state (search text and checkbox value).
    * `SearchBar` needs to display that state (search text and checkbox value).
1. **Find their common parent:** The first parent component both components share is `FilterableProductTable`.
2. **Decide where the state lives**: We'll keep the filter text and checked state values in `FilterableProductTable`.

So the state values will live in `FilterableProductTable`.

Add state to the component with the [`useState()` Hook](/apis/usestate). Hooks let you "hook into" a component's [render cycle](/learn/render-and-commit). Add two state variables at the top of `FilterableProductTable` and specify the initial state of your application:

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);
```

Then, pass `filterText` and `inStockOnly` to `ProductTable` and `SearchBar` as props:

```js
<div>
  <SearchBar
    filterText={filterText}
    inStockOnly={inStockOnly} />
  <ProductTable
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

You can start seeing how your application will behave. Edit the `filterText` initial value from `useState('')` to `useState('fruit')` in the sandbox code below. You'll see both the search input text and the table update:

<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."/>
      <label>
        <input
          type="checkbox"
          checked={inStockOnly} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

In the sandbox above, `ProductTable` and `SearchBar` read the `filterText` and `inStockOnly` props to render the table, the input, and the checkbox. For example, here is how `SearchBar` populates the input value:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Search..."/>
```


Refer to the [Managing State](/learn/managing-state) to dive deeper into how React uses state and how you can organize your app with it.

## Step 5: Add inverse data flow {/*step-5-add-inverse-data-flow*/}

Currently your app renders correctly with props and state flowing down the hierarchy. But to change the state according to user input, you will need to support data flowing the other way: the form components deep in the hierarchy need to update the state in `FilterableProductTable`.

React makes this data flow explicit, but it requires a little more typing than two-way data binding. If you try to type or check the box in the example above, you'll see that React ignores your input. This is intentional. By writing `<input value={filterText} />`, you've set the `value` prop of the `input` to always be equal to the `filterText` state passed in from `FilterableProductTable`. Since `filterText` state is never set, the input never changes.

You want to make it so whenever the user changes the form inputs, the state updates to reflect those changes. The state is owned by `FilterableProductTable`, so only it can call `setFilterText` and `setInStockOnly`. To let `SearchBar` update the `FilterableProductTable`'s state, you need to pass these functions down to `SearchBar`:

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

Inside the `SearchBar`, you will add the `onChange` event handlers and set the parent state from them:

```js {5}
<input
  type="text"
  value={filterText}
  placeholder="Search..."
  onChange={(e) => onFilterTextChange(e.target.value)} />
```

Now the application fully works!

<Sandpack>

```jsx App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText} placeholder="Search..."
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

You can learn all about handling events and updating state in the [Adding Interactivity](/learn/adding-interactivity) section.

## Where to go from here {/*where-to-go-from-here*/}

This was a very brief introduction to how to think about building components and applications with React. You can [start a React project](/learn/installation) right now or [dive deeper on all the syntax](/learn/describing-the-ui) used in this tutorial.
