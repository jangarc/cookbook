---
description: VUE 3 Composition API / 可組合API
---

# Composition API

{% hint style="info" %}
本篇主要針對Vue3的Composition API做說明，如果想簡略的了解為何該使用Composition API，請參考上一篇的[Composition API介紹](https://jang-arc.gitbook.io/my-cookbook/vue3/vue3_rfcs_note#zhi-you-jie-shao-composition-api-zu-he-shi-api)
{% endhint %}

觀看本篇前需要事先了解的事情，這也是我認為最先需要了解的特性，就是在Vue3中提供的全域化API，如下例。我們可以看到，可以把Vue提供的函式解耦後直接使用。

```javascript
const { h, createApp } = Vue;

const app = createApp({
  template: `<h1 class="title">Hello Vue 3 Composition API</h1>`
});

app.mount("#app");
```

[前往了解VUE3的=&gt;全域化API說明](https://jang-arc.gitbook.io/my-cookbook/vue3/vue3_rfcs_note#quan-yu-hua-api)

## \# Composition API的基本開發環境及程式撰寫風格

這裡來我們來看一下如果你使用Composition API開發程式碼，大致的開發結構。

**\[範例 1\]** 不依賴 Vue.createApp的情況下使用Composition API

```markup
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/3.0.0-rc.5/vue.global.prod.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>

<div>
  <h1>Vue 3 Composition API + jQuery</h1>
  <h2>一個沒有this的專案開發風格，也不強制使Virtual DOM</h2>
  <p>Count: <span id="CountBlock"></span></p>
  <p>
    <button id="Increase">+</button>
    <button id="Decreasing">-</button>
  </p>
</div>

<script>

// 解耦全域API
const { reactive, watchEffect } = Vue;

// 建立一個reactivity資料
const state = reactive({
  count: 0,
});

// 建立&綁定按鈕事件
const doIncrease = () => { state.count++; };
const doDecreasing = () => { if(state.count > 0) state.count-- };
$("#Increase").click(doIncrease);
$("#Decreasing").click(doDecreasing);

// 觸發事件
const CountBlock = $("#CountBlock");

// 監控state.count，當資料變化時觸發顯示變更
watchEffect(() => CountBlock.text(state.count));

</script>
```

上例我們可以看到幾個特性：

1. 全域API的使用方式
2. 沒有強制使用Virtual DOM\(沒有Vue.createApp\(\).mount過程\)
3. 可以使用watchEffect來補捉資料的變更
4. 沒有this

**\[範例 2\]** 使用 Vue.creaetApp.mount\(\) + HTML template

```markup
<div id="app">
  <h1>Vue 3 Composition API</h1>
  <h2>一個沒有this的專案開發風格</h2>
  <p>Count: {{state.count}}</p>
  <p>
    <button v-on:click="increase">+</button>
    <button v-on:click="decreasing">-</button>
  </p>
</div>

<script>

  // 解耦全域API
  const { reactive, createApp } = Vue;

  const app = createApp({
    setup(){

      // 建立一個reactivity資料
      const state = reactive({
        count: 0,
      });

      // 建立&綁定按鈕事件
      const increase = () => { state.count++; };
      const decreasing = () => { if(state.count > 0) state.count-- };

      return {
        // 資料
        state,
        // 事件
        increase, decreasing
      }

    }
  });

  app.mount("#app");

</script>
```

上例和第一個例子的差異，可以看到是在沒有使用watch或watchEffect做資料監控，使用setup做為人口函式，將監控的資料return給Vue使用。

**\[範例 3\]** 使用 Vue.creaetApp.mount\(\) + return render function

```markup
<div id="app"></div>

<script>

  // 解耦全域API
  const { h, reactive, createApp } = Vue;

  const app = createApp({
    setup(){

      // 建立一個reactivity資料
      const state = reactive({
        count: 0,
      });

      // 建立&綁定按鈕事件
      const increase = () => { state.count++; };
      const decreasing = () => { if(state.count > 0) state.count-- };

      // [注意] 這裡是一個函式
      return () => [
        h("h1", "Vue 3 Composition API"),
        h("h2", "一個沒有this的專案開發風格"),
        h("p", ["Count: ", state.count]),
        h("p", [
          h("button", {onClick: increase}, "+"),
          h("button", {onClick: decreasing}, "-"),
        ]),
      ];

    }
  });

  app.mount("#app");

</script>
```

以上三個例子結果都是等價的，接下來讓我們看一下一些常用的API。

## \# 資料類API

### + reactive API

**\[功能\]** 用來建立俱反應性資料，可以配合watchEffect和watch做監控，當資料變更時可以補捉並進行事件觸發。 **\[允許資料格式\]** Object/Array

```javascript
const { reactive } = Vue;

const state = reactive({ count: 0 });

const increase = () => { state.count++; };
const decreasing = () => { if(state.count > 0) state.count-- };
```

### + ref API

**\[功能\]** 功能和reactive相同，不同的是ref可以針對Boolean/Number/String等非object資料進行監控。 **\[允許資料格式\]** Boolean/Number/String

```javascript
const { ref } = Vue;

const count = ref(0);

const increase = () => { count.value++; };
const decreasing = () => { if(count.value > 0) count.value-- };
```

### + computed API

**\[功能\]** 就像Vue2中的option computed用途相同

```javascript
const { reactive, computed } = Vue;

const userData = reactive({
  firstName: "arc",
  lastName: "jang",
  fullName: computed(()=> firstName + ' ' + lastName)
});
```

同樣支援get、set的方式

```javascript
const { ref, computed } = Vue;

const name = ref("arc");

const upperName = computed({
  get(){
    return name.value.toUpperCase();
  },
  set(value){
    name.value = value;
  }
})
```

### + readonly API

**\[功能\]** 用來引用指定reactive或ref資料並產生一個只讀資料

```javascript
const { reactive, readonly } = Vue;

const state = reactive({ count: 0 });
const readonlyState = readonly(state);

readonlyState.count++; // 這將會觸發一個錯誤因為資料唯讀
```

### + watch / watchEffect API

**\[功能\]** 可以針對已建立的反應性資料\(reactive, ref\)進行監控，讓開發人員在沒有綁定Vue.createApp的情況下仍然擁有監控資料的能力。

**\[使用方式\]**

1. For reactive: watch\(`Function`\) 
2. For ref: watch\(`variable name`, `Function`\) 
3. For ref: watch\(`variable name Array`, `Function`\) 

下例為 reactive + watchEffect

```javascript
const { reactive, watchEffect } = Vue;

// 定對俱反應性資料
const state = reactive({ count: 0 });

// 建立資料變更函式
const increase = () => { state.count++; };
const decreasing = () => { if(state.count > 0) state.count-- };

// 建立事件監控
let stopWatch = watchEffect( ()=> console.log(state.count) );

increase(); // 產生Log訊息

stopWatch(); // 停止監控

increase(); // => 不會再被觸發
```

## \# 生命週期Hook API

{% hint style="info" %}
生命週期Hook只能使用在Vue Option Hook中，例如: setup
{% endhint %}

功能就像是Vue2 option裡的 baforeCreate, created, beforeMount, mounted但由於生命週期基於Vue，所以必需綁定在Vue的生命週期過程中，所以在Composition API的開發過程中則需綁定在，option setup中。

**\[使用方式\]** onBeforeMount\(`Function`\)

在Vue3轉換成全域API如下：

* onBeforeMount
* onMounted
* onBeforeUpdate
* onUpdated
* onBeforeUnmount
* onUnmounted
* onErrorCaptured
* onRenderTracked**\[新增\]**
* onRenderTriggered**\[新增\]**

```javascript
const { createApp, reactive, watchEffect, onMounted } = Vue;

createApp({
  setup(){

    // 定對俱反應性資料
    const state = reactive({ x: 0, y: 0 });

    // 監聽滑鼠事件
    function startListenerMouseXY(){
      window.addEventListener("mousemove", (event)=>{
        state.x = event.clientX;
        state.y = event.clientY;
      });
    }

    // 當資料變更時進行變更
    watchEffect( () => document.body.innerHTML = `X:${state.x} / Y: ${state.y}` );

    // 建立資料變更函式
    let stopListenerMouseXY;

    // 當mounted時掛載滑鼠監聽
    onMounted( ()=> stopListenerMouseXY = startListenerMouseXY() );

    // 當unmounted時解除滑鼠監聽
    onUnmounted( () => stopListenerMouseXY() );

  }
})
.mount("body")
```

## \# 資料注入API

在Vue3中提供了全域的資料應用API`provide/inject`，應用方式是在指定模組中使用provide提供接口，當有其它的component需要使用時，只要使用inject就可以從中提取資料。

**\[使用方式\]**

1. provide\(`String:Prop Name`, `Data:Any`\)
2. inject\(`String:Prop Name`, `Default Data`\)

```markup
<div id="app">
  <example></example>
</div>

<script>
  const { h, createApp, reactive, provide, inject } = Vue;

  createApp({
    components: {
      example: {
        setup(){
          const state = inject("state", {});
          return () => h("h1", null, ["Count: ", state.count]);
        }
      }
    },
    setup(){
      const state = reactive({ count: 0 });
      provide("state", state);
    }
  })
  .mount("#app")
</script>
```

## \# template

在Vue中獲得template的方式如下：

* 基本的綁定方式使用HTML Template

```markup
<div id="app">
  <h1>{{title}}</h1>
</div>

<script>

const { createApp, ref } = Vue;

createApp({
  setup(){

    let title = ref("Vue 3 Composition API");

    return {
      title
    };
  }
})
.mount("#app")

</script>
`
```

* 和HTML template相同的方式可以使用option template，下例與上例基本上是等價的。

```markup
<div id="app"></div>

<script>

const { createApp, ref } = Vue;

createApp({
  template: `<h1>{{title}}</h1>`,
  setup(){

    let title = ref("Vue 3 Composition API");

    return {
      title
    };
  }
})
.mount("#app")

</script>
`
```

使用以上這種方式雖然方便，但在進階應用上其實少了某些應用彈性。如下例：

```markup
<div id="app">
  <h1>Books</h1>
  <h2>Art book</h2>
  <ul>
    <li v-for="book in artItems">{{book.name}}</li>
  </ul>
  <h2>Design book</h2>
  <ul>
    <li v-for="book in designItems">{{book.name}}</li>
  </ul>
</div>
```

我們可以看到在art和design的清單上基本都是同一個邏輯清單，但在template方解決方式內時，我們只能使用component的方式或多重迴圈的方式來解決。從這個點上我們可能會希望那個重覆區塊是一個可重用區塊，並在我們希望能擴充功能時，只要更改單一區域就能同時讓多個顯示區域都得到更新。說了這麼多我們來看一下另一種template選擇。

* 使用render function

```javascript
const { h, createApp, reactive } = Vue;

createApp({
  setup(){

    let state = reactive({
      artItems: [/* ... */],
      designItems: [/* ... */],
    });

    // 抽離部份顯示邏輯
    let renderBookItems = items => h(
      "ul", 
      null, 
      items.map( book => h("li", null, [book.name]))
    );

    return () =>  [
      h("h1", "Books"),
      renderBookItems(state.artItems), // 重用
      h("h2", "Art book"),
      h("h2", "Design book"),
      renderBookItems(state.designItems), // 重用
    ];

  }
})
.mount("#app");
```

看完了上例，我們可以發現重用性的問題解決了，但似乎又衍伸了另一個煩人的問題，render function的使用方式用來開發較複雜旳專案時，將會變的難以維護！於是乎我最後的選擇就是能保有html template的簡略同時也能保有render function的彈性的選擇jsx。

* 使用jsx

```javascript
const { h, createApp, reactive } = Vue;

createApp({
  setup(){

    let state = reactive({
      artItems: [/* ... */],
      designItems: [/* ... */],
    });

    let renderBookItems = items => <ul>
      { items.map( book => <li>{item.name}</li> ) }
    </ul>;

    return () => [
      <h1>Books</h1>
      renderBookItems(state.artItems),
      <h2>Art book</h2>,
      <h2>Design book</h2>,
      renderBookItems(state.designItems),
    ];

  }
})
.mount("#app");
```

最終我們可以看到以上程式碼，不僅擁有邏輯拆分的彈性也能保有HTML節點式的開發方式，一切似乎變的更好了。

## \# component

在了解了上篇的template應用後，不管邏輯如何拆分為了更好的分類管理以重用，我們可能需要把component化管理，讓component可以沿伸應用再不同專案中。

### + 使用resolveComponent使用全域component

使用該函式可以確保引用到己註冊compnent

```javascript
const { h, createApp, resolveComponent } = Vue;

const app = createApp({
  setup(){
    const header = resolveComponent("header");
    return h(header);
  }
});

// 註冊全域component
app.component("header", {template: `<h1>Example</h1>`})

app.mount("#app");
```

### + functional component

```javascript
const { h, createApp, resolveComponent } = Vue;

const app = createApp({
  setup(props, ){

    const state = {
      title: "Example",
      bestBooks: [
        {name:"book1"}, 
        {name:"book2"}
      ],
    };

    const header = resolveComponent("header");
    const books = resolveComponent("books");

    return () => [
      h(header, {title: state.title}),
      h(books, {items: state.bestBooks})
    ];

  }
});

// functional component
const headerComp = (props) => h("h1", null, props.title);
headerComp.props = ["title"];

const bookItemsComp = (props) => h("ul", null, props.items.map( book => h("li", null, book.name) ) ) ;
bookItemsComp.props = ["items"];

app.component("header", headerComp);
app.component("books", bookItemsComp);

app.mount("#app");
```

## \# 程式碼的拆分與組合概念

**待撰寫**

撰寫備註：

1. 程式撰寫風格的選擇，一定需要改變嗎？改變帶來了什麼？
2. 拆分與組合程式碼，那些概念能為你帶來效益
3. 實例分析

