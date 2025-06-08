---
title: "[Java] JVM 架構"
tags:
  - Java
categories:
  - Programming
date: 2020-07-01 16:41:00
slug: java-jvm-structure
---

前一篇介紹到 Java 程式經過編譯後會產生 .class 檔 (bytecode)，只能運行在 JVM 上，而 JVM 在運算時，如同電腦需要記憶體儲存運算所需的各種資料及指令，這篇文章將紀錄 JVM 的架構。

<!--more-->

## JVM Structure

![](https://imgur.com/qbAYLHV.png) [1]

上圖為 JVM 執行一個 Java 程式的過程：

### Class Loader

用於將編譯好的 `.class` 文件 － **Java Bytecode** 加載到 Runtime Data Areas。

### Execution Engine

用於執行 Java Bytecode 或是 [native method](https://baike.baidu.com/item/Native%20methods).

Java Bytecode 是以人類仍然可以理解的語言而不是機器語言編寫的。因此，Execution Engine 透過以下兩種方式將字節碼轉為 JVM 中的機器語言。

1. Interpreter
2. JIT (Just-In-Time) Compiler
<table><tr><td bgcolor="#FAFAFA">
<b>什麼時候用 Interpreter 什麼時候用 Compiler?</b><br>
The application code is initially interpreted, but the JVM monitors which part of bytecode are frequently executed and translates them to machine code for direct execution on the hardware. 
For bytecode which is executed only a few times, interpret it will saves the compilation time and reduces the initial latency; 
For frequently executed bytecode, JIT compilation will be used after an initial phase of slow interpretation. 
By interpreting the initial code, execution statistics can be collected before compilation, which helps to perform better optimization.[3]
</td></tr></table>

![](https://imgur.com/H8aD6Eh.png) [2]

![](https://imgur.com/iMbW3Un.png) [3]

### Runtime Data Areas

JVM 運行時所需的的記憶體區塊，總共可以分為六區

- PC Register, JVM Stack 以及 Native Method Stack 依賴於**單個**執行緒的啟動/結束來建立與銷毀。
- Heap, Metaspace(Metod Area & Runtime Constant Pool) 是隨著 JVM 的啟動而存在，為**全部**的執行緒共享。

![](https://imgur.com/ZumQcTh.png) [4]

#### PC Register

Program Counter Register，紀錄當前執行緒所執行到的 bytecode 中的指令位址。為了執行緒切換後能恢復到正確的執行位置，每條執行緒都需要有一個獨立的 program counter，各執行緒之間獨立儲存。

#### JVM Stack

用來放 frame，每個 method 在執行的同時都會建立一個 stack frame，用於儲存局部變數表 (Local Variable Array)、運算元堆疊 (Operand Stack) 和動態連接 (Dynamic Linking) 以及返回值(Return Value)。每個 method 從呼叫直至結束 (return)，都對應著一個 stack frame 在此區塊中入/出 (push/pop) stack 的過程。  
JVM Statck 的大小可以是固定的，或是動態擴展的。如果 thread 需要一個比固定大小大的 Stack，會發生 **StackOverflowError**；如果動態擴展 Stack 時沒有足夠的記憶體空間，則會發生 **OutOfMemoryError**。

    ![](https://imgur.com/WDOImGT.png)

    ![](https://imgur.com/GT53vZO.png)
    [5]

##### 局部變量表 (Local Variable Array)

局部變量表用於存放方法參數和方法內部定義的局部變量，其大小在編譯期(.class 前)就被確定。  
Java 代碼 `int a=0; int b=1; int c=2;` 對應的局部變量表如下：

| Start | Length | Slot | Name | Signature |
| ----- | ------ | ---- | ---- | --------- |
| 2     | 12     | 0    | a    | I         |
| 4     | 10     | 1    | b    | I         |
| 6     | 8      | 2    | c    | I         |

- Start: 變量偏移量。
- Length: 作用域範圍長度。 `Start,Start+Length` 就是該變量的作用域。
- Slot: 變量槽，一個 Slot (一行) 能存儲 32bit 的 primitive type、reference type、returnAddress 數據，long/dobule 則需要兩個 Slot。

##### 運算元棧 (Operand Stack)

用 push,pop 操作數據。operand stack 只是一個臨時的計算過程，要用到 Local variable table 裡面的值，然後得出結果，放入到  local variables 區。

```java
// Java 代碼
int a=1;
int b=2;
int c=a+b;
```

</br>

```
// 運算元棧
0: iconst_1 // push 1到操作棧。大於5的int值會用到 bipush <i> 指令。
1: istore_0 // pop 頂元素，存儲到index=0的本地變量。
2: iconst_2 // push 2 到操作棧
3: istore_1 // pop棧頂元素，存儲到index=1的本地變量。
4: iload_0 // 把index=0的本地變量加載到棧頂
5: iload_1 // 把index=1的本地變量加載到棧頂
6: iadd // 把棧頂兩個數pop出來相加，並把結果存放到棧頂
7: istore_2 // 結果存儲到index=2的本地變量

```

##### 動態連接 (Dynamic Linking)

指符號引用（Symbolic References）轉換成為直接引用（Direct References）的過程。每個 Frame 內部都包含一個指向 runtime constant pool 的引用來支援當前方法的程式碼。

- 符號引用：以一組符號來描述所引用的目標，符號可以是任何形式的字面量，只要使用時能無歧義地定位到目標即可。方法名，類名，字段名都是符號引用。

```java
int i = 1; //把整數 1 賦值給 int 型變量 i，整數 1 就是字面量，
String s = "abc"; //abc 是字面量。
```

- 直接引用：可以是直接指向目標的指針、相對偏移量或是一個能間接定位到目標的句柄。如果有了直接引用，那么引用的目標一定是已經存在於內存中。

#### Native Method Stack

此區塊主要執行 native method。

#### Heap

用於儲存物件的實例(object instance)和陣列，也是 GC 管理的主要區域，因此也被稱為 GC 堆 — Garbage Collected Heap。Heap 是在 JVM 啟動時建立的，為所有執行緒共有。堆的大小可以固定，也可以擴大縮小且不需要是連續空間。

##### 堆記憶體分配

現在的 GC 都採 generation-collect 分代收集算法，所以 Heap 記憶體可劃分為：
![](https://imgur.com/AsMKt4b.png) [6]

- Young Generation（年輕代）
- Old Generation（老年代）
  其中 Young generation 可以進一步分為 3 個區塊: Eden space， Survivor 1 和 Survivor 2

##### Garbage Collection

垃圾回收(garbage collection)是指將佔記憶體空間(heap)的不再被參考的物件
清除的過程。

年輕代回收：又稱小型回收 (minor collection)，執行時機是當年輕代的空間滿的時候會觸發。當新物件產生時會放置在 Eden space，當 Eden space 滿了，JVM 會啟動 Minor GC，把存活的物件往 Survivor 1 或 2 移及以原本在 Survivor 1 或 2 的存活物件往另一個 Survivor 空間移。當 JVM 執行多次 Minor GC 後，會把符合條件的存活物件往 Tenured generation 移，這個過程我們稱為 Minor GC。

老年代或永久代回收稱為完整回收(full collection)或主要回收(major collection)。當 old generation 滿了時，JVM 會回收 Tenured generation 的空間，稱為 Major/Full GC。

##### heap 大小對 GC 的影響

空間比較小的時候，打掃動作比較快，但是相對垃圾累積也比較快滿，所以打掃需要比較頻繁。相反地，空間大，需要打掃的頻率比較低，但是一打掃起來就很費時。通常就是在「時間」、「空間」、「頻率」三者之間作取捨而已。
Minor GC 和 Major/Full 都是 "Stop the World" 事件，只是 Minor GC 時間非常短(幾百 milli-seconds)，使用者較不容易察覺;而 Full GC 時間相對上長很多，且 heap size 愈大時間愈久；因此應儘量避免或減少 Full GC 發生。

若 Heap space 沒空間存放新建立的物作，則 JVM 會丟出 OutOfMemoryError 或 java.lang.OutOfMemoryError heap space

#### MetaSpace (Java8 以前以後)

**Java 7**

![](https://imgur.com/oturBUp.png)

**Java 8**

![](https://imgur.com/AsMKt4b.png)[6]

下面這張能更清楚 java 8 的改變:

![](https://imgur.com/AoP8OjJ.png)[7]

原先存在 heap 用來存放存放 byte code, JIT information, class metadata 以及 static 的變數, 方法的永久代 PermGen 被移到 native memory 的 metaspace 了。

## Reference

[1]http://mauryasunil007.blogspot.com/2016/04/understanding-jvm-internal-architecture.html  
[2]https://www.datawareventures.com/single-post/2015/10/09/Field-Specialization-Just-another-JIT  
[3]https://stackoverflow.com/questions/1326071/is-java-a-compiled-or-an-interpreted-programming-language  
[4]https://blog.jamesdbloom.com/JVMInternals.html  
[5]https://alvinalexander.com/scala/fp-book/recursion-jvm-stacks-stack-frames/  
[6]https://docs.deistercloud.com/content/Axional%20development%20libraries.20/Axional%20Server.4/Tunning.xml?embedded=true  
[7]https://stackoverflow.com/questions/39675406/difference-between-metaspace-and-native-memory-in-java
