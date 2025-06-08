---
title: "[Javascript] Debugging in Chrome"
author: Ula Hsieh
tags:
  - Javascript
categories:
  - Programming
date: 2020-04-12 21:13:00
slug: js-debugging-in-chrome
---

Debugging 是指在一個腳本中找出錯誤。市面上大宗瀏覽器都支持開發人員工具，能幫助開發者一步步地追蹤代碼，查看當前實際運行情況。
此篇文章以常用的 Google Chrome 做說明－

<!--more-->

## 進入開發人員工具

按 `F12` 或是右上角選單中的 `更多工具` \ `開發人員工具` 開啟面板。

![](https://imgur.com/4kW4nnK.png)

通常 debugging 最常使用到的是 `source` 以及 `console` 面板。

### Console

控制台，可以輸入命令執行，以及用來輸出資料(如 `console.log` 將顯示於此)。

![](https://imgur.com/p73UTZ9.png)

### Source

![](https://imgur.com/TX1JU5q.png)

Sources 頁面包含三個部分：

1. 文件導航 (File Navigator)：列出了所有依附於此頁面的文件。
2. 代碼編輯 (Code Editor)：展示程式碼。
3. JavaScript Debugging：

#### debugging 操作

| 按鈕                                                 |          操作          | 描述                                                                                                                                                         |
| ---------------------------------------------------- | :--------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="https://imgur.com/GQ1VrFh.png" width="35"> |         Resume         | 繼續執行直到下一個斷點。如果沒有遇到斷點，則繼續正常執行。                                                                                                   |
| <img src="https://imgur.com/cPPUvKC.png" width="35"> |      Long Resume       | 繼續執行，將斷點停用 500 毫秒。便於暫時跳過斷點，否則會持續暫停執行代碼，例如，循環內的斷點。點擊並按住 Resume，直到展開以顯示操作。                         |
| <img src="https://imgur.com/qNWOS6e.png" width="35"> |       Step Over        | 運行下一條指令，但不會進入到函數中，會在無形中執行函數調用，跳過函數內部。執行會在該函數執行後立即暫停。如果我們對該函數的內部執行不感興趣，這命令會很有用。 |
| <img src="https://imgur.com/JQnb7H3.png" width="40"> |       Step Into        | 如果下一行包含一個函數調用，Step Into 將跳轉並在其第一行暫停該函數。                                                                                         |
| <img src="https://imgur.com/IZrFrbH.png" width="40"> |        Step Out        | 函數調用後，執行當前函數剩餘部分，然後在下一個語句暫停。                                                                                                     |
| <img src="https://imgur.com/lKQK3Bj.png" width="35"> |          Step          | 運行下一條語句。一次接一次地點擊，整個腳本的語句會被逐個執行。                                                                                               |
| <img src="https://imgur.com/MGhfpuH.png" width="35"> | Deactivate breakpoints | 暫時停用所有斷點。用於繼續完整執行，不會真正移除斷點。再次點擊以重新啟用斷點                                                                                 |
| <img src="https://imgur.com/UZGcRJh.png" width="35"> |  Activate breakpoints  | 啟用斷點                                                                                                                                                     |

#### debugging 狀態

![](https://imgur.com/NDSHkql.png)

1. Watch  
   顯示任意表達式的當前值。可以點擊加號 + 然後輸入一個表達式。便會隨時顯示它的值，並在執行過程中自動重新計算該表達式。(同在 scope 中看到的變數)
2. Call Stack  
   檢視目前堆疊上的函式或程序呼叫。
3. Scope  
   顯示當前的變量。 Local 顯示當前函數中的變量；Global 顯示全局變量（不在任何函數中）。
4. 斷點  
   在程式碼中設定進入中斷點有許多好處，可以讓 JavaScript 發生例外狀況的時候透過中斷點暫停執行，並檢查當下的   區域變數  (Locals)、設定監看式  (Watch) 與瀏覽呼叫堆疊  (Call Stack) 等等，對於程式的除錯非常有幫助。
   Javascript 斷點添加的方式有以下：
   <font color=DarkBlue>**Sources 斷點**</font>  
    直接在 source 頁面中，點擊　 Activate breakpoints 再點欲加入段點的行號即可完成操作。當斷點添加完畢後，刷新頁面後會執行到斷點位置停住，在 sources 中會看到當前作用域(Scope)中所有變量和值。  
   <font color=DarkBlue>**條件斷點**</font>  
    在 Sources 里還可以設置條件斷點，在斷點位置的右鍵菜單中選擇「Edit Breakpoint...」可以設置觸發斷點的條件，就是寫一個表達式，表達式為 true 時才觸發斷點。  
   <font color=DarkBlue>**Debugger 斷點**</font>  
    Debugger 斷點的添加就是通過在代碼中添加"debugger;"語句，當代碼執行到該語句的時候就會自動斷點。

## 參考資料

[1] https://developers.google.com/web/tools/chrome-devtools/javascript/step-code
[2] https://zh.javascript.info/debugging-chrome

###### 補充

常常在 debug 的時候會佐以 log 去查看每個階段的變數，但在 debugger 查看 array 或是 object 的變數時，Chrome 會顯示最新的值，而非這個 log 時間點的值，舉例：

<img src="https://imgur.com/tcSMSza.png">

解決方法是，在印 array 前先將 array 轉成字串

![](https://imgur.com/k93iP9W.png)
