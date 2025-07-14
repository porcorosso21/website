# C#爬蟲：HtmlAgilityPack 與 AngleSharp 比較

在 C# 的網路爬蟲領域，HtmlAgilityPack (HAP) 和 AngleSharp 是開發者最常使用的兩大 HTML 解析函式庫。兩者都功能強大，但其設計理念、API 使用方式及效能表現上存在顯著差異。

## **核心差異**

兩者最根本的差異在於其 HTML 的解析方式。**HtmlAgilityPack** 以其「寬容」的解析聞名，能夠處理各式不規範、甚至是破碎的 HTML 程式碼，這使其在面對真實世界中充斥著不完美網頁的爬蟲任務時，顯得非常可靠。 HAP 已有超過十五年的發展歷史，經過了時間的考驗，對於需要處理老舊或結構不良網站的開發者來說，是一個相當穩定的選擇。

相對地，**AngleSharp** 則是一款嚴格遵循 W3C 官方規範的現代化函式庫，其目標是模擬現代瀏覽器的行為，提供更精準的 DOM 結構。 這使得 AngleSharp 在處理符合 HTML5 和 CSS3 標準的現代化網頁時，表現得更為出色且可預測。

## **選擇器：XPath 與 CSS Selector**

在選取網頁元素方面，兩個函式庫也採用了不同的主要策略。

* **HtmlAgilityPack** 的核心強項在於其對 **XPath** 的原生支援。 XPath 是一種功能強大的查詢語言，對於熟悉 XML 相關技術的開發者來說，上手會非常快。雖然 HAP 本身不直接支援 CSS 選擇器，但可以透過安裝像是 `Fizzler` 等擴充套件來補足此功能。

* **AngleSharp** 則原生支援 **CSS 選擇器**，這與現代前端開發的習慣更為貼近，語法也更為直觀易懂。 AngleSharp 同樣可以透過擴充套件來支援 XPath，讓開發者可以根據個人偏好或專案需求來選擇。

## **效能表現**

在效能方面，兩者各有千秋，選擇哪一個取決於具體的應用場景。

一般來說，對於單純的 HTML 解析任務，**HtmlAgilityPack** 在速度上可能稍佔優勢，尤其是在處理結構相對簡單或不規範的頁面時。 然而，當頁面包含大量的內嵌 CSS 樣式時，AngleSharp 因為需要同時解析 CSS，可能會花費額外的時間。

**AngleSharp** 的設計則更注重於在符合標準的前提下，提供高效能的解析與 DOM 操作。 在處理複雜且符合現代網頁標準的大型文件時，AngleSharp 的表現相當優異。

## **處理動態內容**

現代網站越來越多地使用 JavaScript 來動態載入內容，這對純 HTML 解析器提出了挑戰。

* **HtmlAgilityPack** 本身無法執行 JavaScript，因此它只能解析到網頁初始載入時的靜態 HTML 內容。 若要爬取由 JavaScript 動態生成的資料，開發者通常需要結合使用 Selenium 或 PuppeteerSharp 等瀏覽器自動化工具，先由這些工具模擬瀏覽器行為以完整呈現網頁，再將最終的 HTML 原始碼交給 HtmlAgilityPack 進行解析。

* **AngleSharp** 同樣不直接執行客戶端腳本，但其架構設計上更易於與 JavaScript 引擎（如 Jint）整合，來處理一些簡單的腳本。 不過，對於複雜的單頁應用程式（SPA），同樣建議搭配 Selenium 或 PuppeteerSharp 使用，以確保能獲取到完整的動態內容。

## **社群與維護**

* **HtmlAgilityPack** 擁有龐大的使用者社群和悠久的歷史，在 NuGet 上的下載量非常可觀，這意味著開發者可以輕易找到大量的教學文件和社群支援。

* **AngleSharp** 作為一個更現代化的函式庫，也擁有活躍的開發社群和良好的文件支援。 雖然總下載量不及 HAP，但其專案活躍度與日俱增，並被認為是處理現代網頁的趨勢。

## **程式碼範例比較**

為了更直觀地展示兩者的使用差異，以下提供一個簡單的範例，目標是從一個虛構的書籍網站上爬取所有書名。

**使用 HtmlAgilityPack (搭配 XPath):**
```csharp
using HtmlAgilityPack;
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class HapScraper
{
    public static async Task Scrape()
    {
        var url = "http://books.toscrape.com/";
        var httpClient = new HttpClient();
        var html = await httpClient.GetStringAsync(url);

        var htmlDocument = new HtmlDocument();
        htmlDocument.LoadHtml(html);

        var bookNodes = htmlDocument.DocumentNode.SelectNodes("//h3/a");

        if (bookNodes != null)
        {
            foreach (var bookNode in bookNodes)
            {
                Console.WriteLine(bookNode.GetAttributeValue("title", ""));
            }
        }
    }
}
```

**使用 AngleSharp (搭配 CSS Selector):**
```csharp
using AngleSharp;
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class AngleSharpScraper
{
    public static async Task Scrape()
    {
        var config = Configuration.Default.WithDefaultLoader();
        var context = BrowsingContext.New(config);
        var url = "http://books.toscrape.com/";
        var document = await context.OpenAsync(url);

        var bookElements = document.QuerySelectorAll("h3 > a");

        foreach (var element in bookElements)
        {
            Console.WriteLine(element.GetAttribute("title"));
        }
    }
}
```

## **如何選擇？**

總結來說，HtmlAgilityPack 和 AngleSharp 都是優秀的 C# 爬蟲工具，選擇哪一個更取決於您的具體需求：

**選擇 HtmlAgilityPack 的時機：**

* 目標網站的 HTML 結構較為老舊或不規範。
* 專案對爬取速度有極高要求，且主要是處理靜態頁面。
* 您的團隊對 XPath 語法更為熟悉。
* 需要一個輕量級且經過長期驗證的穩定解決方案。

**選擇 AngleSharp 的時機：**

* 需要處理遵循 HTML5/CSS3 標準的現代化網站。
* 希望使用更直觀的 CSS 選擇器語法。
* 重視解析的精確度和標準相容性，需要模擬瀏覽器的行為。
* 專案可能會涉及對 CSS 樣式的解析或與 JavaScript 引擎的整合。
