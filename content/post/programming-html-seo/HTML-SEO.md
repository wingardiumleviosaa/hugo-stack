---
title: "[HTML] 設置 html 標籤達成 SEO"
categories:
  - Programming
tags: ["HTML"]
date: 2020-07-12 22:51:00
slug: "html-seo"
---

## SEO

Search Engine Optimization 搜尋引擎優化，是一種透過了解搜尋引擎的運作規則來調整網站，以及提高目的網站在有關搜尋引擎內排名的方式。

<!--more-->

SEO 優化的是指不須付費的原生流量。

![](https://imgur.com/2EAfGvU.png)

以下提供幾種簡單的方式來達成

### `<meta>` tag

HTML meta 標籤可以用來提供網頁內容的資訊給瀏覽器或是搜尋引擎。

- `<title>頁面標題</title>`：頁面標題是爬蟲第一個檢索到的要素。
- `<meta name="description" content="網頁說明" />`：是在搜尋結果頁中呈現的網頁說明。

![](https://imgur.com/3zYVPix.png)

針對不同社群網站的 meta tag，

- facebook  
  例如下面這四行是寫給 facebook 看的，分享到 facebook 上會呈現指定畫面：

```html
<meta property="og:title" content="網站名稱或標題" />
<meta property="og:url" content="網址" />
<meta property="og:image" content="要顯示的縮圖網址" />
<meta property="og:description" content="網頁描述" />
```

- app link  
  在手機瀏覽網頁時，會自動偵測是否有安裝 app，以下以 iOS 舉例：

```html
<!-- 當使用者未安裝 App，會跳出的建議下載文字 -->
<meta property="al:ios:app_name" content="TripAdvisor" />
<!-- 這可以讓使用者在未安裝 App 時，連結至安裝位置 -->
<meta property="al:ios:app_store_id" content="284876795" />
```

---

### JSON-ld

JavaScript Object Notation for Linked Data，JSON 格式的結構化資料，簡單來說就是在描述這個網頁的型態及內容，Google 會較傾向去讀取網頁中這樣的結構化資料，並顯示在搜尋結果上。

#### JSON-ld 用法

由 @context 定義 JSON-LD 使用的各種類別，@type 宣告應用的屬性，再選擇[schema.org](https://schema.org/docs/full.html)中有提到的種類，例如: WebSite, Organization, Product 等，後續再根據後續屬性，如："name","url" 對應填入描述的格式即可。

```html
<script type="application/ld+json">
  [
    {
      "@context": "http://schema.org",
      "@type": "NewsArticle",
      "thumbnailUrl": "https://uc.udn.com.tw/photo/2020/07/12/91/8174102.jpg",
      "url": "https://udn.com/news/story/6813/4695186",
      "mainEntityOfPage": "https://udn.com/news/story/6813/4695186",
      "headline": "川普終於戴口罩了 首次公開亮相！",
      "articleSection": "全球",
      "datePublished": "2020-07-12T07:19:07+08:00",
      "dateModified": "2020-07-12T21:06:13+08:00",
      "keywords": "口罩,疫情,川普,福特",
      "image": {
        "@type": "ImageObject",
        "contentUrl": "https://uc.udn.com.tw/photo/2020/07/12/91/8174102.jpg",
        "url": "https://pgw.udn.com.tw/gw/photo.php?u=https://uc.udn.com.tw/photo/2020/07/12/91/8174102.jpg&s=Y&x=0&y=0&sw=652&sh=435&exp=3600",
        "name": "在疫情期間一直抗拒戴口罩的川普總統，11日訪問華特里德軍醫院時，難得地戴上口罩；這是他自新冠疫情爆發以來，首度戴著口罩公開亮相。美聯社",
        "height": "1000",
        "width": "800"
      },
      "author": {
        "@type": "Person",
        "name": "世界新聞網／即時報導"
      },
      "publisher": {
        "@type": "Organization",
        "name": "聯合新聞網 | 聯合新聞網",
        "url": "https://udn.com",
        "sameAs": "",
        "logo": {
          "@type": "ImageObject",
          "url": "https://udn.com/static/img/UDN_BABY.png",
          "height": "1000",
          "width": "800"
        }
      }
    }
  ]
</script>
```

呈現在 Google 搜尋結果則會像下面這樣：

![](https://imgur.com/slB8jwM.png)

#### JSON-ld 相關資源

在 [Google 開發者文件](https://developers.google.com/search/docs/data-types/article)中有提供了 20 多種的類型可以在搜尋結果有不一樣的呈現，其中都有範例以及欄位說明，而 JSON-LD 標準的欄位其實多到數不完，詳細可以參考[schema.org](https://schema.org/)。

#### JSON-ld 測試工具

另外 Google 也提供了[結構化資料測試工具](https://search.google.com/structured-data/testing-tool)可以讓開發者快速的檢查是否有欄位缺漏。

---

### robot.txt

給網頁爬蟲看的檔案，可以標記出那些是要爬的哪些不是。如果你有特定頁面（例如未完成或測試用的頁面）會傷害使用者體驗（UX），則可以透過 robot.txt 去阻止該頁面出現在搜尋結果中。以下是標示在檔案內的資訊

- User-agent：填入搜尋引擎爬蟲（\* 號代表全部）
- Disallow：填入你希望搜尋引擎**別**檢索的頁面路徑
- Allow：若你禁止檢索的頁面路徑裡面又有特定路徑你希望搜尋引擎檢索，則填入。

---

### sitemap.xml

網站地圖 sitemap.xml 包含了網站內所有網頁的目錄檔案，提供給搜尋引擎的爬蟲閱讀，讓搜尋引擎可以知道網站內到底有哪些網頁。另外當你在網站中新增了一篇文章，從新增到被搜爬蜘蛛搜爬可能已經過了好幾天。只要更新 sitemap 並重新提交，就可以讓爬蟲知道你的網站有進行更新，進而重新檢索網頁。  
sitemap 會上傳到網站的根目錄，輸入網址 `http://www.example.com/sitemap.xml`，就可以看到。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.example.com.tw</loc>
  </url>
  <url>
    <loc>https://www.example.com.tw/a.html</loc>
  </url>
  <url>
    <loc>https://www.example.com.tw/b.html</loc>
  </url>
</urlset>
```

## Reference

- [Wiki-搜尋引擎最佳化
  ](https://zh.wikipedia.org/wiki/%E6%90%9C%E5%B0%8B%E5%BC%95%E6%93%8E%E6%9C%80%E4%BD%B3%E5%8C%96)
- [structured-data-JSON-LD](https://www.webdesigns.com.tw/structured-data-JSON-LD.asp)
- [使用 JSON-LD 處理 SEO，並讓 Google 針對不同形式網站做獨特的搜尋結果呈現](https://medium.com/@z3388638/%E4%BD%BF%E7%94%A8-json-ld-%E8%99%95%E7%90%86-seo-%E4%B8%A6%E8%AE%93-google-%E9%87%9D%E5%B0%8D%E4%B8%8D%E5%90%8C%E5%BD%A2%E5%BC%8F%E7%B6%B2%E7%AB%99%E5%81%9A%E7%8D%A8%E7%89%B9%E7%9A%84%E6%90%9C%E5%B0%8B%E7%B5%90%E6%9E%9C%E5%91%88%E7%8F%BE-9c74783c017a)
- [meta-robots-and-robots-txt](https://www.yesharris.com/meta-robots-and-robots-txt/)
- 此篇為觀看 Lidemy FE101 的筆記，部分內容取自上課影片
