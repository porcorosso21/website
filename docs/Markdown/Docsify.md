# docsify

docsify 是一個輕量級的 JavaScript 工具，能快速將 Markdown 文件轉換為網站。它與傳統靜態網站產生器不同，docsify 不預先生成 HTML，而是在瀏覽器端即時解析 Markdown，使更新內容更迅速。其特色在於輕巧易用、無需複雜設定，適合快速建立文件網站、個人筆記或產品說明。透過簡單的 HTML 檔案和 Markdown 文件，即可快速部署，非常適合用於 GitHub Pages 等靜態網站服務。

詳細設定文件請見 <https://docsify.js.org>


## 安裝docsify-cli工具

```bash
npm i docsify-cli -g
```

## 初始化

將根目錄建置在`doc`資料夾中


```bash
docsify init ./docs
```

## 開始建立文檔

初始化成功後，可在`./docs`目錄看到幾個預設的文件

* `index.html` 首頁網頁
* `README.md` 作為首頁的文件
* `.nojekyll` 用阻止GitHub Pages忽略掉下底線開頭的文件

可以直接編輯`README.md`開始建立首頁內容

## 預覽

1. 使用docsify本地伺服器預覽  
  通過執行`docsify serve`啟動一個本地伺服器。預設網址為http://localhost:3000。

  ```bash
  docsify serve docs
  ```

1. 使用Live Server預覽  
  若使用VS Code編輯Markdown文件，可安裝`Live Server`外掛，在`index.html`中按右鍵選擇 ***Open with Live Service*** 來自動開啟預覽網頁。