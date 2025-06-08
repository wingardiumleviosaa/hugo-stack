---
title: "[Java] What are JDK, JRE, JDM ?"
tags:
  - Java
categories:
  - Programming
date: 2020-07-01 16:38:00
slug: java-jdk-jre-jvm-intro
---

## 比較

**JDK（Java Development Kit，Java 開發工具包）**  
是用來編譯、調試 Java 程序的開發工具包。組成包含 Java 工具（javac/java/jdb 等）和 Java 基礎的類庫。

<!--more-->

**JRE（Java Runtime Environment， Java 運行環境）**  
所有的 java 程序都要在 JRE 下才能夠運行。組成包含 JVM 和 Java 核心類庫和支持文件。

**JVM（Java Virtual Machine， Java 虛擬機）**  
是 JRE 的一部分，虛擬機代表通過在實體計算機上模擬計算機功能，JVM 有自己的硬體架構以及指令系統，它的工作就是解釋（編譯）自己的指令集（即字節碼）並映射到本地的 CPU 指令集和 OS 的系統調用，不同的操作系統會有不同的 JVM 映射規則，實現 Java 的跨平台特性。
組成包含字節碼指令集、寄存器、棧、垃圾回收堆和存儲方法域...等。

![](https://imgur.com/iOWaxCJ.png)
[1]

## 小結

使用 JDK 開發 JAVA 程序，再通過 JDK 中的編譯程序（javac）將 Java 程序編譯成 Java Byte Code（字節碼），在 JRE 上運行這些字節碼，JVM 會解析並映射到真實操作系統的 CPU 指令集和 OS 的系統調用。 [2]

![](https://imgur.com/WYgyqID.png)
[3]

## Java SE、Java ME、Java EE

隨著  Java  的應用領域擴展，而發展出的不同的應用開發平台。
- Java SE：Java Platform, Standard Edition，Java  各應用平臺的基礎，可以分作四個主要的部份：JVM、JRE、JDK 與 Java 語言。
- Java EE：Java Platform，Enterprise Edition，是在  Java SE  的基礎上構建的，提供  Web  服務、元件模型、管理和通訊  API，
- Java ME：Java Platform，Micro Edition，是  Java  平臺版本中最小的一個，目的是作為小型數位設備上開發及部署應用程式的平臺，像是消費性電子產品或嵌入式系統等。

##   小結

Java SE  是做電腦上執行的軟體。
Java EE  是用來做網站的-（我們常見的 JSP 技術）
Java ME  是做手機軟體的。

##  source

- [1] [difference-between-jdk-jre-and-jvm](https://www.javatpoint.com/difference-between-jdk-jre-and-jvm)
- [2] https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/654140/
- [3][c-and-java-virtual-machine-code-execution]( https://stackoverflow.com/questions/21810538/c-and-java-virtual-machine-code-execution)
- [4] https://www.ithome.com.tw/article/124269
