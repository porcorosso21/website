# RDLC直接列印

```csharp
private void PrintRDLC(DataTable datatable)
{
    //LocalReport
    LocalReport lr = new LocalReport();
    String res = "rdlc.Report1.rdlc"; //namespace + report name
    lr.ReportEmbeddedResource = res;

    //DataSources
    List<ReportDataSource> rdsl = new List<ReportDataSource>();
    rdsl.Add(new ReportDataSource("DataSet1", datatable));
    lr.DataSources.Add(rdsl[0]);
    /*  OR
        ReportDataSource rds = new ReportDataSource("DataSet1", datatable);
        lr.DataSources.Add(rds);
    */
    lr.Refresh();

    string deviceInfo = @"<DeviceInfo><OutputFormat>EMF</OutputFormat></DeviceInfo>";
    Warning[] warnings;
    var streams = new List<Stream>();
    lr.Render("Image", deviceInfo, (name, fileNameExtension, encoding, mimeType, willSeek) =>
    {
        var stream = new MemoryStream();
        streams.Add(stream);
        return stream;
    }, out warnings);
    foreach (var stream in streams)
    {
        stream.Position = 0;
    }
    _streams = streams;

    //PrintDocument
    PrintDocument printDoc = new PrintDocument();
    printDoc.PrinterSettings.PrinterName = "Microsoft Print to PDF"; //印表機名稱
    printDoc.DefaultPageSettings.PaperSize = new PaperSize("A4", 827, 1169); //設定紙張大小(單位是百分之一英寸)
    printDoc.DefaultPageSettings.Landscape = true; //設定橫印
    printDoc.PrintPage += new PrintPageEventHandler((s, ev) =>
    {
        Metafile pageImage = new Metafile(_streams[_currentPageIndex]);
        ev.Graphics.DrawImage(pageImage, ev.PageBounds);

        _currentPageIndex++;
        ev.HasMorePages = (_currentPageIndex < _streams.Count);
    });
    _currentPageIndex = 0;
    printDoc.Print();
}
private List<Stream> _streams;
private int _currentPageIndex;
```
