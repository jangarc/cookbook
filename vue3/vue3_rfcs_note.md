---
description: 本篇主要是我讀完Vue3 RFCS全部項目後做的筆記
---

# Vue 3 RFCS Note

Vue3本次變更就是依照該份文件變改，本份文件就只針對Vue3的標準功能部份的變更做簡略說明及示例，如果想了解每個RFC更多的設計原理，可以直接前往 [Github參考vuejs/rfcs](https://github.com/vuejs/rfcs)

> **\[注意\]** 由於RFCS清單內的流水號順序，並不是很適合想從Vue2升Vue3的使用者觀看，所以我會依應用順序逐一解說**（會關聯對應的RFCS NO.供連結）**，如果有較大型的新概念或變更將會抽離到獨立的文章上做介紹。

## \# 官方Beta版手冊已出來\(只有英文版\)

* [Vuv3手冊beta版首頁](https://v3.vuejs.org/guide/installation.html)
* [Vue2轉Vue3](https://v3.vuejs.org/guide/migration/introduction.html)
* [Composition API中文手冊](https://composition-api.vuejs.org/zh/)

## \# VUE 3 相關範例

{% hint style="info" %}
請參照 [\[ jangarc.github.io \]](https://jangarc.github.io/index.html) 
{% endhint %}

## \#  全域化API

該變化最主要是分離了一些內部API到全域上，而且還很貼心了兼容了解耦引用，也就是說全部被公開的API都不再依賴this，我覺的這也是為了composition api（可組合API）做優化，同時也提升了開發的可選擇性，該變更帶來的好處就是在開發打包時我們沒有引用的包，將可以不被打包在專案裡。

```javascript
// 直接使用
Vue.createApp({/*...*/})

// ［全域化］解耦後使用
const {createApp} = Vue;
createApp({/*...*/})
```

\[ RFCS.[0004.global api treeshaking](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0004-global-api-treeshaking.md) \]

> **這裡也附上該項目造成使用方式上的變更範例**

* Vue專案的建立\(create\)與安裝\(mount\)

```javascript
// in Vue 2.x ========================================

Vue.use(/*...*/);
Vue.mixin(/*...*/);
Vue.component(/*...*/);
Vue.directive(/*...*/);

// => 接續使用
new Vue({el: "#app"});

// => 分段使用
const oapp = new Vue({/*...*/});
oapp.$mount("#app");

// in Vue 3.x ========================================

const { createApp } = Vue;

// => 接續使用
createApp({/*...*/}).mount("#app");

// => 分段使用
const app = createApp({/*...*/});
app.use(/*...*/);
app.mixin(/*...*/);
app.component(/*...*/);
app.directive(/*...*/);
app.mount("#app");
```

* 全域的渲染函式 h\(\)

```javascript
const { h, createApp } = Vue;

// 建立component
const ctitle = {
  render(){
    // 這裡的h來自全域
    return h("h1", null, [this.title])
  }
};

const app = createApp({
  components: {
    ctitle
  }
});

app.mount("#app");
```

* 不喜歡this的人改用Composition API，官方已有完整的[Composition API中文手冊](https://composition-api.vuejs.org/zh/)

```javascript
const { h, createApp, reactive, ref, watchEffect } = Vue;

createApp({
  setup(props, {emit}){

    // 在setup裡不用this了
    let state = reactive({});
    let title = ref(""); // 建立data

    watchEffect(title, ()=> console.log(title.value));

    // [注意] 使用setup()時return的是function，使用render()時return的是VNode
    return () => h("div", null, [
      h("h", null, title.value)
    ]);

  }
})
.mount("#app")
;
```

## \# data不再允許是物件

Vue2中建立App時data可以是object也可以是function，但是在Vue3中建立App時data必需函式，不然將會出現錯誤訊息

```javascript
// in Vue 2.x
new Vue({
  data:{/*...*/}
});

// in Vue 3.x
const app = createApp({
  data(){
    return {/*...*/}
  }
});
```

\[ RFCS.[00019.remove data object declaration](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0019-remove-data-object-declaration.md) \]

## \# 全域註冊API使用方式變更

原本在Vue2時使用的全域註冊API**\(use，mixin，component，directive\)**，可以使用Vue.use\(...\)直接引用 在Vue3中必需先使用var app = createApp\(...\)建立一個APP後再引用

```javascript
// in Vue 2.x
Vue.config.ignoredElements = [/^app-/]; // 建立內建元素白名單
Vue.use(/*...*/); // 使用plugin
Vue.mixin(/*...*/); // options mixin
Vue.component(/*...*/); // 註冊全域component
Vue.directive(/*...*/); // 註冊自定義指令

Vue.prototype.customProperty = () => {}; // 建立全域變數

// in Vue 3.x
const app = createApp(/*...*/);
app.config.isCustomElement = tag => tag.startsWith('app-');
app.use(/*...*/);
app.mixin(/*...*/);
app.component(/*...*/);
app.directive(/*...*/);

app.config.globalProperties.customProperty = () => {};
```

\[ RFCS.[0009.global api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0009-global-api-change.md) \]

## \# v-model功能變更

在Vue3裡使用v-model替換原本的v-bind:\[key\].sync功能，這個改變實在太棒了，這樣的改變可以在開發時更直覺更便利，由其是在開發component時最常用到的也就是APP和component之間的資料綁定，由於Vue是基於資料變更來觸發事件發生的，而且在Vue2裡大部份的初學者最搞不懂的，通常都是為什麼資料在組件中變化時_有時候_不會觸發變更？

主要原因當然是VUE禁止組件裡的資料直接影響APP本身，所以給組件提供了emit的功能用來通知APP事件發生了，在Vue3中更是強化了這個規範，使用Vue3時當你試著直接變更prop時你將會收到一個警告訊息，讓我們逐一的看一下以下範例：

* 取代v-bind:datakey.sync

\[ RFCS.[0005.replace v-bind sync with v-model argument](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0005-replace-v-bind-sync-with-v-model-argument.md) \]

```markup
<!-- in Vue 2.x -->
<posthead v-bind:title.sync="postTitle"></posthead>

<!-- in Vue 3.x -->
<posthead v-model:title="postTitle"></posthead>
```

=&gt; 回顧一下上例的解釋方式

```markup
<posthead v-bind:title="postTitle" v-on:update:title="event => title=event"></posthead>
```

=&gt; 回顧一下在component裡怎麼調到update:title這個事件

```javascript
const posthead = {
  template: `
  <div>
    <input :value="title" @input="onUpdateTitle">
  </div>
  `,
  props: {
    title: String
  },
  methods: {
    onUpdateTitle(event){

      // 把變更資料推送回APP綁事的事件update:title，由APP來變更資料已達到事件觸發條件目標
      this.$emit("update:title", event.target.value);

      // [注意]以下這一個宣告在Vue2裡不會報錯且可以變更posthead裡的title值
      // 但是在Vue3裡是會出現警告的
      // 該句語法事實上是沒有任何意義的，因為當APP的postTitle變更時，posthead裡的prop.title也會跟改變，
      this.title = event.target.value;

    },
  }
};
```

* 保留原本的特性，所以以下方法仍然是有效的

```markup
<!-- Vue2 & Vue3 通用 -->
<input v-model="postTitle" />
```

\[ RFCS.[0011.v-model api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0011-v-model-api-change.md) \]

## \# 動態參數

可以在v-bind/v-on/v-slot宣告時使用動態參數，這會看起來像 **v-bind:\[dynamic-data-key\]** **v-on:\[dynamic-hendle\]** **v-slot:\[dynamic-slot-name\]**

使用結果如下例，可以將鍵值做動態綁定，當綁定的值變更時一起變更綁定的內容\(v:bind資料鍵值,v-on:觸發事件,v-slot:插槽區域\)：

```markup
  <script>

    Vue.createApp({
      // ...省略
      data(){
        return {
          // ...省略
          valuekey: "msg1",
          eventkey: "focus",
          slotkey: "head"
        }
      },
      // ...省略
    })
  <script>

  <input v-bind:[valuekey]="message" v-on:[eventkey]="pushLog" />

  <example title="Dynamic slot logs" v-bind:logs="logs">
    <template v-slot:[slotkey]></template>
  </example>
```

\[ RFCS.[003.dynamic directive arguments](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0003-dynamic-directive-arguments.md) \]

## \# 取消了v-on的keycode支援

由於原生事件**KeyboardEvent.keyCode已被棄用**所以在Vue3中也將停止支援，包括config中的keyCodes也一同失效。

**\[注意\]** 這裡除了keycode部份被移除，像其它.prevent/enter/ctrl/alt...等的修飾符是有被保留的

```markup
<!-- in Vue 2.x-->
<input @keyup:13="doSubmit">

<!-- in Vue 3.x -->
<input @keyup:enter="doSubmit">
```

\[ RFCS.[0014.drop keycode support](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0014-drop-keycode-support.md) \]

## \# 棄用inline-template

老實說這個功能我真的沒用過....，不過了解這個功能的目的後，也確實認為它不應該在存在。就算是Vue2的官網也幾乎沒有這個功能的Example，直接使用下列範例來說明如何有效的取代。

* 使用x-template

```markup
<script type="text/x-template" id="ComponentTemplate">
  <div>
    {{ hello }}
  </div>
</script>

<script>
  const hello = {
    template: "#ComponentTemplate"
  }
</script>
```

* 使用slot

```markup
<hello v-slot="state">
  Hello {{state.name}}
</hello>

<script>

// hello component
const hello = {
  template: `
  <div>
    <slot v-bind="state">Hello</slot>
  </div>
  `,
  props: {
    state: Object
  }
}

// 以下省略...

</script>
```

\[ RFCS.[0016.remove inline templates](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0016-remove-inline-templates.md) \]

## \# 整合並統一slots概念

這個特性在Vue2.6版開就被變引用了，原由是因為原本有$slots和$scopedSlots這二種用法，但事實上對Vue內部來說實現方式都是雷同的，所以在這版將插槽的概念精簡統一，讓剛入門的新手對slots的概念更清晰，本項目沒有範例簡單的說就是：使用v-slot取代slot和slot-scope。

\[ RFCS.[0006.slots unification](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0006-slots-unification.md) \]

## \# 新的slot用法

基本上的變更是使用v-slot取代掉原本的slot-scope，如果一有使用Vue2的使用者會知道，這個規則早就在Vue2.6.0時早就被採用了。

**&gt; 基本的使用方式**

假設我們導入了以下component，該元件裡提供了一個title-before的slot。[範例連結](/Vue3Example/vslot.html)

```javascript
const example = {
  template: `
  <div>
    <h1>
      <slot name="title-before" v-bind="logs"></slot>
      {{title}}
    </h1>
    <ul v-if="logs && logs.length > 0">
      <li v-for="log in logs">{{log}}</li>
    </ul>
  </div>
  `，
  props:{
    title: String，
    logs: {
      type: Array，
      default: []
    }
  }
};
```

**+ 以下為使用示例**

```markup
<!-- 1. 直接使用 -->
<example title="User login log"></example>

<!-- 2. 使用v-slot 在title前加上#號 -->
<example title="User login log">
  <template v-slot:title-before># </template>
</example>

<!-- 3. 使用導入資料 -->
<example title="User login log">
  <template v-slot:title-before="logs">({{log.length}}) </template>
</example>
```

\[ RFCS.[001.new slot syntax](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md) \]

## \# 增加了v-slot縮寫

v-slot添加了縮寫符\(\#\)

```markup
<!-- 使用縮寫 -->
<example title="User login log">
  <template #title-before="logs">({{log.length}}) </template>
</example>
```

\[ RFCS.[0002.slot syntax shorthand](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0002-slot-syntax-shorthand.md) \]

## \# 棄用$on, $off, $once等Event Emitter

取消了事件觸發函式`$on, $off, $once`，並建議有需要的人改用[mitt](https://github.com/developit/mitt)，這個改變只是去除了獨立宣告的事件，並不影響v-on喔，小心不要被混淆了，我們直接使用範例來比對一下差異。

```javascript
// in Vue 2.x
const app = new Vue({/*...*/});

// 監聽事件
app.$on("change:msg", msg => console.log(msg));

// 觸發事件
app.$emit("change:msg", "Hello~");

// 取消監控事件
app.$off("change:msg");

// in Vue 3.x

const app = createApp({/*...*/});

// app 將不存在 $on/$off/$once等函式
console.log(typeof app.$on); // => undefined
console.log(typeof app.$off); // => undefined
console.log(typeof app.$once); // => undefined
```

* 而v-on的作用最主要在於component與APP之間的相事件傳遞，所以還是不可被捨棄的

> 下例我們可以看到APP與組件間事件的傳遞過程： App.msg 綁定到 change-test.text =&gt;&gt; change-test.$refs.input在input事件時觸發並傳送input.value給App綁定的update:value事件， =&gt;&gt; app.v-on：update:value收到了來自change-test的$emit觸發資料 =&gt;&gt; 最後由app.v-on：update:value綁定的事件onUpdateValue取得收到的input.value，並完成資料的變更，最終經由變更APP的資料完成了事件連動反應\(reactiv\)

```markup
<h1>{{msg}}</h1>
<change-text :text="msg" v-on:update:value="onUpdateValue"></change-text>

<script>

Vue.createApp({
  components: {
    "change-text": {
      template: `
      <input ref="input" :value="text" @input="$emit('update:value', event.target.value)" />
      `,
      props: ['text'],
    }
  },
  data(){
    return {
      msg: "Change Text Example",
    }
  },
  methods: {
    onUpdateValue(value){
      this.msg = value;
    }
  }
})

</script>
```

\[ RFCS.[0020.events api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0020-events-api-change.md) \]

## \# attr使用fallthrough方式進行識別

在該項說明前我們先來定義一下什麼是Fall-Through

```javascript
// 一般的if/else判斷
let type;

if(a==1) type = "game";
else if(a==2) type = "music";
else if(a==3) type = "food";
else type = "other";

// 當商品的種類再下一層時
if(["pc_game", "tv_game"].indexOf(name) >= 0) type = "game";
else if(["Jazz", "rock"].indexOf(name) >= 0) type = "music";
else if(["cake", "cookies"].indexOf(name) >= 0) type = "food";
else type = "other";

// 改用switch改寫的版本
switch(name){

  case "pc_game":
  case "tv_game":
    type = "game";
    break;

  case "Jazz":
  case "rock":
    type = "music";
    break;

  case "cake":
  case "cookies":
    type = "food";
    break;

  default:
    trype = "other";
    break;

}
```

**\[結論\]** Fall-Through是將單個或多個條件導向同一個結果的過程

> 如果想多了解可以使用前往以下連結了解 [1. Fall-Through定義](https://www.fooish.com/javascript/switch-case.html) [2. Example:Fall-through](https://exlskills.com/learn-en/courses/javascript-fundamentals-basics_javascript/conditional-statements-zgrXFcSqdfIF/the-switch-statement-zExqBoStZBEp/fall-through-behavior-nrZCtHxppaWk)

本次變更的內容如下：

1. v-on在component的監聽器上將進行fallthrough，並在子組件根節點\(root node\)上註冊監聽事件，所以將不再使用.native修飾符\(Vue3中將棄用\)

   在Vue2裡使用v-on事件是不會進行綁定的而是會傳遞給component.$listener，而在Vue3中v-on會直接將事件綁定在component的root節點當你傳遞的事件是 @keyup, @keydown, @input, @change時，如果子節點中有input節點，事件將會被自動綁定，當然這也可以在component中設定inheritAttrs為false讓root節點不自動繼承，而變更為手動繼承。

   > 棄用項目**.native**

2. 在Vue3裡inheritAttrs為false時已經可以影響class和style屬性

   在Vue2中就算開啟了inheritAttrs為false，你會發現class和style屬性還是依然產生在root節點，在Vue3中已經改變了這個規則

3. $attr負責補捉除了props己定義的傳入參數，其中包括了class,style,v-on事件

   > 棄用項目**$listeners**

   讓我們看一下以下範例

   ```markup
   <script>

    // example component
    const example = {
      inheritAttrs: true, // => inheritAttrs值預設為true
      props: ["title"],
      template: `
      <div>
        <h1>{{title}}</hr>
        <input type="text" />
      </div>
      `
    }

    // ... 以下省略

   </script>

   <!-- [實際使用] -->
   <example
    class="postTitle"
    style="font-size: 24px;"
    v-on:click="log"
   ></example>

      <!--
        預設產生的結果如下例
        而且點擊div(root節點)及h1,input時都將會觸發click事件
      -->
      <div class="postTitle" style="font-size: 24px;">
        <h1>{{title}}</h1>
        <input type="text" />
      </div>

      <!--
        當我們使用inheritAttrs: false時預設產生的結果如下例,
        div(root節點)將不會綁定任何屬性, 也不會自動綁定click事件
      -->
      <div>
        <h1>{{title}}</h1>
        <input type="text" />
      </div>
   ```

4. functional components的attribute將調整如下：
   * 可以使用props
   * $attrs補捉沒被props定義屬性，的只支援class和style還有v-on
5. 不再只能使用一個根節點\(和React的Fragment同概念\)

   使用過Vue2的使用者都知道，不管是APP或是component都只能有一個root節點，除了Vue2中的functional component，但是在Vue3中已經取消了這個限制，使用方式如下例：

   ```javascript
   // RFCS上的範例
   const customComponent = {
    template: `
    <template>
      <h1 class="headTitle">{{title}}</h1>
      <p>
        <slot name="body">Example Body</slot>
      </p>
    </template>
    `,
   };

   // 下面的這個寫法也是可以的
   const customComponent2 = {
    template: `
    <h1 class="headTitle">{{title}}</h1>
    <p>
      <slot name="body">Example Body</slot>
    </p>
    `,
   }
   ```

   **\[注意\]** 使用此方式時$attrs是不會自動做inheritAttrs的，因為多根節點時並沒有明確的根節點，所以Vue無法進行自動inheritAttrs作業，所以必需使用inheritAttrs:false，來自定義$attrs。

\[ RFCS.[0031.attr fallthrough](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0031-attr-fallthrough.md) \]

## \# 變更$attr的強制判斷規則\(這篇有點冗長\)

> Vue針對一些HTML裡的特定屬性\(enumerated attribute\)分別有currently contenteditable, draggable and spellcheck這幾個參數，做為列舉屬性

1. 變更enumerated attribute的內部概念，讓它們和一般的非Boolean屬性的attr使用一樣的概念。
2. 當attribute屬性被設定為flase時，不再移除該屬性，而是保留屬性為false，如果需要移除該屬性時應該設定為null或undefined。

在做變更的比對前我們先來看看Vue2attribute分類並做了那些分類及如何處理：

1. 針對些 element/attribute組合，Vue使終給於對應的IDL屬性，就好像input,select,progress element中的value attribute。 [前往了解何謂IDL屬性](https://developer.mozilla.org/zh-TW/docs/Web/HTML/Attributes#%E5%85%A7%E5%AE%B9%E8%88%87IDL%E5%B1%AC%E6%80%A7)
2. 針對 boolean attribute（checked,selected...）及xlins，如果使用時參數為falsy\( = undefined \|\| null \|\| false\)時，Vue將會移除他們，不為falsy時才會進行設定。
3. 針對enumerated attribute時將強制將你設定的值轉成字符串\(目前僅針對contenteditable做了修正\)。
4. 其它的attribute參數為falsy\( = undefined \|\| null \|\| false\)時，Vue將會移除他們，不為falsy時才會進行設定。

以下是Vue2實現對照表

| 綁定參數 | attr=基本屬性 | attr=列舉屬性 |
| :--- | :--- | :--- |
| :attr=\"null\" | / | draggable=\"false\" |
| :attr=\"undefined\" | / | / |
| :attr=\"true\" | foo=\"true\" | draggable=\"true\" |
| :attr=\"false\" | / | draggable=\"false\" |
| :attr=\"0\" | foo=\"0\" | draggable=\"true\" |
| attr=\"\" | foo=\"\" | draggable=\"true\" |
| attr=\"foo\" | foo=\"foo\" | draggable=\"true\" |
| attr | foo=\"\" | draggable=\"true\" |

從上表可得知在Vue2中如果將非prop指定attr的值設定為false時，該attr在Render時會被移除

以下是Vue3變更後的結果

| 綁定參數 | attr=基本屬性 | attr=列舉屬性 |
| :--- | :--- | :--- |
| :attr=\"null\" | / | /\(差異\) |
| :attr=\"undefined\" | / | / |
| :attr=\"true\" | foo=\"true\" | draggable=\"true\" |
| :attr=\"false\" | foo=\"false\"\(差異\) | draggable=\"false\" |
| :attr=\"0\" | foo=\"0\" | draggable=\"0\"\(差異\) |
| attr=\"\" | foo=\"\" | draggable=\"\"\(差異\) |
| attr=\"foo\" | foo=\"foo\" | draggable=\"foo\"\(差異\) |
| attr | foo=\"\" | draggable=\"\"\(差異\) |

可以看到Vue3把屬性的對應調整的更合理化了

> 補充以下範例，如果沒有使用v-bind時都將被補捉成資料格都皆為string:

```markup
<example value1="true" v-bind:value2="true" ></example>

<script>

  // example component
  const example = {
    // ...省略
    created(){
      console.log(this.$attrs)
      /* 結果會是
      {
        value1: "true",
        value2: true
      }
      */
    }
  }

</script>
```

\[ RFCS.[0024.attribute coercion behavior](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0024-attribute-coercion-behavior.md) \]

## \# 移除filters內容過濾

移除了Vue2中的filters option了，直接看下例：

```markup
<!-- in Vue 2.x -->
<div id="app">{{ cash | format }}</div>

<!-- in Vue 3.x -->
<div id="app">{{ format(cahs) }}</div>

<script>

// in Vue 2.x
new Vue({
  data: {
    cash: 100
  },
  filters:{
    format(value){
      return `${value}.NT`;
    }
  }
})
.$mount("#app")
;

// in Vue 3.x
Vue.createApp({
  data(){
    return {
      cash: 100
    }
  },
  methods: {
    format(value){
      return `${value}.NT`;
    }
  }
})
.mount("#app")
;

</script>
```

\[ RFCS.[0015.remove filters](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0015-remove-filters.md) \]

## \# 自定義指令API\(directive\)變更

1. 統一了生命週期宣告方式，和APP引用方式相同
2. 自定義指令的相關使用方式，將與RFCS.0031-attr-fallthrough相同也就是本篇（attr使用fallthrough特性）的內容

本次的變更對原本看到directive就退縮的人應該有很大的幫助

```javascript
// in Vue 2.x
const MyDirective = {
  bind(el, binding, vnode, prevVnode) {},
  inserted() {},
  update() {},
  componentUpdated() {},
  unbind() {}
}

// in Vue 3.x 這樣改實在太貼切了 \(^-^)/
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {}, // <= bind
  mounted() {}, // <= inserted
  beforeUpdate() {}, // <= update
  updated() {}, // <= componentUpdated
  beforeUnmount() {}, // [新加的]
  unmounted() {} // <= unbind
}
```

\[ RFCS.[0012.custom directive api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0012-custom-directive-api-change.md) \]

## \# render函式的變更

我們都知道Vue採用Virtual DOM技術，也就是說當我們使用template或在Vue mount的節點中輸入的HTML都將被解析成VNode，如以下範例：

```markup
<div id="app">
  <div class="head-title">Hello</div>
</div>
```

上例我們可以看成下例，其結果是等價的

```javascript
const App = {
  template: `<div class="head-title">Hello</div>`
};
```

實際上會被解析為以下Vnode，並被包裝成Vue裡的render option\(指的不是render函式\)

```javascript
// in Vue 2.x

const App = {
  render(h){
    h("div", {class:"head-title"}, ["Hello"])
  }
}

// in Vue 3.x

const { h, createApp } = Vue;

const App = {
  render(){
    h("div", {class:"head-title"}, ["Hello"])
  }
}
```

有了以上概念後，讓我們來看一下該項目的變更內容：

1. render函式已變為全域函式

```javascript
// in Vue 2.x

new Vue({
  render(h){
    // h來自render的arg[0]
    return h("div", "hello")
  }
});

// in Vue 3.x

const { h, createApp } = Vue;

createApp({
  render(){
    // [差異] h()函式來至全域
    return h("div", "hello");
  }
})
```

1. render函式的參數規則，已與stateful component及functional components規則相同

```javascript
// functional components
const exampleComponent = (props, { slots, attrs, emit }) => {
  // ...
};
```

1. vonde結構改採用平面化結構

```javascript
// in Vue 2.x

new Vue({
  prop: ["onClick"],
  render(h){

    return h("div", {
      // on是個物件 = 非平面化結構
      on: {
        click: this.onClick
      }
    }, "hello")

  }
});

// in Vue 3.x

const { h, createApp } = Vue;

createApp({
  prop: ["onClick"],
  render(){
    return h("div", {
      // => 被攤平了 = 平面化結構
      onClick: this.onClick,
    }, "hello");
  }
})
```

\[ RFCS.[0008.render function api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0008-render-function-api-change.md) \]

## \# teleport / React Portals\(傳送門\)

提到這一篇就不得不貼一下這一篇[React Protals](http://react.html.cn/docs/portals.html)，基本上來說teleport的概念來自protal，就作者的說明telport的命名也是未了與protal衝突，這個想法實在太讚了同時做到了區區隔化。

其實這其中的概念也就是，讓APP綁定範圍中的某個區塊，能Render在不在範圍中的另一個區塊，我們直接進範例：

如下方我們有二個主要區塊，\#App是Vue主要mount的區塊，而.globalModal則是我們定義的共用Modal區塊，\#AppModalView是我們希望能顯示.globalMoad的目標區塊

```markup
<div id="AppModalView"></div>

<div id="app">
  <div class="globalModal">
    {{modalMessage}}
  <div>
<div>
```

在上方程式碼我們可以看到，如果要直接將\#App裡的.globalModal區塊，render在\#AppModalView裡，基本上在Vue2裡是做不到的，在Vue2裡要實現該功能就必需得自己做些手腳了，不過如果常用的話也實在有點麻煩，因此就產生了portal\(傳送門\)的概念，接下來我們用下例看Vue3怎麼做，如下例：

```markup
<div id="AppModalView"></div>

<div id="app">
  <telport to="#AppModalView">
  <div class="globalModal">
    {{modalMessage}}
  <div>
  </telport>
<div>
```

基本上就是使用了telport的節點包起來再下個to參數就解決了，實在是很貼心阿。telport有以下二個參數

1. **to**: 也就是指向

**\[語法\]** to=String

支援指定id,class和使用動態變數，如下例：

```markup
  <telport to="#AppModalView" :disabled="false"></telport>
  <telport to=".anyClass" :disabled="false"></telport>
  <telport :to="'#AppModalView'" :disabled="false"></telport>
  <telport :to="TeleportBlock" :disabled="isDisabled"></telport>
```

1. **disabled\(boolean\)**: 也就是停用

**\[語法\]** disabled=Boolean

如果停用後，telport將顯示在APP原先綁定的區塊

這個功能被廣泛的應用在，youtube,bilibili等影音平台，看你已經在播放某部影片時，但你仍試著找其它影片時，正在播放的影片就會被縮小成一定區塊，被顯示在右下方。

\[ RFCS.[0025.teleport](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0025-teleport.md) \]

## \# 內建transition節點將不再影響root\(TODO: 實例測式後有誤差\)

> **\[注意\]**目前我測式Vue3.rc.5版本，實測的部份和該份內容有落差，我直接在transition使用v-if仍然是有做用的，不知道是目前功能尚未完整還是有Bug

當component中如果使用`<transition>`進行動態效果切換時\(指的是component的動態變更\)，該component的root節點將不會再觸該發動態效果，就範例看來就是transition節點不能直接在節點上v-if=Boolean。

應用範例：

```markup
<!-- component如下 -->
<template>
  <transition>
    <h1 class="modal">{{title}}</h1>
  </transition>
</template>

<!-- 如下方使用 -->
<example> v-if="showModal" :title="headTitle"></example>

<!-- == 修飾後範例 ================================== -->

<template>
  <transition>
    <h1 v-if="show" class="modal">{{title}}</h1>
  </transition>
</template>

<!-- usage -->
<example :show="showModal">hello</example>
```

改變的差異如下例：

```markup
<!-- 在Vue2可以這樣使用，但在Vue3中這將沒有效果 -->
<transition v-if="show">
  <div></div>
</transition>

<!-- Vue3正確的使用方式 -->
<transition>
  <div v-if="show"></div>
</transition>
```

\[ RFCS.[0017.transition as root](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0017-transition-as-root.md) \]

## \# 內建transition CSS class命名方式變更

`<transition>`節點對應的CSS樣式命名方式變更

```markup
<style>
/* in Vue 2.x */
.v-enter, .v-leave-to{
  opacity: 0;
}

/* in Vue 3.x */
.v-enter-from, .v-leave-to{
  opacity: 0;
}
</style>

<div id="app">
  <button @click="toggle">Toggle</button>
  <transition name="enter">
    <div v-if="show">Example</div>
  </transition>
</div>
```

\[ RFCS.[0018.transition class change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0018-transition-class-change.md) \]

## \# Functional Component/純函式Component\(元件\)

有二項變更：

1. 純函式Component必需使用建立成函式\(function\)而不是物件\(Object\)，在Vue2中是Object且需加上`functional: true`。
2. 在異步載入Component時必需使用專用的函式Vue.defineAsyncComponent**\(該方法將在RFCS.0026獨立做說明\)**

> **\[注意\]** 以下範例僅適用於使用vite或有架設webpack開發環境的人員

```javascript
// in Vue 2.x
const FunctionalComp = {
  functional: true,
  render(h) {
    return h('div', `Hello! ${props.name}`)
  }
};

// in Vue 3.x
import { h } from 'vue'

const FunctionalComp = (props, { slots, attrs, emit }) => {
  return h('div', `Hello! ${props.name}`)
}
```

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => import('./Foo.vue'))
// 上例等價下例：可以看到函式內需返回一個Promise物件
//const AsyncComp = defineAsyncComponent(() => new Promise( (resolve, reject) => resolve(Foo) ))
```

在本項RFCS中還有一個需要注意的項目，**＃明確的聲明props**

```javascript
const FunctionalComp = props => {
  return h('div', `Hello! ${props.name}`)
}

FunctionalComp.props = {
  name: String
}
```

\[ RFCS.[0007.functional async api change](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0007-functional-async-api-change.md) \]

## \# 異步Component專用API

Vue3中當你需要導入異步component時需要使用專用的函式Vue.defineAsyncComponent

基本的用法如下例：

```javascript
import { defineAsyncComponent } from "vue"

// simple usage
const AsyncFoo = defineAsyncComponent(() => import("./Foo.vue"))
```

1. defineAsyncComponent的功能最主要是解析異步回傳的Component，當本身是Promise物件時，自動解析Component到Vue Component格式
2. 和Vue2最大的差異在，Vue3中只接受完整的Promise物件，不再接收resolve或reject的Promise物件

```javascript
// in Vue 2.x
const Foo = (resolve, reject) => {
  /* ... */
}

// in Vue 3.x
const Foo = defineAsyncComponent(() => new Promise((resolve, reject) => {
  /* ... */
}))
```

也支援使用options**\(解決方案較完整\)**方式進行，如下例：

```javascript
import { defineAsyncComponent } from "vue"

const AsyncFooWithOptions = defineAsyncComponent({
  loader: () => import("./Foo.vue"), // ===> 格式為 () => Promise<Component>
  loadingComponent: LoadingComponent, // 讀取中時顯示的Component
  errorComponent: ErrorComponent, // 錯誤時回傳該Component
  delay: 100, // 預設為: 200毫秒 (和Vue2完全相同)
  timeout: 3000, // 預設為: 不限時間 (和Vue2完全相同)
  suspensible: false, // 預設為: true
  // 當加載錯誤時，可使用onError進行重試
  onError(error, retry, fail, attempts) {
    if (error.message.match(/fetch/) && attempts <= 3) {
      retry()
    } else {
      fail()
    }
  }
})
```

options使用上Vue2與Vue3的使用比較如下例：

```javascript
// in Vue 2.x
() => ({
  component: Promise<Component>
  // ...other options
})

// in Vue 3.x
defineAsyncComponent({
  loader: () => Promise<Component>
  // ...other options
})
```

\[ RFCS.[0026.async component api](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0026-async-component-api.md) \]

## \# 自定義內部element\(元素/標籤\)

**變更：**

1. 在Vue2中的`Vue.config.ignoredElements`設定，在Vue3中被改為`app.config.isCustomElement`。
2. 該資料格式從原本的string, array\[string, regex\], regex被改為function，如果不是函式時將報錯。

**功能：**

用來自定義像`<transition>,<component>`標籤一樣的Vue內建元素，只要定義過後，只有要component和其\(同名/比對披配時\)都將被視為保留標籤，自定義內部標籤，現在白名單\(whitelisting tags\)的編譯改在模板編譯期間執行時進行檢查，且應通過編譯時設置選項而不是在運行時才進行設置。

本項目附帶is修飾符的變更：

1. 將專用語法is目前只能在`<component>`標籤內使用。
2. 加入v-is指令，用來解決在指定HTML元素中只能存在指定子元素的問題，像`<ul>,<ol>,<table>,<select>`只能放`<li>,<tr>,<option>`這類的限制。

本次的變更最主要是為了優化Vue內建標籤的效能，及用法上的改進，在Vue2中白名單\(whitelisting tags\)是在Vue.config.ignoredElements中設定的，它的缺點是，使用該設置選項時，需要對每個vnode建立的調用進行檢查。

**\[RCF中的說明\]** 在Vue3中則是在模板編譯期間執行時進行檢查，例如我們要建立以下的內建節點：

```markup
<plastic-button></plastic-button>
```

編譯器會將上例編譯為以下代碼：

```javascript
function render() {
  const component_plastic_button = resolveComponent('plastic-button')
  return createVNode(component_plastic_button)
}
```

但是當編譯器未找到對應component時將會發出警告訊息，事實上如果使用者如果希望建立名稱為plastic-button的自定義元素，則所需的生成代碼應為：

```javascript
function render() {
  return createVNode('plastic-button')
}
```

那要怎麼告訴編譯器`<plastic-button>`是自定義元素呢？

基本的使用方法如下例：

```javascript
const app = Vue.createApp(/*...*/)

app.config.isCustomElement = tag => tag === 'plastic-button'
```

如果你使用webpack的vue-loader則應該如下例：

```javascript
{
  // in webpack config
  rules: [
    {
      test: /\.vue$/,
      use: 'vue-loader',
      options: {
        compilerOptions: {
          isCustomElement: tag => tag === 'plastic-button'
        }
      }
    }
    // ...
  ]
}
```

> **\[注意\]** 運行時設置只影響運行時編譯的模板，不會影響預設編譯的模板。

本章提到的新指令`v-is`和舊指令`is`的差異

1. v-is的value只能是字串，而is的用法和attr用法相同`is="字串"`和`:is="變數"`。
2. 在Vue3中，v-is可以在各類HTML標籤裡使用包括了`<component>`標籤，而`is`被限制只能在`<component>`標籤裡使用。

在Vue2裡我們要在特定標籤裡`<ul>,<ol>,<table>,<select>`render component，只能使用以下方式解決:

```javascript
// in Vue 2.x

<table>
  <tr is="blog-post-row"></tr>
</table>
```

在Vue3中，因為is指令規則的變更，我們需要將`is`改為`v-is`，如下例：

```javascript
// in Vue 3.x

<table>
  <tr v-is="'blog-post-row'"></tr>
</table>
```

\[ RFCS.[0027.custom elements interop](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0027-custom-elements-interop.md) \]

## \# \[只有介紹\] Composition API / 組合式API

Composition API可以說是Vue3中的主推功能，至少網路上一面倒這麼說。由於概念與Vue2的使用概念完全不同。他的生成我在看完作者的介紹後，大致上做了以下的簡單結論，Composition API就是為了大專案而生的一種使用方式。

**\(@\)** 那Composition API能做到什麼呢？

1. 能有效率的拆分大型專案的各類程式邏輯區塊\(大=&gt;中=&gt;小=&gt;極小\)，使用上注重分離與組合，這樣更能有效的增加重用性。
2. 能被抽離的不只有操作邏輯，還有資料邏輯也提供了對應的方法可進行抽離。
3. 使用上完全避開了this，對this概念的同學們應該開心了不少。
4. 資料的觸發上，可以針對指定的資料進行監控，只有在對應的資料變更的區塊才會觸發事件。
5. 由於將程式邏輯都拆分了，在使得HTML內容變的更容易閱讀，可以參考本篇[非侵入式JavaScript](https://zh.wikipedia.org/wiki/%E9%9D%9E%E4%BE%B5%E5%85%A5%E5%BC%8FJavaScript)。
6. 使用上幾乎全部使用函式及基本的變數聲明，這也使得該開發方式更適合與typescript做結合。
7. 不一定需要依賴createApp，才能開發反應式程式，可以自由的搭配原本的常用javascript library。

所以你的專案，如果只是簡單的單一功能頁面，事實上是不需要這樣使用的。

所以在這裡我只暫時提供相關資源，之後將獨立該章節方便做一連貫的說明。

1. [Composition API中文手冊](https://composition-api.vuejs.org/zh/)

\[ RFCS.[0013.composition-api](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0013-composition-api.md) \]

