# VUE 3 Composition API / 可組合API

{% hint style="info" %}本篇主要針對Vue3的Composition API做說明，如果想簡略的了解為何該使用Composition API，請參考上一篇的[Composition API介紹](https://jang-arc.gitbook.io/my-cookbook/vue3/vue3_rfcs_note#zhi-you-jie-shao-composition-api-zu-he-shi-api){% endhint %}

觀看本篇前需要事先了解的事情，這也是我認為最先需要了解的特性，在Vue3中提供了全域化API，如下例。我們可以看到可以直接把Vue提供的函式解耦後直接使用。

````javascript

const { h, createApp } = Vue;

const app = createApp({
  template: `<h1 class="title">Hello Vue 3 Composition API</h1>`
});

app.mount("#app");

````

[前往了解VUE3的=>全域化API說明](https://jang-arc.gitbook.io/my-cookbook/vue3/vue3_rfcs_note#quan-yu-hua-api)

## # Composition API的基本開發環境及程式撰寫風格

這裡來我們來看一下如果你使用Composition API開發程式碼，大致的開發結構

**[範例 1]** 不使用 Vue.createApp

````html

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

````

上例我們可以看到幾個特性：

1. 全域API
2. 沒有強制使用Virtual DOM(沒有Vue.createApp().mount過程)
3. 俱備reatcivity(反應性)函式watchEffect
4. 沒有this

**[範例 2]** 使用 Vue.creaetApp.mount() + HTML template

````html

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
````

上例和第一個例子的差異，可以看到是沒有使用watch或watchEffect做資料監控，使用setup做為人口點，將監控的資料return給Vue內部

**[範例 3]** 使用 Vue.creaetApp.mount() + return render function

````html

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
````

接下來我將逐一介紹常用API

## # 資料類API

### ## reactive API

**[功能]** 用來建立俱反應性資料，可以配合watchEffect和watch做監控，當資料變更時可以補捉並進行事件觸發。
**[允許資料格式]** Object/Array

````javascript

const { reactive } = Vue;

const state = reactive({ count: 0 });

const increase = () => { state.count++; };
const decreasing = () => { if(state.count > 0) state.count-- };

````

### ## ref API

**[功能]** 功能和reactive相同，不同的是ref可以針對Boolean/Number/String等非object資料進行監控。
**[允許資料格式]** Boolean/Number/String

````javascript

const { ref } = Vue;

const count = ref(0);

const increase = () => { count.value++; };
const decreasing = () => { if(count.value > 0) count.value-- };

````

### ## computed API

**[功能]** 就像Vue2中的option computed用途相同

````javascript

const { reactive, computed } = Vue;

const userData = reactive({
  firstName: "arc",
  lastName: "jang",
  fullName: computed(()=> firstName + ' ' + lastName)
});

````

### ## readonly API

**[功能]** 用來引用指定reactive或ref資料並產生一個只讀資料

````javascript

const { reactive, computed } = Vue;

const userData = reactive({ count});

````

### ## watch / watchEffect API

**[功能]** 可以針對已建立的反應性資料(reactive, ref)進行監控，讓開發人員在沒有綁定Vue.createApp的情況下仍然擁有監控資料的能力。

下例為 reactive + watchEffect

````javascript

const { reactive, watchEffect } = Vue;

// 定對俱反應性資料
const state = reactive({ count: 0 });

// 建立資料變更函式
const increase = () => { state.count++; };
const decreasing = () => { if(state.count > 0) state.count-- };

watchEffect( ()=> console.log(state.count) );

increase();

````
