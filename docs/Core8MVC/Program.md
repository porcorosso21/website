# Program.cs 設定範例及說明

下面是 `Program.cs` 的設定說明及範例

```csharp
using Microsoft.EntityFrameworkCore;
using webapp1.Models; //應用程式Model的命名空間

var builder = WebApplication.CreateBuilder(args);

// 註冊 MVC 控制器與檢視相關的服務。這使得應用程式能夠處理 HTTP 請求並呈現使用者介面。
builder.Services.AddControllersWithViews();

// 非開發環境下的設定
if (!builder.Environment.IsDevelopment())
{
    //設定 Kestrel 伺服器的監聽端口 (開發環境伺服器的監聽端口通常由 launchSettings.json 配置)
    builder.WebHost.ConfigureKestrel(options =>
    {
        // HTTP 端口設定
        options.ListenLocalhost(5000); // 監聽localhost的5000端口，使用HTTP協議
        //options.ListenLocalhost(5001, ListenOptions => ListenOptions.UseHttps()); // 監聽localhost的5001端口，使用 HTTPS協議
        options.ListenAnyIP(8080); // 監聽所有IP的8080端口，使用 HTTP協議
        //options.ListenAnyIP(8081, ListenOptions => ListenOptions.UseHttps()); // 監聽所有IP的8081端口，使用 HTTPS協議
    });
}

// 設定資料庫連線字串 (使用 MySQL)
// 將 DBContext 註冊為服務，並配置使用 MySQL 資料庫。
builder.Services.AddDbContext<DBContext>(
    options => options.UseMySql(builder.Configuration.GetConnectionString("DefaultConnection"),
    new MySqlServerVersion(new Version(10, 11, 9)) //版本
));
// 在appsettings.json中設置對應的DefaultConnection
// {
//   "ConnectionStrings": {
//         "DefaultConnection": "Server=localhost;Database=db;Uid=user;Pwd=password;"
//   }
// }

var app = builder.Build();

//  非開發環境下的設定
if (!app.Environment.IsDevelopment())
{
    // 在非開發環境下啟用異常處理，將錯誤導向至指定的路徑
    app.UseExceptionHandler("/Home/Error");
    // 啟用 HTTP 嚴格傳輸安全 (HTTP Strict Transport Security, HSTS) 。
    // HSTS 告知瀏覽器只能透過 HTTPS 存取網站，以提升安全性。
    // 預設值為 30 天，建議在生產環境中根據需求調整。
    app.UseHsts();
}

// 是否使用 HTTPS 重定向。
// 如果啟用，所有 HTTP 請求都會被重新導向到 HTTPS。
// 在發布環境中，若未正確設定 SSL 憑證，啟用此功能可能會導致連線問題。
//app.UseHttpsRedirection();

// 啟用靜態檔案，允許應用程式提供 wwwroot 目錄下的靜態檔案 (例如：CSS、JavaScript、圖片等)。
app.UseStaticFiles();

// 啟用路由，將傳入的 HTTP 請求匹配到應用程式中的端點 (通常是控制器動作)。
app.UseRouting();

// 啟用身份驗證，驗證使用者的身份。
// 必須在 UseRouting() 之後和 UseAuthorization() 之前調用。
app.UseAuthentication();

// 啟用授權驗證，判斷已驗證的使用者是否有權限存取請求的資源。
// 必須在 UseRouting() 之後調用。
app.UseAuthorization();

// 設定路由規則及預設值。
// 定義 URL 模式如何對應到控制器和動作。
// "{controller=Home}/{action=Index}/{id?}" 這個模式表示：
// - 當沒有指定控制器時，預設使用 "Home" 控制器。
// - 當沒有指定動作時，預設執行 "Index" 動作。
// - "{id?}" 表示一個可選的路由參數，名稱為 "id"。
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// 啟動應用程式，開始監聽 HTTP 請求。
app.Run();

```
