# 產生BarCode及QRCode

## BarCode

```csharp
//使用BarcodeLib
BarcodeLib.Barcode b = new BarcodeLib.Barcode();
b.Alignment = BarcodeLib.AlignmentPositions.CENTER; //ALIGNMENT
b.IncludeLabel = true; //顯示標籤
b.LabelPosition = BarcodeLib.LabelPositions.BOTTOMCENTER; //標籤位置
System.Drawing.Image img = b.Encode(BarcodeLib.TYPE.CODE128, "123456", Color.Black, Color.White, 300, 100);

//給pictureBox使用
pictureBox1.Image = img;

//儲存為PNG檔
b.SaveImage("C:\\barcode.png", BarcodeLib.SaveTypes.PNG);

//儲存為字串，供rldc使用
MemoryStream ms = new MemoryStream();
b.SaveImage(ms, BarcodeLib.SaveTypes.PNG);
String base64str = Convert.ToBase64String(ms.ToArray());
```

## QRCode

```csharp
//使用QRCoder
QRCodeGenerator qrGenerator = new QRCodeGenerator();
QRCodeData qrCodeData = qrGenerator.CreateQrCode("https://www.google.com/", QRCodeGenerator.ECCLevel.Q);//容錯
QRCode qrCode = new QRCode(qrCodeData);
Bitmap qrCodeImage = qrCode.GetGraphic(12, Color.Black, Color.White, true);

//給pictureBox使用
pictureBox1.Image = qrCodeImage;

//儲存為PNG檔
qrCodeImage.Save("C:\\qrcode.png", System.Drawing.Imaging.ImageFormat.Png);

//儲存為字串，供rldc使用
MemoryStream ms = new MemoryStream();
qrCodeImage.Save(ms, System.Drawing.Imaging.ImageFormat.Png);
byte[] imageBytes = ms.ToArray();
string base64String = Convert.ToBase64String(imageBytes);
```
