---
title: "[OS] 作業系統概論"
categories: ["SRE"]
draft: false
date: 2020-02-02 18:05:00
slug: "os-introduction"
---

## 程式、程序、執行緒

### 程式(Program)

尚未載進記憶體的**靜態程式碼集合**。

<!--more-->

### 程序(Process)

正在執行並載進記憶體中的**動態程式**，是作業系統分配資源的最小單位（OS 以程序為單位，分配系統資源，OS 下的工作管理員所標示的一個個 task 即為一個程序，擁有獨立的 PID 以及記憶體空間）。

### 執行緒(Thread)

又稱 Light-weight Process (LWP)，是 OS 中進行運算排程的最小單位，被包在一個 Process 中，而同一 Process 下的各個執行緒之間共享該 Process 資源。

<table><tr><td bgcolor=#FAFAFA>
三者的關係是：一個程式可以會有多個程序，一個程序可能會有多個執行緒。
</td></tr></table>

![](https://imgur.com/nnn6bxW.png)

## 併發(concurrency) v.s. 並行(parallelism)

### 併發

同時執行多個獨立的程式邏輯，但並不代表同時進行處理。
若程式執行在單核單執行緒的 CPU 上，所有任務都須排隊等待 CPU 資源。

### 並行

讓程式**真正的同時**處理多個任務，並行並不是程式能夠帶來的特性，而是需要靠硬體，需仰賴多核 CPU 或是使用多台伺服器組成叢集。

<table><tr><td bgcolor=#FAFAFA>
兩者的關係是：並行的狀況一定會併發，但發生併發時卻不一定是並行。
</td></tr></table>

![](https://imgur.com/KxNlQOJ.png)

## Multiprograming、Multiprocessing、Multitasking、Multithreading

### Multiprogramming

電腦同時<font color=#FF5959>(concurrency)</font>執行多個程式。（如同時使用 excel 以及 firefox）

### Multiprocessing

電腦同時使用多個處理器，多個程序可在同一時間<font color=#FF5959>(parallelism)</font>執行。

### Multitasking

多個任務共享一個資源(如一顆 CPU 或記憶體)。

### Multithreading

指一個程序中有多個執行緒在執行，彼此共用相同的程序資源。

雖然字面上看起來意思接近，但每個名詞都有些微的差距，其中特別記錄在其他網站中看到針對 multitasking and multiprogramming 的闡述＿

```
Multitasking is a logical extension of multi programming. The major way in which multitasking differs from multi programming is that multi programming works solely on the concept of context switching whereas multitasking is based on time sharing alongside the concept of context switching. [1]
```

Multitasking 是 Multiprogramming 的邏輯擴展，主要的不同在於 Multiprogramming 是多個程式基於上下文切換的概念上獨立運作，而 Multitasking 的觀念則是較著重於 time-sharing（可能是同一個程式中的不同程序）。

## Kernel Space & User Space

### Kernel Space 內核空間

有關作業系統的所有數據、程序皆在此運行。

### User Spaceg 使用者空間

指 user 程序的運行空間，權限有限，若要調用系統資源，需透過 system call。

p.s. 兩個空間的劃分依據不同位元以及不同版本的作業系統而不同。

### Protection Ring

為了隔絕不同的程序，CPU 提供了一個 Pretection Ring 的機制，依據不同架構的 CPU，所劃分的層級也不同，但決定於作業系統的需求，例如 Linux 或 Windows Server 2008 僅使用了 Ring 0 和 Ring 3 兩個級別。[2]  
下圖為 x86 處理器可用的特權級別

![](https://imgur.com/FscjQVY.png)

### Kernel Mode & User Mode

當程序運行在 kernel space 時 CPU 就處於 kernel mode，切換到 RING 0 最高權限模式，能直接與硬體互動，而運行在 user space 時 CPU 則處於 user mode，CPU 只採用 RING 3 最低權限模式。

## 模式切換(Mode Switch)

指從 user mode 切換到 kernel mode 的狀況，有以下三種：

1. 系統呼叫 (system call)：使用者程式要求 OS 服務。
2. 異常 (exception)：程式執行非法動作 (stack overflow, divided by zero)。
3. 中斷 (interruption)：當外圍設備完成 user 請求的操作後，會向 CPU 發出中斷訊號，這時 CPU 會暫停當前的程序，轉而去執行中斷訊號對應的處理程序。比如硬碟讀寫操作完成時，系統會切換到硬盤讀寫的中斷處理程序中執行後續操作。

### 小結

區分兩個空間 (space) 以及不同權限 (mode) 的用途與好處:

1. 確保系統數據不被隨意操作，防止破壞
2. 確保資源不被單一使用程序霸佔
3. 確保兩個使用程序不會相互干擾
4. 確保硬體操作正確

上面基礎概念講完後，回到執行緒，討論更深的概念：

## Single-thread & Multi-thread

![](https://imgur.com/6xyHblp.jpg)

每個 Thread 擁有獨立的 stack，多個 Thread 之間因為共用 Process 的記憶體，故可以共用變數。

### Single Thread 單執行緒

每個正在執行的程式(即程序)，至少包括<span class="dotunderletter">**一個**</span>執行緒，稱作主執行緒，它在程式啟動時被建立，用於執行 main 函式。  
而程式若只有一個主執行緒，稱作<font color=DarkBlue>單執行緒程式。</font>  
主執行緒負責執行程式的所有程式碼，這些程式碼只能順序執行，無法併發執行。

### Multi Thread 多執行緒

擁有<span class="dotunderletter">**多個**</span>執行緒的程式，彼此可獨立併發執行。  
能有效地避免程式碼阻塞，提高程式的執行效能。

## ULT & KLT & LWP

### User Thread 使用者執行緒

又稱 User Level Thread (ULT)，在 user mode 下進行，OS 不知 ULT 存在，不需要 OS 介入管理，主要由 user-level thread library (如 Java threads, Win32 threads, POSIX Threads aka Pthreads) 管理。

- 優點：產生、管理成本較低
- 缺點：當 kernel 為 single thread，程序的 user thread 發出 system call 時，則整個程序會被鎖住。

### Kernel Thread 核心執行緒

又稱 Kernel Level Thread (KLT)，在 kernel mode 下進行，OS 知道 KLT 存在，且由 OS 介入管理。一般作業系統皆屬之，如 Linux, Unix, Windows, Mac OS X...

- 優點：當程序內某條執行中的 kernel-thread 被鎖住時（如等待其他資源），不會導致整個程序亦被鎖住，可同時進行其他同在此程序中的其他 Thread。
- 缺點：速度較慢，一個程序中的兩個 threads 切換需要經過 mode switch。

### Low-weight Process 輕量級程序

{{< notice info >}}
這裡簡直是惡夢，在網路上的定義百百種，而且都很模糊，只好在篩選資訊後，把覺得合理的解釋都一併放了上來，將就看一下。
{{< /notice >}}

輕量級執行緒 (LWP) 是一種由核心支援的使用者執行緒。
每一個程序可能有一個或多個 LWPs，每個 LWP 支援一個或多個 ULTs，而每個 LWP 由一個 KLT 支援。

```
A user-level library multiplex(多路通訊) user threads on top of LWPs and provides facilities for inter-thread scheduling, context switching, and synchronization without involving the kernel.[3]
```

</br>

```
注：
1. LWP的術語是借自於SVR4/MP和Solaris 2.x。
2. 有些系統將LWP比喻為虛擬處理器，或是介於user thread 和 kernel thread之間的資料結構，可對 user thread 進行排程以執行在 user-thread library。
3. 將之稱為輕量級程序的原因可能是：在核心執行緒的支援下，LWP是獨立的排程單元，就像普通的程序一樣。所以LWP的最大特點還是每個LWP都有一個核心執行緒支援。[4]
```

## Multithread Model

只執行緒間的溝通模式，主要有三種

![](https://imgur.com/2jUidLK.png)

### 1 : 1

一個 ULT 對應到一個 LWP 再對應到一個 KLT，如同上圖的 process 4，即屬於此模型。

- 優點：實現 parallelism，當一執行緒阻塞，不會影響其他執行緒
- 缺點：每件一個 user thread，就需產生一個 kernel thread，執行緒建立的開銷較大

### N : 1

此模型又稱 pure user thread，多個 User thread 對應到一個 Kernel thread 溝通，如同上圖的 process 2，執行緒管理在使用者空間完成，此模式中使用者執行緒對 OS 不可見

- 優點：User thread 要開幾個都沒問題，且上下文切換發生在使用者空間，避免 mode switch，效能較佳
- 缺點：對多核心處理器 (Multi-processor) 來說很浪費。一個使用者執行緒如果阻塞在系統呼叫中，則整個程序都將會阻塞。

### M : N

kernel thread & user thread 的數量比為 M : N，模型提供了兩級控制，首先 user thread 對映這個可排程的輕量級程序 (LWP)，LWP 再一一對映到 kernel thread。如上圖中的 process 3

- 優點：綜合了前兩種優點，大部分的執行緒上下文切換發生在使用者空間，且多個核心執行緒又可以充分利用 CPU 資源

### Combined

如上圖中的 process 5，此程序結合 1:1 模型及 M:N 模型。開發人員可以針對不同的應用特點調節 kernel thread 的數目來達到物理並行性和邏輯並行性的最佳方案。

## CPU bound v.s. I/O bound

### CPU 密集型 (CPU bound)

aka 計算密集型，大部份時間用 CPU 來做計算、邏輯判斷等動作的<u>程序</u>稱之 CPU bound。

CPU bound 的程序一般而言**CPU 佔用率相當高**，這可能是因為任務本身不太需要訪問 I/O 設備，也可能是因為程序是多執行緒實現因此屏蔽掉了等待 I/O 的時間。

而由於主要會消耗 CPU 資源，因此，代碼運行效率至關重要。Python 這樣的腳本語言運行效率很低，就不適合計算密集型任務。

### I/O 密集型 (I/O bound)

I/O 密集型涉及到**網絡、檔案讀寫**的任務都是 I/O 密集型任務，這類任務的特點是 **CPU 消耗很少**，任務的大部分時間都在等待 I/O 操作完成。常見的 I/O 密集型任務，如 Web 應用、資料庫讀寫。

對於 I/O 密集型任務，最合適的語言就是開發效率最高的語言，腳本語言是首選。

### 執行緒池 Thread Pool

一種執行緒使用模式。執行緒過多會帶來調度開銷，進而影響快取局部性和整體性能。而執行緒池維護著多個執行緒，等待著監督管理者分配可並發執行的任務，降低在處理短時間任務時創建與銷毀執行緒的代價；能保證核心的充分利用以及防止過分調度。[5]

### 執行緒池與 CPU bound or I/O bound 的配置

CPU 密集型任務：應配置儘可能**小的**執行緒池，因為多個執行緒間頻繁進行上下文切換對於程式效能損耗較大，所以執行緒數應配置 `CPU 核心數 + 1`。

I/O 密集型任務：應配置儘可能**大的**執行緒池，當一個任務執行 IO 操作時，執行緒將被阻塞，於是處理器可以立即進行(因 CPU 消耗少) 上下文切換以便處理其他就緒執行緒。如果我們只有處理器核心數那麼多個執行緒的話，即使有待執行的任務也無法排程處理，所以執行緒數應配置 `2 * CPU`。

## Context Switch

上下文轉換指 CPU 在不同任務間切換時，必須將舊任務的狀態儲存起來，再載入新任務的儲存狀態的動作，上下文可以視為環境，以程序來說，就是**程序執行的環境**。

根據 CPU 執行的任務的不同，上下文切換分為三種：

### 程序上下文切換（Process Context Switch）

顧名思義即為不同程序的切換所產生的 cost，上下文為使用者程序傳遞給 kernel 的一些變數和暫存器值和當時的環境等，主要儲存在 PCB(Process Control Block)當中。
PCB 為在 Kernel 中針對每一個程序所建立的一個資料結構，記載該 程序的相關資訊。

![](https://imgur.com/jPEtZMl.png)

欲了解 PCB 組成，請參考
[wiki](https://en.wikipedia.org/wiki/Process_control_block)

### 執行緒上下文切換（Thread Context Switch）

有兩種狀況:

1. 前後兩個執行緒所屬不同程序，此切換可與程序上下文切換視為同一種。
2. 前後兩個執行緒屬於同一個程序，因同程序內的不同執行緒會共享資源，所以在切換時，虛擬記憶體的資源不變，只需要切換執行緒的私有數據、暫存器等不共享的數據。此切換所消耗的 cost 較小。

### 中斷上下文切換 (Interrupt handling)　

中斷上下文切換是為了響應硬件的各種事件設計出來的，中斷程序會打斷其他程序的執行。例如，當前 CPU 正在執行某程序，這個時候我們滑動鼠標，按了下鍵盤，CPU 就必須中斷正在執行的程序，轉而去響應這些硬件的事件。

可以看作就是硬體傳遞過來的這些引數和核心需要儲存的一些其他環境（主要是當前被打斷執行的程序環境）。

更仔細的釐清中斷上下文時發現：中斷上下文的過程也可能包含上方所提到的 user mode 切換到 kernel mode 的 mode switch

例如：「Timer」為一種硬體，在協助「Round robin」排程、防止單一 Process 霸佔而保護 CPU 時，會觸發

![](https://imgur.com/bZChgTB.png)

①②③④ 步驟都涉及到保存 register 或恢復 register 的操作。

② 跟 ③ 是屬於程序上下文切換——從進程 A 的上下文切換至進程 B 的上下文。

而 ① 和 ④ 步驟由 CPU 硬體完成，由中斷觸發，跟程序切換沒有直接關係。不過其實也可以把這個步驟歸稱之為上下文切換，只不過這是 CPU kernel mode 與 user mode 之間的上下文切換（mode switch）。[6]

### 小結

- CPU 的狀態可以歸納為以下三個
  - Kernel Mode，跑程序上下文，程序運行於 Kernel space
  - Kernel Mode，跑中斷上下文，硬體運行於 Kernel space
  - User Mode，跑 user 程序，程序運行於 User space
- 如果僅僅是中斷（例如系統調用），而未發生進程切換，並不需要保存進程的上下文信息(process context switch)。

### 補充：上下文切換執行於 Kernel space 的原因

大部分的 OS 使用虛擬記憶體（virtual memory），用以增加實體記憶體的使用效率。每個程序都有自己的虛擬地址空間（virtual address space），透過 CPU 中的內存管理單元（memory management unit，MMU）標識當前正在運行程序的地址轉換映射表（address translation map），將虛擬位址轉為物理位址。

![](https://imgur.com/v5Eup0o.png)

很多系統以頁表（page table）的方式來實現這些映射。在當前程序將 CPU 讓給另外一個程序時（即發生了上下文切換（context switch）），kernel 會將這些暫存器(regs)和指針(k-stack)加載到新進程的轉換映射表中。而 MMU 寄存器是受特權保護的(ring 0)，只能在 kernel mode 下訪問。這就確保一個程序只能引用它自己空間中的地址，而不能訪問或者修改其他進程的地址空間。

## 參考資料

[1][https://www.geeksforgeeks.org/difference-between-multitasking-multithreading-and-multiprocessing/](https://www.geeksforgeeks.org/difference-between-multitasking-multithreading-and-multiprocessing/)  
[2][https://zh.wikipedia.org/wiki/%E5%88%86%E7%BA%A7%E4%BF%9D%E6%8A%A4%E5%9F%9F](https://zh.wikipedia.org/wiki/%E5%88%86%E7%BA%A7%E4%BF%9D%E6%8A%A4%E5%9F%9F)  
[3][https://stackoverflow.com/questions/10484355/what-the-difference-between-lightweight-process-and-thread](https://stackoverflow.com/questions/10484355/what-the-difference-between-lightweight-process-and-thread)  
[4][https://www.tutorialspoint.com/lightweight-process-lwp](https://www.tutorialspoint.com/lightweight-process-lwp)  
[5][https://www.easyatm.com.tw/wiki/%E5%9F%B7%E8%A1%8C%E7%B7%92%E6%B1%A0](https://www.easyatm.com.tw/wiki/%E5%9F%B7%E8%A1%8C%E7%B7%92%E6%B1%A0)  
[6][http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf)
