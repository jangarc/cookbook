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

## \# 基本開發環境及程式撰寫風格

這裡來我們來看一下如果你使用Composition API開發程式碼，大致的開發結構。

### + 不綁定Vue使用Composition API

**\[ 範例 1 \]** 不依賴 Vue.createApp的情況下使用Composition API

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

### + 使用Option setup的結構

**\[ 範例 2 \]** 使用 Vue.creaetApp.mount\(\) + HTML template

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

上例和第一個例子的差異，可以看到是在沒有使用watch或watchEffect做資料監控，使用setup做為人口函式，將監控的資料return給Vue使用，事件觸發的部份在mount時已自動完成綁定了。

### + 在setup裡使用tamplate

**\[ 範例 3 \]** 使用 Vue.creaetApp.mount\(\) + return render function

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

**\[ 功能 \]** 用來建立俱反應性資料，可以配合watchEffect和watch做監控，當資料變更時可以補捉並進行事件觸發。 **\[ 允許資料格式 \]** Object/Array

```javascript
const { reactive } = Vue;

const state = reactive({ count: 0 });

const increase = () => { state.count++; };
const decreasing = () => { if(state.count > 0) state.count-- };
```

### + ref API

**\[ 功能 \]** 功能和reactive相同，不同的是ref可以針對Boolean/Number/String等非object資料進行監控。 **\[ 允許資料格式 \]** Boolean/Number/String

```javascript
const { ref } = Vue;

const count = ref(0);

const increase = () => { count.value++; };
const decreasing = () => { if(count.value > 0) count.value-- };
```

### + computed API

**\[ 功能 \]** 就像Vue2中的option computed用途相同

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

**\[ 功能 \]** 用來引用指定reactive或ref資料並產生一個只讀資料

```javascript
const { reactive, readonly } = Vue;

const state = reactive({ count: 0 });
const readonlyState = readonly(state);

readonlyState.count++; // 這將會觸發一個錯誤因為資料唯讀
```

### + watch / watchEffect API

**\[ 功能 \]** 可以針對已建立的反應性資料\(reactive, ref\)進行監控，讓開發人員在沒有綁定Vue.createApp的情況下仍然擁有監控資料的能力。

**\[ 使用方式 \]**

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
生命週期Hook只能使用在Vue Option Hook中，例如: setup, beforeMount, mounted ....
{% endhint %}

功能就像是Vue2 option裡的生命週期Hook，且因為生命週期基於Vue，所以必需綁定在Vue的生命週期過程中，所以在Composition API的開發過程中則建議綁定在，option setup中。

**\[ 使用方式 \]** onBeforeMount\(`Function`\)

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

**\[ 使用方式 \]**

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

在Vue中使用template的方式大致上分為下列：

### + 基本的綁定方式使用HTML Template

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

### + 使用option template

和HTML template相同的方式可以使用option template，下例與上例基本上是等價的。

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

### + 純文字template存在的缺陷

使用以上這種方式雖然方便，但在進階應用上其實少了某些應用彈性。

1. 不適合在複雜或較大的專案裡直接使用。
2. 比較不好進行後續維護，例如事後需要拆分部份邏輯成component。

如下例：

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

### + 使用render function

**\[概念\]**不管使用HTML template或option template其實都是，將其取得的純文字內容的部份，再將其內容compile成render function再進行render。

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

### + 使用jsx

不用害怕jsx，因為它就是一個基於html的模版語言。

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
      <h1>Books</h1>,
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

使用resolveComponent函式可以確保引用到己註冊compnent

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

**撰寫中...**

撰寫備註：

* [x] 程式撰寫風格的選擇，一定需要改變嗎？改變帶來了什麼？
* [x] 拆分與組合程式碼，那些概念能帶來效益
* [ ] 實例分析

本篇大部份都會是我自己的想法，實際應用還是得要依需求評估。

### + 程式撰寫風格的選擇

以下是一段廢文可以跳過不用理會，但是如果你覺的事情老是做不完時可以花1分鐘看一下

在我剛開始大量吸收程式開發風格的概念時，面對各種的理論及結構，總是會讓我面臨無法決擇冏境。當我決定了某一種開發模式及規範後開始進行專案的開發後，其它風格或概念什麼都似乎沒有那麼重要了，至少我可以開始解決問題了。當然這個想法也是天真的，當我用固定的模式開發了一陣子後，很快就遇到了管理與維護的問題，當然如果只有一個人作業時我可以不顧一切的向下撰寫程式，不需要重視邏輯抽離的事情，但其實這樣子只會讓自己停滯不前，因為我發理我老是在做類似的事情而且我常因為總是處理同樣的事情時而延誤了其它真正需要處理的事情，我相信沒有人喜歡這樣子，但是大部份的人可能都會覺的事情太多了所以自己沒辦法停下來思考。經歷了幾年的時間浪費後，我發覺我只能選擇逃避自己寫過的程式碼或是花二倍或以上的時間去了解我之前做了些什麼，不然只能面對直接進行重構這件事，似乎之前的努力都是一張白紙。

**\[ 廢話結束！開始我們的正題 \]**

在Vue2中只能使用option的開發方式，分別把不同範疇的程式碼按照分類放在不用的option裡，Vue3可以做到一樣的事情，但也提供了不同的（選擇）方式。我們直接看下例：

**\[ 在Vue2.x中我們只能這麼使用 \]**

```javascript
// in vue2.x
const app = new Vue({
  el: "#app",
  template: `<div>...</div>`;
  data: {},
  methods: {},
  computed: {},
  watch: {},
  created(){},
  mounted(){},
});
```

**\[ Vue2.x直接轉Vue3.x我們可以這麼做 \]**

```javascript
// in vue3.x option
createApp({ // ［差異］
  template: `<div>...</div>`;
  data(){}, // ［差異］
  methods: {},
  computed: {},
  watch: {},
  created(){},
  mounted(){},
})
.mount("#app"); // ［差異］
```

**\[ 使用Composition API建議的模式 \]**

```javascript
// in vue3.x composition
createApp({
  setup(){
    const state = reactive({}); // data
    function doSomething(){} // method
    let variable = computed(state.any); // computed
    watch(()=> {}); // watch
    onBeforeMounted(()=>{});// lifecycle hook
    // template
    return () => h(customComp, { /* option... */, /* childer... */})
  }
})
.mount("#app");
```

**\[ Composition的進階方式 \]**我們可以從下例看到，拆分邏輯後再重用的例子，二個專案使用共同的依賴包。

```javascript
// in vue3.x composition

// project A => init.js ======================================
import { counterModel } from "./model"
import { counterControll } from "./controll"
import { CounterView } from "./view"

return default createApp({
  setup(){
    const data = counterModel();
    const { increase, decreasing } = counterControll(data);
    return CounterView( data, { increase, decreasing } );
  }
})

// project B => init.js ======================================
import { CounterModel, LogModel } from "./model"
import { counterControll, LogControll } from "./controll"
import { CounterView, LogView } from "./view"

return default createApp({
  setup(){
    const count = CounterModel();
    const logItems = LogModel();
    const { addLog, generateLog } = LogControll(logItems);
    const { increase, decreasing } = counterControll(
      count,
      {
        onCountChange(changeInfo){
          let info = `do ${changeInfo.type} count to ${changeInfo.value}`;
          addLog( generateLog(info) )
        }
      }
    );
    const logView = LogView( logItems, "Count Log" );
    return CounterView(
      count,
      { increase, decreasing },
      logView
    );
  }
})
```

我將會在 **\[ 拆分與組合程式碼 \]** 章節實踐上例

{% hint style="info" %}
在開發的過程中，並不需要一定得使用那一種開發方式來進行，因為我們都知道程式開發並不存在『銀彈』，而是要針對專案的各種狀況，進而評估使用那種方式最適合用來開發目前的專案。
{% endhint %}

### + 拆分與組合程式碼

在javascript中存在著各類能供我們拆分與組合工具函式，本節最主要就是針對這些方法做說明及示例。

**\[ Functional programming \| 函數式編程 \]**

透過函數式編程來拆分或組合邏輯，我們先看一下未拆分的範例：

```javascript
createApp({
  setup(){
    // 建立計數器初始資料
    const count = ref(0);

    function increase(){
      count.value++;
    }

    function decreasing(){
      if(count.value > 0) count.value--;
    }

    return () => h("div", null, [
      h("h1", null, [ "Count: ", count.value ]),
      h("p", null, [
        h("button", { onClick: increase }, "+"),
        h("button", { onClick: decreasing }, "-")
      ])
    ])
  }
})
```

接下來我將逐一拆分，最後重組上例：

**1.** **\[拆分\]**資料邏輯

```javascript
// model.js
// 每次調用時都取得我們定制好的資料邏輯概念等同model
function countModel(){
  const count = ref(0);
  return count;
}
```

**2.** **\[拆分\]**資料控制邏輯

```javascript
// controll.js
// 將用到的函式
function counterControll(count){
  function increase(){
    count.value++;
  }

  function decreasing(){
    if(count.value > 0) count.value--;
  }

  return { increase, decreasing }
}
```

**3.** **\[拆分\]**顯示邏輯

```javascript
// view.js
function CounterView(count, { increase, decreasing }){
  return () => h("div", null, [
    h("h1", null, [ "Count: ", count.value ]),
    h("p", null, [
      h("button", { onClick: increase }, "+"),
      h("button", { onClick: decreasing }, "-")
    ])
  ]);
}
```

**4.** **\[組合\]**使用已拆分的模組重新組合

該例我們使用browser的引用方式來，還原了上一個章節的第3個範例

```markup
<!--
import { counterModel } from "./model"
import { counterControll } from "./controll"
import { CounterView } from "./view"
 -->
<script src="./model.js"></script>
<script src="./controll.js"></script>
<script src="./view.js"></script>

<script>
createApp({
  setup(){
    const count = counterModel();
    const { increase, decreasing } = counterControll(data);
    return CounterView( data, { increase, decreasing } );
  }
}).mount("#app")
</script>
```

最後我們成功的將使用邏輯拆分成獨立的js檔，也有助於之後如果需要重用時，能直接拿來套用或升級。

**\[ Memoization \| 記憶函數快取 \]**

上節我們提到利用函數式編程來拆分程式區塊，當大量採用函數時就會有可能會遇到計算函數或異步函數等較花費效能與時間函數。這時為了提升效能，我們可能我們會針對某一些性值的函數進行快取，目的是為了讓某些函數在帶入指定的值進行計算或請求時，能暫時保存固定的請求結果以增進程式效能。

下面我們引用mem.js的範例來做說明：

```javascript
// 假設有一個mem.js的Memoization方法

function add( a, b ){
  return a + b
}

const memAdd = mem(increase);

memAdd(1, 1); // 第一次使用 => 2
memAdd(1, 1); // 第二次使用 => 將會由快取返回結果 2
```

給快取指定有效期限

```javascript
// 當我們給快取一個週期時
const memAdd = mem(incerease, 2000); // 二秒後過期

memAdd(1, 1); // 第一次使用 => 2
memAdd(1, 1); // 第二次使用 => 將會由快取返回結果 2
setTimeout(() => memAdd(1, 1), 3000); // 由於已超過生命週期2秒所以這裡返回的結果是重新計算的
```

如果使用了Memoization方法就可以有效的控制不必要的執行，以減少效能的開銷。

**\[ Promise \| 異步編程 \]**

假設我們有一個Blog APP我們希望它在取得user和user的Blog文章後再開始APP.mount，當使用非promise概念請求資料時，例如jQuery.ajax+callback來接續邏輯時，看起來可能如下例：

```javascript
function getUser(id, callback){
  $.ajax({
    url: "https://example.conm/user/"+id,
    dataType: "json",
    success(result){
      if(typeof callback==="function") callback(result)
    },
    error(err){
      console.log(err)
    }
  })
}

function getPostsByUser(userid, callback){
  $.ajax({
    url: "https://example.conm/posts/"+userid,
    dataType: "json",
    success(result){
      if(typeof callback==="function") callback(result)
    },
    error(err){
      console.log(err)
    }
  })
}

function init(afterInit){
  var id = 1;
  var user = null;
  var posts = null;

  getUser(id, userData => {
    user = userData;
    getPostsByUser(user.id, postsData => {
      posts = postsData;
      // 如果還需要N個回調時
      /*
      getNext1(req, result => {
        getNext2(req, result => {
          getNext3(req, result => {
            getNext4(req, result => {
              // ....Next(N)
            })
          })
        })
      })
      */
      afterInit({user, posts}); // 開始mount
    });
  });
}
```

接下來我們來看一下使用Promise的方式來進行開發

```javascript
function getUser(id, user){
  return axios.get("https://example.conm/id/"+id)
}

function getPosts(userId){
  return axios.get("https://example.conm/posts/"+userId)
}

function init(afterInit){
  var id = 1;
  var user = null;
  var posts = null;

  getUser(id)
    .then(userData => {
      user = userData;
      getPosts(user.id);
    })
    .then( postsData => {
      posts = postsData;
      afterInit({user, posts})
    })
    /*
    // 如果還有N個事件時
    .then(()=>{ N1... })
    .then(()=>{ N2... })
    .then(()=>{ N3... })
    */
    ;
}
```

比較上面二個案例，我們看到使用Promise能避免一直一層一層的進行回調，而是完成某個任務後再向下進行下一個任務。不僅如此Promise更能有效減少錯誤補捉邏輯對程式閱讀上的影響，如下例：

```javascript
// ...沿續上例我們加上錯誤補捉
getUser(id)
    .then(userData => {
      user = userData;
      getPosts(user.id);
    })
    .then( postsData => {
      posts = postsData;
      afterInit({user, posts})
    })
    // 只要設定一組補捉, 就可以同時管理 getUser, getPosts, next(N)層的錯誤了
    .catch(console.error)
    // 如果你有需要執行不管結果成功或失誤時都必需觸發的事件
    .finally(finallyFn)
```

**\[ async/await \| 異步編程 \]**

使用promise後我們已可以大大的提昇程式可讀性及可維護性了，但似乎有更好的選擇。讓我們用async和await改善上例init部份：

```javascript
async function init(afterInit){
  try
  {
    var id = 1;
    var user = await getUser(id);
    var posts = await getPosts(user.id);
  }
  catch(e){
    console.error(e);
  }
}
```

### + 實例分析

**\[待撰寫\]**

