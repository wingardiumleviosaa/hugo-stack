---
title: "[golang] 使用 tebeka selenium 爬蟲模擬操控網頁"
tags:
  - Golang
categories:
  - Programming
date: 2022-01-10 16:03:00
slug: golang-tebeka-selenium
---

最近碰到一個需從網頁去擷取圖片的需求，原本拿到的 sample code 是由 python 的 selenium 爬的，但後來發現有大神將此包改寫成 golang [tebeka/selenium](https://github.com/tebeka/selenium)，所以就試著改寫看看，在此紀錄一下。

<!--more-->

## 目標網頁爬取需求

![](https://imgur.com/xlxg7kZ.png)

目標網頁是一個爐溫監控的網站，任務是爬取指定產品所經的迴焊爐的生產歷史紀錄。需要模擬的步驟如下

1. 查詢頁面需先輸入的參數有：
   - 下拉式選單選取線別
   - 文字框時間範圍 （開始與結束）
   - 文字框產品編號
2. 點選查詢按鈕
3. 回傳搜尋結果表格，點選表格的每一列的任意位置/欄位
4. 彈出一個 modal 視窗，內容為被點選列的詳細爐溫圖表

![](https://imgur.com/TBkYGVS.png)

5. 擷取螢幕並裁切圖片到目標範圍存到本機
6. 點選 OK 按鈕以關閉 modal 視窗
7. 繼續點選下列，重複 3 ～ 6 步驟，直到表格的最後一列資訊

## 運行環境

- Ubuntu v20.04
- Golang v1.15
- Google Chrome v96.0.4664.110

## 事前準備

須在 ubuntu 下載 Chrome 以及相對應的 chromedriver。

1. 更新系统

```
sudo apt-get update
```

2. 安裝相關的必要套件

```
sudo apt-get install libxss1 libappindicator1 libindicator7
# 安裝 xvfb 以便可以用 headless 模式（跑在背景，不開啟瀏覽器）運行 Chrome
sudo apt-get install xvfb
```

3. 下載安裝包

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

4. 安装 chrome

```
sudo apt-get install dpkg
# 安装chrome，安裝過程中由於缺少一些依賴而報錯是正常的
sudo dpkg -i google-chrome*.deb
#自動安裝上一步缺少的依賴
sudo apt-get install -f
```

5. 下載 chromedriver
   確認安裝的 Chrome 版本，並下載與之匹配的 chromedriver。

![](https://imgur.com/sO3eqDc.png)

```
wget -N https://chromedriver.storage.googleapis.com/96.0.4664.110/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
# 移動 driver 到指定資料夾
sudo mv -f /home/ula/Downloads/chromedriver ~
```

## 程式碼

```go
package main

import (
	"fmt"
	"image"
	"image/png"
	"io/ioutil"

	"os"
	"strconv"
	"time"

	"github.com/anaskhan96/soup"
	"github.com/tebeka/selenium"
	"github.com/tebeka/selenium/chrome"
)

func main() {
	// 1. enable selenium service
	opts := []selenium.ServiceOption{
		//selenium.Output(os.Stderr), // Output debug information to STDERR
	}
	// lauch a webdriver instance
	service, err := selenium.NewChromeDriverService("/home/nexdata/chromedriver", 9515, opts...)
	if err != nil {
		fmt.Printf("Error starting the ChromeDriver server: %v", err)
	}
	// delay service shutdown
	defer service.Stop()

	// 2. call browser
	caps := selenium.Capabilities{
		"browserName": "chrome",
	}
	// set chrome arguments
	chromeCaps := chrome.Capabilities{
		Args: []string{
			// "--headless",   // do not open the browser (run in background)
			"--no-sandbox", //  allow non-root to execute chrome
			"--disable-deb-shm-usage",
			// "--window-size=1400,1500",
			"--start-maximized",
		},
	}
	caps.AddChrome(chromeCaps)

	// connect to the webdriver instance which running locally
	wd, err := selenium.NewRemote(caps, "http://127.0.0.1:9515/wd/hub")
	if err != nil {
		fmt.Printf("connect to the webDriver faild: %v", err)
	}
	// delay closing Chrome
	defer wd.Quit()

	// connect to the target website
	if err := wd.Get("http://10.90.1.100:9000/productionRecordHistory"); err != nil {
		fmt.Printf("connect to the reflow server failed: %v", err)
	}
	// select the line
	sel_line, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/div[1]/select")
	line, _ := selenium.Select(sel_line)
	line.SelectByValue("6")

	// select start date
	start_date, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/div[2]/table/tbody/tr[2]/td[1]/div/input")
	start_date.Clear()
	start_date.SendKeys("11/28/2021 12:00 PM")

	// select end date
	end_date, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/div[2]/table/tbody/tr[2]/td[2]/div/input")
	end_date.Clear()
	end_date.SendKeys("11/30/2021 12:00 AM")

	// input serial number
	barcode, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/table/tbody/tr[2]/td[3]/input")
	barcode.SendKeys("TBCBB2039913")

	// click query button
	query_button, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/p/button")
	// args := []interface{}{query_button}
	// wd.ExecuteScript("arguments[0].click();", args)
	query_button.SendKeys(selenium.EnterKey)

	time.Sleep(time.Duration(3) * time.Second)

	// parse to get the number of the column
	html_parse, _ := wd.PageSource()
	doc := soup.HTMLParse(html_parse)
	table := doc.Find("div", "class", "react-bs-container-body").FindAll("tr")

	// enter to the result by the order of the columns
	for i := 0; i < len(table); i++ {
		ele := "/html/body/div/div/div/div/section[2]/div[2]/div/div/div[2]/div[2]/div/div/div/div[2]/div[2]/table/tbody/tr[" + strconv.Itoa(i+1) + "]/td[1]"
		enter_modal, _ := wd.FindElement(selenium.ByXPATH, ele)
		enter_modal.Click()
		pop_up, _ := wd.WindowHandles()
		wd.SwitchWindow(pop_up[0])
		//fmt.Println(pop_up)
		time.Sleep(time.Duration(1) * time.Second)
		scrnshot, _ := wd.Screenshot()
		ioutil.WriteFile("test"+strconv.Itoa(i+1)+".png", scrnshot, 0666)
		// get the modal location and size to crop the image
		modal, _ := wd.FindElement(selenium.ByClassName, "modal-body")
		loc, _ := modal.Location()
		sz, _ := modal.Size()
		// fmt.Println(loc)
		// fmt.Println(sz)
		file, _ := os.Open("./test" + strconv.Itoa(i+1) + ".png")
		defer file.Close()
		img, _ := png.Decode(file)
		sub_image := img.(interface {
			SubImage(r image.Rectangle) image.Image
		}).SubImage(image.Rect(loc.X, loc.Y, loc.X+sz.Width, loc.Y+sz.Height))
		file, _ = os.Create("./crop" + strconv.Itoa(i+1) + ".png")
		png.Encode(file, sub_image)

		time.Sleep(time.Duration(5) * time.Second)

		ok_button, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div[2]/div/div[2]/div/div/div[3]/div/div[2]/button[4]")
		// args = []interface{}{ok_button}
		// wd.ExecuteScript("arguments[0].click();", args)
		ok_button.SendKeys(selenium.EnterKey)

		time.Sleep(time.Duration(2) * time.Second)
	}

}
```

中途遇到幾個問題在這紀錄一下，

1.  button click() 沒反應
    中間操作時，button 元素找到後進行 Click() 點擊，但是沒有反應。在爬文後還是沒找出確切原因，但有兩個方法可以解決： - 直接調用 javascript 的點擊事件
    `go
	query_button, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/p/button")
	args := []interface{}{query_button}
	wd.ExecuteScript("arguments[0].click();", args)
	`

        - 使用 enter 操作
         ```go
        query_button, _ := wd.FindElement(selenium.ByXPATH, "/html/body/div/div/div/div/section[2]/div[1]/div/div/div[2]/p/button")
        query_button.SendKeys(selenium.EnterKey)
        ```

2.  解析 HTML 原始碼
    因為要取得回傳表格的 row 數，如果用 selenium package 有點難爬。後來發現大部份的人如果要解析及取得 HTML 原始碼各個標籤的元素資料，都會用 python 的靜態爬取的爬蟲 Beatifulsoup 套件，而 golang 也有大神改寫成 [soup](https://github.com/anaskhan96/soup)。
    目標需要爬取的元素包在 `react-bs-container-body` class 中，如下圖。爬取整個 html 後，用 Find 找指定 class 後再用 FindAll 篩選 tr 標籤以取得 row 數（len(table)）。

![](https://imgur.com/jF71MfF.png)

```go
// parse to get the number of the column
html_parse, _ := wd.PageSource()
doc := soup.HTMLParse(html_parse)
table := doc.Find("div", "class", "react-bs-container-body").FindAll("tr")
```

3. tebeka/selenium 沒有包 select function
   所幸 issue 中有大神把 SeleniumHQ Java code 改寫成 golang，請參考這個 [pull request](https://github.com/tebeka/selenium/pull/238)。可以無痛參考使用 T_T

4. tebeka/selenium 的 Screenshot 無法指定範圍
   所以只好將就，先把整個畫面截圖後儲存，再另外使用圖片處理的套件 image，將圖片讀取後另外擷取再儲存。 （速度慢但目前無其他方法 Q_Q）

## Reference

- https://leileiluoluo.com/posts/golang-selenium.html
- https://blog.epoch.tw/2021/05/28/%E4%BD%BF%E7%94%A8-Go-%E6%93%8D%E4%BD%9C-Chrome-%E7%80%8F%E8%A6%BD%E5%99%A8/
- https://blog.csdn.net/shjsfx/article/details/106006255
