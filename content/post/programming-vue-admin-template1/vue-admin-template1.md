---
title: "[Vue] 解析 vue-admin-template"
categories:
  - Programming
tags: ["Vue"]
date: 2020-10-12 20:36:00
slug: "vue-admin-template"
---

## 專案結構(src)

<!--more-->

```sh
user@pc10:~/vue-admin-template/src$ tree -L 1
├── api                 # 所有請求，對後端的請求皆寫在這裡
├── App.vue             # 入口頁面
├── assets              # 圖片等資源
├── components          # 全局公共組件，非公共組件在各自的 view 中維護
├── icons
├── layout              # 全局 layout
├── main.js             # 程式入口
├── permission.js       # 權限管理
├── router              # 路由，把頁面放進導航欄
├── settings.js
├── store               # 全局 store 管理，Vuex 倉庫，用於儲存狀態
├── styles              # 全局樣式
├── utils               # 全局公共方法，非公共在各自的 view 中維護
└── views               # 所有頁面，自添加的頁面放這裡
```

---

## main.js

```js
import Vue from "vue";

import "normalize.css/normalize.css"; // A modern alternative to CSS resets

import ElementUI from "element-ui";
import "element-ui/lib/theme-chalk/index.css";
import locale from "element-ui/lib/locale/lang/en"; // lang i18n

import "@/styles/index.scss"; // global css

// In Vue Webpack template, Webpack is configured to replace @/ with src path，@ 是webpack中定義的別名，代替 resolve('src') 指向的路徑

import App from "./App";
import store from "./store";
import router from "./router";

import "@/icons"; // icon
import "@/permission"; // permission control

/**
 * If you don't want to use mock-server
 * you want to use MockJs for mock api
 * you can execute: mockXHR()
 *
 * Currently MockJs will be used in the production environment,
 * please remove it before going online ! ! !
 * 下面 if 區塊的用途是指在 production 環境下會使用 mock server，如果後端已經做好準備上線的話，請記得註解掉這邊
 */
if (process.env.NODE_ENV === "production") {
  const { mockXHR } = require("../mock");
  mockXHR();
}

// 補充: Mock 是一個模擬數據生成器，用來模擬 ajax 請求，當後端還沒開發完成時，前端可自行寫靜態模擬數據

// set ElementUI lang to EN
// 如果想要中文版 element-ui，按 Vue.use(ElementUI) 方式聲明
Vue.use(ElementUI, { locale });
// Vue.use(plugin) 是用來安裝插件的，就可以在組件中調用 this.$router、this.$route、this.$store、this.$alert() (ElementUI的彈窗組件) 參數(方法)。

Vue.config.productionTip = false;
// 關掉提醒 production 的警告訊息，開發環境下 Vue 會提供很多警告來幫你對付常見的錯誤與陷阱。而在生產環境下，這些警告語句卻沒有用，反而會增加應用的體積。

new Vue({
  el: "#app", // 要綁定的 DOM 元件
  router,
  store, // 在根實例中註冊 store，會注入到所有子元件中供他們使用。使子元件能用 this.$store 存取 store 中的狀態。
  render: (h) => h(App), //createElement
});
/*
使用 new Vue 建構 vue 實體(instance)，傳入一個選項物件(options object)，裡面含了所有實體所需的屬性，這邊就將掛載元素(el)、router、store、render 等屬性，其他屬性還有如資料(data)、方法(methods)、模板(template)、生命週期掛鉤等等，都可以在官網 API 文件中查找。
*/

// 除了使用 el 綁定 HTML 元素外，也可以在宣告完實例後用 $mount 的方法來掛載
/* 
new Vue({
  render: h => h(App)
}).$mount('#app')
*/
```

---

## App.vue - router-view

```html
<template>
  <div id="app">
    <router-view />
  </div>
</template>

/* 整個 div 的視圖由 router-view
取代，為路由的進入點，路由切換的時候會在這顯示不同的內容。*/

<script>
  export default {
    name: "App",
  };
</script>
/* export default 為 ES6 的語法 export 用來導出模塊/變數，通常是一個 Vue
instance 的選項物件(options object)，可以在其他地方使用 import
引入，舉個栗子，這邊 export 了 App.vue 模組，在上面 main.js 使用 import 匯入了。
export 可區分為兩種： -named
export（具名匯出）：可匯獨立的物件、變數、函式等等，匯出前必須給予特定名稱，而匯入時也必須使用相同的名稱。另外，一個檔案中可以有多個
named export。 -default export（預設匯出）：一個檔案僅能有唯一的 default
export，而此類型不需要給予名稱。 name: ‘app’ 相當於一個全局
ID；可以不寫；寫了可以提供更好的調用。 */
```

---

## router/index.js

```js
import Vue from "vue";

// 引用 vue-router
import Router from "vue-router";
Vue.use(Router);

/* Layout */
import Layout from "@/layout";

export const constantRoutes = [
  {
    path: "/login", // 指定要跳轉的路徑
    component: () => import("@/views/login/index"), // 路由對應到的組件
    hidden: true, // 項目將不會顯示在 sidebar
  },

  /* 
    component: () => import('@/views/login/index'),
    lazy loading，把不同路由對應的組件分割成不同的代碼塊，然後當路由被訪問的時候才載入組件，更加高效。
  */

  {
    path: "/404",
    component: () => import("@/views/404"),
    hidden: true,
  },

  {
    path: "/",
    component: Layout,
    redirect: "/dashboard", // 重定向
    children: [
      {
        // 嵌套路由的子路由
        path: "dashboard",
        name: "Dashboard", // 路由名稱
        component: () => import("@/views/dashboard/index"),
        meta: { title: "Dashboard", icon: "dashboard" }, // 路由元訊息
      },
    ],
  },

  // 404 page must be placed at the end !!!含有通配符(*)的路由應該放在最後，通常用於客戶端 404 錯誤。
  { path: "*", redirect: "/404", hidden: true },
];

const createRouter = () =>
  new Router({
    scrollBehavior: () => ({ y: 0 }), // 對於所有路由導航，切換後讓頁面滾動到頂部。
    routes: constantRoutes,
  });

const router = createRouter();

export function resetRouter() {
  const newRouter = createRouter();
  router.matcher = newRouter.matcher; // reset router
}

export default router;
```
