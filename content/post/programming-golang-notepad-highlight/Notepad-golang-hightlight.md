---
title: 為 Notepad++ 加上 golang 語法高亮
tags:
  - Golang
categories:
  - Programming
date: 2022-01-05 21:05:00
slug: golang-highlight-on-notepad
---

原生 Notepad++ 沒有支援 golang 的語法，在此紀錄一下要怎麼在 Notepad 中加入自定義的 golang 程式語言的語法高亮(使用暗黑模式 Obsidian)。目前 notepad 使用的版本為 v8.1.9.3。

<!--more-->

## 定義使用者自訂語言

在工具列中點選 `語言>自訂程式語法>開啟自訂語法樣式資料夾`，在該資料下新增一個 `userDefineLang-Go-Obsidian.xml` 檔案，內容為以下 xml 代碼。

```xml
<NotepadPlus>
    <UserLang name="Go" ext="go" udlVersion="2.1">
        <Settings>
            <Global caseIgnored="no" allowFoldOfComments="yes" foldCompact="no" forcePureLC="0" decimalSeparator="0" />
            <Prefix Keywords1="no" Keywords2="no" Keywords3="no" Keywords4="no" Keywords5="no" Keywords6="no" Keywords7="no" Keywords8="no" />
        </Settings>
        <KeywordLists>
            <Keywords name="Comments">00// 01 02 03/* 04*/</Keywords>
            <Keywords name="Numbers, prefix1"></Keywords>
            <Keywords name="Numbers, prefix2">0X 0x 0</Keywords>
            <Keywords name="Numbers, extras1">A B C D E F a b c d e f</Keywords>
            <Keywords name="Numbers, extras2"></Keywords>
            <Keywords name="Numbers, suffix1"></Keywords>
            <Keywords name="Numbers, suffix2"></Keywords>
            <Keywords name="Numbers, range"></Keywords>
            <Keywords name="Operators1">! % &amp; ( ) * + , - . / : ; &lt; = &gt; [ ] ^ { | }</Keywords>
            <Keywords name="Operators2"></Keywords>
            <Keywords name="Folders in code1, open"></Keywords>
            <Keywords name="Folders in code1, middle"></Keywords>
            <Keywords name="Folders in code1, close"></Keywords>
            <Keywords name="Folders in code2, open"></Keywords>
            <Keywords name="Folders in code2, middle"></Keywords>
            <Keywords name="Folders in code2, close"></Keywords>
            <Keywords name="Folders in comment, open"></Keywords>
            <Keywords name="Folders in comment, middle"></Keywords>
            <Keywords name="Folders in comment, close"></Keywords>
            <Keywords name="Keywords1">_ false iota nil true</Keywords>
            <Keywords name="Keywords2">break case continue default defer else fallthrough for go goto if import range return select switch</Keywords>
            <Keywords name="Keywords3">append cap close complex copy delete imag len make new panic print println real recover</Keywords>
            <Keywords name="Keywords4">ComplexType FloatType IntegerType Type Type1 bool byte complex128 complex64 error float32 float64 int int16 int32 int64 int8 rune string uint uint16 uint32 uint64 uint8 uintptr</Keywords>
            <Keywords name="Keywords5">chan const func interface map package struct type var</Keywords>
            <Keywords name="Keywords6"></Keywords>
            <Keywords name="Keywords7"></Keywords>
            <Keywords name="Keywords8"></Keywords>
            <Keywords name="Delimiters">00&quot; 01\ 02&quot; 03&apos; 04\ 05&apos; 06` 07 08`</Keywords>
        </KeywordLists>
        <Styles>
            <WordsStyle name="DEFAULT" fgColor="E0E2E4" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="COMMENTS" fgColor="66747B" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="LINE COMMENTS" fgColor="66747B" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="NUMBERS" fgColor="FFCD22" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS1" fgColor="678CB1" bgColor="293134" fontName="" fontStyle="1" nesting="0" />
            <WordsStyle name="KEYWORDS2" fgColor="93C763" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS3" fgColor="A082BD" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS4" fgColor="5AB9BE" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS5" fgColor="93C763" bgColor="293134" fontName="" fontStyle="1" nesting="0" />
            <WordsStyle name="KEYWORDS6" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS7" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="KEYWORDS8" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="OPERATORS" fgColor="E8E2B7" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="FOLDER IN CODE1" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="FOLDER IN CODE2" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="FOLDER IN COMMENT" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS1" fgColor="EC7600" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS2" fgColor="FF8409" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS3" fgColor="D39745" bgColor="293134" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS4" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS5" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS6" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS7" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
            <WordsStyle name="DELIMITERS8" fgColor="000000" bgColor="FFFFFF" fontName="" fontStyle="0" nesting="0" />
        </Styles>
    </UserLang>
</NotepadPlus>
```

## 啟用自動完成功能

在 notepad 的安裝目錄下的 `plugins\APIs\` 目錄，新增 `go.xml` 檔案並複製以下代碼到該檔案

```xml
<NotepadPlus>
	<AutoComplete language="Go">
		<KeyWord name="_" />
		<KeyWord name="false" />
		<KeyWord name="iota" />
		<KeyWord name="nil" />
		<KeyWord name="true" />
		<KeyWord name="break" />
		<KeyWord name="case" />
		<KeyWord name="continue" />
		<KeyWord name="default" />
		<KeyWord name="defer" />
		<KeyWord name="else" />
		<KeyWord name="fallthrough" />
		<KeyWord name="for" />
		<KeyWord name="go" />
		<KeyWord name="goto" />
		<KeyWord name="if" />
		<KeyWord name="import" />
		<KeyWord name="range" />
		<KeyWord name="return" />
		<KeyWord name="select" />
		<KeyWord name="switch" />
		<KeyWord name="append" func="yes">
			<Overload retVal="[]Type" >
				<Param name="slice []Type" />
				<Param name="elems ...Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="cap" func="yes">
			<Overload retVal="int" >
				<Param name="v Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="close" func="yes">
			<Overload retVal="" >
				<Param name="c chan<- Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="complex" func="yes">
			<Overload retVal="ComplexType" >
				<Param name="r" />
				<Param name="i FloatType" />
			</Overload>
		</KeyWord>
		<KeyWord name="copy" func="yes">
			<Overload retVal="int" >
				<Param name="dst" />
				<Param name="src []Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="delete" func="yes">
			<Overload retVal="" >
				<Param name="m map[Type]Type1" />
				<Param name="key Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="imag" func="yes">
			<Overload retVal="FloatType" >
				<Param name="c ComplexType" />
			</Overload>
		</KeyWord>
		<KeyWord name="len" func="yes">
			<Overload retVal="int" >
				<Param name="v Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="make" func="yes">
			<Overload retVal="Type" >
				<Param name="Type" />
				<Param name="size IntegerType" />
			</Overload>
		</KeyWord>
		<KeyWord name="new" func="yes">
			<Overload retVal="*Type" >
				<Param name="Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="panic" func="yes">
			<Overload retVal="" >
				<Param name="v interface{}" />
			</Overload>
		</KeyWord>
		<KeyWord name="print" func="yes">
			<Overload retVal="" >
				<Param name="args ...Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="println" func="yes">
			<Overload retVal="" >
				<Param name="args ...Type" />
			</Overload>
		</KeyWord>
		<KeyWord name="real" func="yes">
			<Overload retVal="FloatType" >
				<Param name="c ComplexType" />
			</Overload>
		</KeyWord>
		<KeyWord name="recover" func="yes">
			<Overload retVal="interface{}" >
			</Overload>
		</KeyWord>
		<KeyWord name="ComplexType" />
		<KeyWord name="FloatType" />
		<KeyWord name="IntegerType" />
		<KeyWord name="Type" />
		<KeyWord name="Type1" />
		<KeyWord name="bool" />
		<KeyWord name="byte" />
		<KeyWord name="complex128" />
		<KeyWord name="complex64" />
		<KeyWord name="error" />
		<KeyWord name="float32" />
		<KeyWord name="float64" />
		<KeyWord name="int" />
		<KeyWord name="int16" />
		<KeyWord name="int32" />
		<KeyWord name="int64" />
		<KeyWord name="int8" />
		<KeyWord name="rune" />
		<KeyWord name="string" />
		<KeyWord name="uint" />
		<KeyWord name="uint16" />
		<KeyWord name="uint32" />
		<KeyWord name="uint64" />
		<KeyWord name="uint8" />
		<KeyWord name="uintptr" />
		<KeyWord name="chan" />
		<KeyWord name="const" />
		<KeyWord name="func" />
		<KeyWord name="interface" />
		<KeyWord name="map" />
		<KeyWord name="package" />
		<KeyWord name="struct" />
		<KeyWord name="type" />
		<KeyWord name="var" />
	</AutoComplete>
</NotepadPlus>
```

## 重啟應用程式並設置對應語言以及樣式

1. 重啟 Notepad
2. 將樣式改成 `Obsidian`

![](https://imgur.com/jYuRdUj.png)

![](https://imgur.com/U3MZ3Id.png)

3. 開啟一個 golang 的檔案，將語法設為自定義的 `Go`

![](https://imgur.com/9dPX4HZ.png)

並可成功顯示高亮語法

![](https://imgur.com/D7ORjqh.png)

## Reference

- https://www.reddit.com/r/golang/comments/3vgrwn/go_syntax_highlighting_and_builtin_function/
- https://github.com/haikubox/configs/tree/master/notepad%2B%2B
