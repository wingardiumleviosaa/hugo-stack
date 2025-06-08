---
title: "[Python & Golang] Selenium Screenshot to the Specific Area"
tags:
  - Python
  - Golang
categories:
  - Programming
date: 2022-01-26 17:46:00
slug: screenshot-specific-area-by-selenium
---

本篇文章紀錄如何使用 python 以及 golang 改寫的 selenium，螢幕截圖指定網址的特定範圍並存成圖片。

<!--more-->

## 目標

預計爬取 [selenium 官網](https://www.selenium.dev/projects/) 的 project 頁面，並擷取指定範圍存成圖片。
![](https://imgur.com/ZMc0iPJ.png)

## Python Sample Code

```python
from PIL import Image
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

print("開始爬取")

if __name__ == "__main__":
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--no-sandbox')
    options.add_argument('lang=zh_TW.UTF-8')
    driver = webdriver.Chrome('./chromedriver', options=options)
    driver.set_window_size(1400, 1500) # 設定視窗大小

    driver.get("https://www.selenium.dev/projects/")
    time.sleep(1)

    driver.save_screenshot("./scrnsht.png")

    # crop curve table only
    ele = driver.find_elements_by_xpath("/html/body/div/main/div[1]/div")

    left = ele[0].location['x']
    top = ele[0].location['y']
    right = left + ele[0].size['width']
    bottom = top + ele[0].size['height']

    im = Image.open("./scrnsht.png")
    im = im.crop((left, top, right, bottom))
    im.save("./crop.png")

    driver.quit()

print("爬取完成")
```

## Golang(tebeka/selenium) Sample Code

```go=
package main

import (
	"fmt"
	"image"
	"image/png"
	"io/ioutil"

	"os"
	"time"

	"github.com/tebeka/selenium"
	"github.com/tebeka/selenium/chrome"
)

func main() {

	opts := []selenium.ServiceOption{
		selenium.Output(os.Stderr), // Output debug information to STDERR
	}
	service, err := selenium.NewChromeDriverService("/home/nexdata/chromedriver", 9515, opts...)
	if err != nil {
		fmt.Printf("Error starting the ChromeDriver server: %v", err)
	}
	defer service.Stop()

	// call browser
	caps := selenium.Capabilities{
		"browserName": "chrome",
	}
	// set chrome arguments
	chromeCaps := chrome.Capabilities{
		Args: []string{
			"--headless",   // do not open the browser (run in background)
			"--no-sandbox", //  allow non-root to execute chrome
			"--disable-deb-shm-usage",
			"--window-size=1400,1500",
			//"--start-maximized",
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
	if err := wd.Get("https://www.selenium.dev/projects/"); err != nil {
		fmt.Printf("connect to the reflow server failed: %v", err)
	}

	time.Sleep(time.Duration(1) * time.Second)

	ele, err := wd.FindElement(selenium.ByXPATH, "/html/body/div/main/div[1]/div")
	if err != nil {
		fmt.Printf("target element doesn't exist!")
	}
	scrnsht, _ := wd.Screenshot()
	ioutil.WriteFile("scrnsht.png", scrnsht, 0666)
	loc, _ := ele.Location()
	sz, _ := ele.Size()
	// fmt.Println(loc)
	// fmt.Println(sz)
	file, _ := os.Open("./scrnsht.png")
	defer file.Close()
	img, _ := png.Decode(file)
	sub_image := img.(interface {
		SubImage(r image.Rectangle) image.Image
	}).SubImage(image.Rect(loc.X, loc.Y, loc.X+sz.Width, loc.Y+sz.Height))
	file, _ = os.Create("./crop.png")
	png.Encode(file, sub_image)

	fmt.Println("爬取完成")
}
```

## Result

### scrnsht.png

![](https://imgur.com/IbihnjU.png)

### crop.png

![](https://imgur.com/Id6JtNo.png)
