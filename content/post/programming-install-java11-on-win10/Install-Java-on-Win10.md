---
title: "[Java] Install Java on Win11"
tags:
  - Java
categories:
  - Programming
date: 2020-07-01 17:41:00
slug: install-java10-on-win11
---

## 選擇 Java 版本

Java 更新速度之快，截至今天已經出到 Java 14 了！本來是想隨波逐流的裝 Java 8 但參考了這篇[文章](https://medium.com/swlh/why-do-people-stick-with-java8-acb95ef65f0c)後，還是決定裝 11。

<!--more-->

JDK 可以從 [AdoptOpenJDK](https://adoptopenjdk.net/releases.html) 下載，AdoptOpenJDK 是 Java 使用者社群建立的，致力於倡導 OpenJDK，上面支援的作業系統以及對應的 JDK 版本 (8~11) 最齊全。

## 安裝

在網站上下載 .msi 檔後執行，在客製安裝頁面一併把第三個選項 `Set JAVA_HOME variable` 設起來。

![](https://imgur.com/Ue4j8TH.png)

## 設定 PATH 系統變數

設定 path 環境變數的目的是為了要上作業系統找到 Java 在哪。
編輯系統變數欄位中的 `Path`，並加入 `%JAVA_HOME%` 的選項，然後把它移到最前面。當電腦中安裝兩個以上的不同版本的 Java，則環境變數放在前面的會先被執行。

![](https://imgur.com/Em1c46E.png)

## 確認安裝

開啟 CMD 並下下方兩個指令，確認安裝是否成功。

```
$ java -version
$ javac -version
```

![](https://imgur.com/G8DUM6M.png)

## Hello World!

```java
//Hello.java
public class Hello {
    public static void main(String []args) {
        System.out.println("Hello World!")
    }
}
```

{{< notice note >}}
關於程式內部詳細的解說，之後的文章會記錄。
(不過本來因為數據中台的專案要用 Map Reduce 所以學的，但因為工作又有小變動，所以不知道何年何月才可以再繼續看 Java，先暫緩了 :stuck_out_tongue_closed_eyes: )
{{< /notice >}}

## 編譯檔案

使用 javac 公用程式來編譯檔案。

```
$ javac Hello.java
```

完成後在目錄下就會出現一個 `Hello.class` 的 java 位元碼 (bytecode)。

![](https://imgur.com/YHyPdOX.png)

## 執行

```
$ java Hello
```

使用 java 工具程式執行，不須帶附檔名，Java 會根據類別名稱自動載入 .class 檔案。
![](https://imgur.com/M4ZkQXa.png)
