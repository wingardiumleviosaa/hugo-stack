---
title: "程式初學者基礎知識 - CLI, 計概, 網路, session & cookie, 演算法"
categories:
  - Programming
date: 2020-06-14 22:28:00
slug: lidimy-cs101-note
---

這篇文章是紀錄程式實驗導師第一周觀看課程 CS101 的筆記，主要闡述了程式初學者該具備的基礎知識，包含 Coding 簡介、前後端介紹，以及一些計算機概論。

## 到底寫程式是什麼

`Coding` 的目的就是要跟電腦溝通，要對電腦下指令，讓電腦照著指示做。  
那為什麼需要程式碼呢？ 因為電腦只看得懂以 0 和 1 組成的機器語言，且它針對各種不同的事都只按照同一個標準去做事（標準化），例如不同廠牌的 USB 須按照 USB 標準去生產，而電腦只須懂這個標準就好。
所以寫程式的標準就是程式碼，而市面上的各種程式語言最終都會轉換成電腦懂得機器語言。

<!--more-->

## 程式碼不是重點，解決問題才是

把問題拆解，並試著把解法寫成條列式，一行就是一個動作，例如：找單字裡面有沒有包含字母 P，有的話位置是第幾個？
在經過將無限重複的步驟有限化並加上結束判斷、將提到的變數命名一個代號、將答案回傳等都納入後會得到下面的步驟：

![](https://imgur.com/utdAp5U.png)

## 什麼是 Command Line

與之對應的是圖形化介面 GUI (Graphic User Interface)，GUI 指的是用看得到的圖形介面去操作電腦。而 Command Line Interface 指的是只能透過文字跟電腦做溝通的介面。
問題是，GUI 這麼直覺好用，為什麼還要有 CLI 呢？因為---

1. 有些功能只能用 CLI
2. 有時候用文字去操作指令比較快

這邊羅列幾個常用到的 Command：

- pwd：print working directory，列出當前所在位置
- ls：list Segment 印出檔案清單，加上 `-al` 參數可以列出隱藏檔案 (-a) 並把清單詳細條列 (-l)。
- cd：change directory 切換資料夾，`$ cd ..` 代表回到上一層，`$ cd ~` 代表回到 home 目錄 (使用者的專屬文件目錄)，`$ cd /` 代表回到根目錄 (存放電腦底層檔案)。
- man：manual 可以叫出指令的使用手冊，例如 `$ man ls` 列出 ls 的使用方法。
- touch：`$ touch <fileName>`，如果檔案不存在，則會建立一個新檔案；如果檔案存在，則會修改檔案時間。
- rm：`$ rm <fileName>` 刪除檔案，`$ rm -r <fileName>` 刪除資料夾。
- mkdir：`$ mkdir <directoryName>` 建立資料夾於當前目錄。
- mv：`$ mv <oldName> <newName>` 改檔案名字，`$ mv <oldName> /opt/<newName>` 移動檔案到 /opt 路徑下並改檔名。
- cp：`$ cp <fileName> /opt` 複製檔案到 /opt 目錄底下；　`$ cp <directoryName> -r /opt` 複製資料夾到 /opt 目錄底下。
- cat：catenate，連結檔案、印出檔案內容到 ，`$ cat <fileName>`。
- less：分頁式印出檔案，不同於 `cat` 一次會印出檔案會暫 console 很大版面不好閱讀，`$ less <fileName>`，按 ‵q‵ 可離開分頁式畫面。
- echo：印出字串。
- grep：可以抓取指定關鍵字，例如 `$ grep if hello.js`，把所有 hello.js 檔案中包含 if 的字印出來。
- wget：下載檔案，例如 `$ wget <URL>`
- curl：送出 request，可用於測試 API，例如 `$ curl <api URL>`，會回傳 reponse、`$ curl -I <URL>`，會列出 Header。
- `>`：指重新導向，`$ ls -al > result.txt`，將 ls -al 輸出的資料保存到 result.txt。
- `>>`：可以 append 到檔案結尾，`$ echo "append to the end" >> result.txt`。
- `|`：唸做 pipe，指的是前面的輸出為後面的輸入，例如 `$ cat hello.txt | grep h` 會印出 hello.txt 中有 h 的地方。
- date：印出系統目前的時間。
- top：table of process，印出所有 process，類似 Windows 的工作管理員。
- nslookup：解析 domain name 以及 ip 位址的對應。

![](https://imgur.com/CFwzOQy.png)

- ping：丟封包到指定伺服器，檢測是否能連到該伺服器。

![](https://imgur.com/HIg8vjr.png)

- telnet：可以去檢測某伺服器的某個 port 是否正常服務，例如`$ telnet ptt.cc 23`。

## 二進位是什麼－電腦中的資料儲存與表示方法

首先先來談什麼是進位，以十進位來說，代表單一位數裡面不會有`十`，一旦逢十就要進位。八進位代表單一位數不會有 `八`，所以 7 的下一位是 10，代表進位了。  
以數學角度來看的話，十進位就代表以十為基底，例如 `123 = 1*10^2 + 2*10^1 + 3*10^0`
若 123 為八進位的話，那就表示 `1*8^2 + 2*8^1 + 3*8^0` 等於十進位的 83。

<table><tr><td bgcolor=#FAFAFA>
<b>小知識</b></br>
在很多地方常常會看到色碼，例如 #FFFFFF 為白色、#000000 為黑色。這些數字是以 16 進位表示法。電腦裡面的三原色是 RGB 紅綠藍，每個顏色又細分為 256 個數值 (0~255)。例如黃色是紅色 255, 綠色 255, 藍色 0 組合而成，就可以表示成 #FFFF00
</td></tr></table>


## 電腦儲存單位

    - Bit 位元，資料儲存的最小單位，只能存 1 或 0
    - Byte 位元組，byte = 8 bit
    - KB Kilobyte，1 KB = 1024 byte = 2^10 byte
    - MB Magabyte，1 MB = 1024 KB
    - GB Gigabyte，1 GB = 1024 MB
    - TB, PB, ZB, YB … 以此類推

基本上 PB 級以上的單位日常幾乎用不到，只應用在大數據領域，例如 Google 的 MapReduce 每個月要處理的資料量超過 400 PB、Facebook 註冊用戶超過十億，每天產生 300 TB 以上的資料…

## 數字的儲存

電腦通常會用 32 bits 來存數字，也就是 4 bytes，每一個 bit 可以表示 0 或 1，32 個 bit 就會有 2^32 種情形，大約是 40 億左右。

### 負數的表示方法

最前面位數如果為 1 表示是負數，負數的二進位可以由該數的正數二進位的 0、1 顛倒最後加 1 來求得，這個表示法稱作**二的補數**。

![](https://imgur.com/XnYJE1w.png)

如果數值超過可以表示範圍的話，會有溢位 overflow 的錯誤狀況發生，例如：

![](https://imgur.com/YKNaFiC.png)

因為 112+64=176 > 127 (用 8bit 存的可表示範圍 -128~127)

### 電腦儲存小數的方法

![](https://imgur.com/wd9dXeR.png)

所以 10^-5 的 -5 會存在 exponent 的 11 bits 中，853 存在後面 mantissa 52 bits 中。

## 網路基礎概論

從你在瀏覽器輸入 google.com 到畫面出來後，中間發生了什麼？

先解釋一些網路的基礎名詞：  
 - IP：網路上透過 ip 位址來表示主機所在的位置，為網路 & 電腦溝通的地址  
 - Domain：域名，即為我們常用的網址例如 google.com，比起 IP 可讀且好記 - DNS：Domain Name System，負責把域名轉換成 IP。8.8.8.8 是 goolge 提通的免費 DNS 伺服器  
 - Port：端口或是連接阜，伺服器上通常提供很多服務，服務就是用端口去區分的，例如 80 port 就是開給網頁 http 用的、443 給 https 用。  
 - 前端：只使用者看得到的部分，包含顯示網頁內容的 HTML、負責網頁樣式排版的 CSS 以及負責使用者的操作行為 (如點擊按鈕之後會出現什麼) 的 JavaScript。  
 - 後端：後端指的是背後運作的程序、資料交換或儲存，比如說在搜尋引擎點選搜尋的剎那，就會把關鍵字發送到網頁後端去，後端伺服器就會從資料庫撈資料，這個撈出來再回傳的動作，就是後端看不見的程式完成的。

回到段落一開始的問題---

![](https://imgur.com/g2jU9vl.png)

以上就是基本網頁溝通的情境／流程。

## 內網與外網

一般來說，公司的網路都會形成一個「內部網路」，內部所使用的 IP 是虛擬 IP，只在內網裡面才看得到。但對外面的人來說，他們只看得到你們公司的唯一一個 IP 地址，這就是外網跟內網。

![](https://imgur.com/S4xUVXn.png)

## VPN

Virtual Private Network，有些內網服務為了提升安全性會鎖 IP，只開放內部私有 IP 存取，如果要從外部網路連進去，就需要透過 VPN 來翻牆。

## Session & Cookie 的概念

通常你在瀏覽網頁的時候(譬如購物網站)，當您登入後然後在各個子業面上做切換，為什麼切換時都能保持登入狀態？或是為什麼購物車的東西不會因為切換而不見？
登入功能背後的原理－

![](https://imgur.com/xgc2Q34.png)

登入之後，server 怎麼知道剛剛的 request 跟現在是同一個人?

### session

可以把它想成識別證，server 會儲存 session ID 和與之對應的內容例如帳號。

### Cookie

是讓瀏覽器儲存資訊的地方，server 可以要求瀏覽器設置 cookie 用於存放 session ID，之後的每個 request，瀏覽器都會把 Cookie 帶上來。
總而言之，就是登入後會給你一張識別證，等下次再來的時候出示識別證就能知道你是誰了。
所以加入 session & cookie 的流程變成以下：

![](https://imgur.com/U3g34OK.png)

## 同樣的網頁載不同瀏覽器為什麼會跑版?

網頁的組成固定是由 HTML、CSS 以及 Javascript 組成的都只是一堆文字，而瀏覽器負責翻譯，不同瀏覽器有不同的標準，就會有不同的行為差異。

## 資料庫概念

### SQL

Structural Query Language，用來操作資料庫的語言，常見指令如下：

#### 查詢

`SELECT phone FROM users where name=Peter;`，從 users 資料表裡找到 name 是 Peter 的那列，並把 phone 這個欄位的值取出來。
`SELECT * FROM users;`，取出 users 資料表的所有資料。

#### 刪除

`DELETE FROM users WHERE name=Peter;`，刪除 users 資料表中 name 是 Peter 的那列。

#### 更新

`UPDATE users SET phone=123456 WHERE name=Peter;`，更新 users 裡面 name 是 Peter 的那列，把 phone 設為 123456。

#### 新增

`INSERT INTO users(name,phone) VALUES (Alice, 789012)`，新增一筆紀錄，name 是 Alice，phone 是 789012。

## 初級攻擊手段

### DOS & DDOS

#### DOS

Denial of Service ，惡意不斷發 request，導致有限資源的伺服器要不段處理單一需求，進而沒辦法處理其他人需求，而出現錯誤或是連不上的情形

#### DDOS

Distributed Denial of Service，從不同來源不段發出 request，以癱瘓服務，通常都是經由木馬植入被害人電腦，讓該電腦被操控去做攻擊

### 一些盜帳號的方式

#### 木馬程式

駭客可以透過被裝入木馬的電腦開一個後門，就可以從遠端連線到該電腦去做任何事，可以利用你當跳板去入侵網站，或是讓你成為 DDoS 攻擊的一員，也可以偷你電腦中的資料

#### 暴力破解

嘗試所有可能的字母數字組合，或是使用一些常見的密碼組合去試（字典檔）。

#### SQL Injection

先來一個範例，上面提到資料庫的查詢語法 `SELECT`，如果今天駭客下的是 `SELECT * FROM users WHERE username=''or 1=1' AND password='123456'`；帳號的地方填入`'or 1=1`時，後面都會被省略，而 `or 1=1`保證是 true，所以一定會找到資料。
所以 SQL Injection 代表攻擊者去鑽一些設計不良的程式的漏洞，透多特殊的文字變成程式去騙後端程式。

#### XSS

Cross Site Scripting，同樣是讓輸入變成程式的一部份，例如在網站中可以輸入文字的地方他寫了一段 Javascript 去擷取網站的 cookie，就有可能盜用你的身分去做事情。

SQL Injection 以及 XSS 兩種攻擊方式都是因為沒有處理好「使用者輸入」而造成非預期的程式執行。

## 網站的密碼安全

一般網站針對密碼都不會真正的儲存明碼，而是雜湊過的密碼，使得駭客就算入侵網站取得資料庫，也不曉得較敏感的密碼資訊。

#### 雜湊函數

是一個單項的函數，雜湊後不可反推回去，輸入一樣保證輸出一樣，最有名的函數是 MD5。
不可逆的原因是因為有無限的輸入但只有有限的輸出，所以造成可能有兩個不同的輸入有想同的輸出，代表產生碰撞。
所以這就是為什麼通常忘記密碼只能重設，而無法告訴你原本的密碼。
SHA256 是比 md5 更安全的函數，但要注意的是愈安全，加解密需要更多運算資源，速度就愈慢。

#### 加鹽 (salting)

但如果僅僅是用 md5 的話，還是有可能可以用暴力破解法破解。此時使用加鹽，會自動幫使用者產生一段亂數，做雜湊時是對 (亂數+密碼) 做雜湊，可以確保就算暴力破解了也不會是正確的密碼。

## 程式基礎概念

### 組合語言

低階語言，很接近電腦底層，一個指令只做一兩件事情，可以翻譯成機器語言。

### 編譯器

把原始語言編譯 (Complie) 成目的語言，簡單來說就是個翻譯機。因為電腦只看得懂機器語言，所以寫 C 會有 C 的 Complier，寫 Java 會有 Java 的 Compiler，這些最終都會被翻成機器語言。

### 逆向工程

既然可以從程式碼變成機器看得懂的語言，當然就可以「逆向」回來 把機器碼變成看得懂的語言，這個就叫做逆向工程。
逆向以後你甚至可以修改程式，把某些地方改成你想要的樣子 改完之後再重新翻譯回去，就變成破解版的程式了。
從組合語言回推到機器語言的過程稱作**反組譯** Disassembly

## 各式各樣的程式語言

每一種程式語言都有自己擅長的領域，功能、語法也都不太一樣。

- C：適合打基礎，了解電腦底層運作（記憶體、指標）。
- C++：和 C 的差別在於語法以及多了物件導向的概念。
- Java：編譯出來的程式碼跑在 Java 虛擬機上 (JVM)，達到跨平台的特色，寫 Android 必學語言。
- Javascript：跟 Java 沒關係，原本只能跑在瀏覽器，後來可以在瀏覽器以外的地方執行。
- Python：可以做資料分析、網頁、小工具，資源多且語法簡單，適合初學者。

## 演算法是什麼

演算法就是解決問題的方法，意即當有問題輸入，透過演算法，會得到結果輸出。
可以用時間複雜度與空間複雜度去比較演算法的優劣。

- 時間複雜度：一個演算法平均需要多少時間來完成。詳細可以參考[文章](https://medium.com/appworks-school/%E5%88%9D%E5%AD%B8%E8%80%85%E5%AD%B8%E6%BC%94%E7%AE%97%E6%B3%95-%E5%BE%9E%E6%99%82%E9%96%93%E8%A4%87%E9%9B%9C%E5%BA%A6%E8%AA%8D%E8%AD%98%E5%B8%B8%E8%A6%8B%E6%BC%94%E7%AE%97%E6%B3%95-%E4%B8%80-b46fece65ba5)

![](https://imgur.com/2zF1BAn.jpg)

當資料筆數愈大，所需要的時間的關係。

- 空間複雜度：一個演算法平均需要多少空間來完成。

### 時間換取空間；空間換取時間

有時候可以改變演算法，衡量要速度慢但不會消耗太多記憶體，還是速度快，但消耗記憶體。例如：從 n 個數字裡找目標數字，共有 m 次查詢。

- 時間換空間：m 次查詢，每次都要從頭找 n 次，O(N\*M)。
- 空間換時間：第一次查 n 次，便紀錄在一張表所有數字出現的次數，往後查詢就只要查一次，O(N+M)。

### 二分搜尋法

藉由排序後的特性，一直不斷的往中間切，將搜尋範圍不斷縮小而不用從頭到尾搜尋，更有效率。

### 排序的演算法

#### 選擇排序法

Selection Sort，從還沒排序的數列裡找最小的，然後移到最左邊。

![](https://imgur.com/spa80Vk.png)
[1]

#### 泡沫排序法

Bubble Sort，左到右兩兩比較，把比較大的數字往右邊移 (浮上來)。

![](https://imgur.com/u8VaYqo.png)

#### 插入排序法

Insertion Sort，類似完撲克牌時排序的方法，從左邊開始把每一張牌都放到正確位置。

![](https://imgur.com/wbN9CNv.png)

#### 合併排序法

Merge Sort，把一個數列切成左右兩半，個別排序後再合併起來。

![](https://imgur.com/59Ar0uV.gif)

#### 快速排序法

Quick Sort，挑選一個基準點 (pivot)，讓左邊的數字都小於它，右邊的數字都大於它，然後對左右兩邊重複此操作。

![](https://imgur.com/m1QYSbu.gif)

合併排序及快速排序都是把大問題切割成小問題

## 前端工程師 & 後端工程師做什麼

![](https://imgur.com/fu9Oyrf.png)

就如同前面講的，網頁由 HTML, CSS, Javascript 組成，前端工程師要做的就是寫出這三支程式，主要跟網頁呈現有關。

![](https://imgur.com/yo3AeXA.png)

而後端工程師負責處理後端的業務流程，比如說判斷使用者帳號密碼、去資料庫撈資料...等等，所以除了要懂主機跟寫應用程式外還需要了解資料庫。

![](https://imgur.com/kI0DgWc.png)

## 應用程式開發

### IDE

整合式開發環境 Integrated Development Environment，拿 Android Studio 為例，把所有開發應用程式要用的功能整合起來了。

![](https://imgur.com/8VTXtAY.png)

## API

應用程式介面 Application Programming Interface，前端工程師要接後端的功能，要用 API 串接，要請後端工程師開一個功能串口。直接舉例比較清楚，以 Google 帳號登入的 api 為例，就是 Goole 提供了其他開發者這個`登入的功能`的 `api`
去實做到他們的應用程式裡。

## 基礎的程式概念

- 條件判斷 Conditional，用在需要判斷抉擇的時候。
- 迴圈 Loop，重複做一樣的事情，通常有中止條件，否則就會變無窮迴圈，無窮迴圈會一直跑直到消耗完記憶體容量為止。
- 變數 Variable，用來儲存資訊。
- 函式 Function，把程序中重複性高的地方切開成許多獨立的小程序。

## Source

[1] https://visualgo.net/en/sorting
[All] 此篇文章 Lidemy CS101 的筆記，內容及圖片大部分取自上課影片
