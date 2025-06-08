---
title: "[Vue] 安裝及基本組成簡介"
categories:
  - Programming
tags: ["Vue"]
date: 2020-10-06 14:59:00
slug: "vue-introduction"
---

## install nodejs and npm

安裝最新版 Nodejs 以及 npm

<!--more-->

```
$ sudo apt update
$ sudo apt install -y nodejs
$ sudo apt install -y npm
$ sudo npm install npm@latest -g
$ sudo npm cache clean -f
$ sudo npm install n -g
$ sudo n latest stable
$ sudo npm -v
$ sudo node -v
```

## install vue

```
$ sudo npm install -g @vue/cli
$ sudo chown -R 1000:1000 "/home/user/.npm"
# let npm install without sudo
$ sudo chown -R $USER:$(id -gn $USER) /home/user/.config
# npm update check without sudo
```

## new a vue project

```
$ vue create . # 在當前目錄下建立
# OR
$ vue create myproject # 或是指定新的資料夾
```

選擇要預安裝的套件。可以先選預設，後續還想新增其他的工具，可以再使用 npm 或是 yarn 安裝即可。

```
ue CLI v4.5.6
? Please pick a preset: (Use arrow keys)
❯ Default ([Vue 2] babel, eslint)
  Default (Vue 3 Preview) ([Vue 3] babel, eslint)
  Manually select features
```

按下 Enter 之後就會開始專案建立，接著等它完成。

![](https://imgur.com/h7SoLPG.png)

可以看到最下方提示，進入專案目錄後下 `npm run serve` 的命令就可以開始開發。

```
🎉  Successfully created project myproject.
👉  Get started with the following commands:

 $ cd myproject
 $ npm run serve
```

---

## npm run serve

![](https://imgur.com/CYuCJvb.png)

完成後，就可以用瀏覽器來打開，看到預設專案的畫面了。

## project's composition

![](https://imgur.com/HB0ZoR1.png)

```
myproject/
├── babel.config.js     # Babel 的配置文件
├── package.json        # 專案所用到的套件列表
├── package-lock.json   # 紀錄套件具體的版本號
├── node_modules        # npm 套件存放位置
├── public              # 公開(靜態)文件目錄，在打包過程中除了 html 模板檔案，其他資源會直接複製到 dist 資料夾下，而不需要編譯和壓縮。
│   ├── favicon.ico
│   └── index.html      # 首頁的 HTML 檔，裡面不會有主要內容，可編輯網頁所需的 meta tag 或是引入外部的 css 或 js
├── README.md
└── src                 # 開發目錄
    ├── App.vue         # 頁面入口，綁定在 index.html 中的 <div id ="app"></div> ，要使用的 component 皆會掛在 App.vue 底下
    ├── assets          # 資源，如 css、image
    │   └── logo.png
    ├── components      # 不同的內容或功能分成的一個個元件，可以被互相引用
    │   └── HelloWorld.vue
    └── main.js         # 專案入口，影響全局，作用是引入全局使用的模組、全局的樣式和方法、設置路由等。
```

### main.js

為 vue 程式的進入點，用於初始化 vue instance 並載入各種公共元件。

```js
// 載入所需套件、專案檔案等...
import Vue from "vue";
import App from "./App.vue";

// 關閉 production 提示功能
Vue.config.productionTip = false;

// 初始化應用程序，創建 vue instance，使用 render 函式將 App.vue component 渲染出來，然後將程序掛載在 DOM 上面，html 中 id 為 app 的元素
new Vue({
  render: (h) => h(App),
}).$mount("#app");
```

render: h => h(App) 的最原始寫法如下

```
render: function(createElement){
    return createElement(App);
}
```

根據 ES6 語法可在簡寫成

```
render(createElement){
    return createElement(App);
}
```

最後再用箭頭表示法縮減成 `render: h => h(App)`。
其中把 createElement 由 h 取代，createElement 函數是用來生成 HTML DOM 元素的。h 指 hyperscript，是 HTML 的一部分，表示的是超文本標記語言，為了方便而直接使用縮寫 h。

### App.vue

App.vue 為頁面的入口，所有頁面都是在這裡進行切換的。

```vue
<template>
  <!-- 模板標籤 -->
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
<!-- component 的邏輯 -->
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
<!-- component 的 css 樣式 -- > #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

## vue 檔結構

`*.vue` 檔案，是一個自定義的檔案型別，每個 .vue 檔案包含三種型別的語言塊 `<template>`、`<script>` 和 `<style>`，分別代表了 html、js、css。
其中 `<template>` 和 `<style>` 是支援用預編譯語言來寫的，例如　`<style lang="scss">`，如果沒特別指定的話，就是使用原生寫法。

### &#060;template&#062;

代表 html 結構，每個 .vue 文件最多包含**一個** `<template>` 模塊，必須在裡面放一個 html 標籤來包裹所有的程式碼，如 `<div>`。除了一般的 html 語法外，還可以使用 Vue 指令來操作、修改或是裝飾 DOM 元件。
基本語法有：

- `v-bind:class="xxx"` 動態綁定 class
- `v-bind:style="ooo"` 動態綁定 style
- `@event` 綁定事件，例如 `@click` 表示綁定一個 Click 事件。
- `{{ variable }}` 顯示某一個變數資料、數字或是文字。
- `<my-component>` 使用自定義的元件，例如 `<HelloWorld>` 便是。
- `v-if`、`v-else` 與 `v-else-if` 用於判斷 DOM 生成與否。
- `v-show` 功能跟 v-if 很相似，不過它是改變元素的 CSS 的 display 屬性，當 v-show 的值為 false，元素則會隱藏。
- `v-for` 產生一個迴圈，當你需要重複產生某一段 DOM 時適用。
- `v-model` 特別用於表單元件，用以替代輸入的接收值。
  其他指令可參考[官方文件](https://vuejs.org/v2/guide/syntax.html)

### &#060;script&#062;

代表 javascript 指令，每個 .vue 文件最多包含**一個** `<script>` 模塊，要使用其他 JS 套件可以使用 require()。如果有安裝 babel，則可以使用 import 語法。使用 `export default {...}` 匯出一個 Vue 物件，程式將寫在這裡面。
其中常用的方法與屬性如下：  
- `component:{}`  申明引用的元件，能在  template  中使用。
- `data(){}`  宣告資料能在  template  中使用。  
- `methods:{}`  是指元件的方法，也可以說是函式。  
- `created(){}`  表示當我們的元件載入完成時，需要執行的內容。是  vue  中的勾子(可以理解為  vue  的生命週期)函式(生命週期內各個階段的事件方法)之一。

![](https://imgur.com/rgg2soY.png)

### &#060;style&#062;

代表 css 樣式，每個 .vue 文件可包含**多個** `<style>` 模塊，作用域預設為全局，如果想要限制為區域作用(只在該組件中有效)，可以加上 scoped 屬性 `<style scoped>...</style>`

## Reference

- https://github.com/vuejs-templates/webpack-simple/issues/29
- https://blog.hinablue.me/2019-ithome-ironman-day-1/

## to be read

**Render Function**
https://medium.com/@speechless0922/vue%E7%9A%84%E4%BA%8C%E4%B8%89%E4%BA%8B-render-function-2b3705e4a5bd
