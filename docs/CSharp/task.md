# Task

## 簡單的Task範例

```csharp
private async void buttonGo_Click(object sender, EventArgs e)
{
    //進度條初始化
    progressBar1.Maximum = 10;
    progressBar1.Value = 0;

    //非同步執行
    Console.WriteLine("非同步執行開始");
    await Task.Run(() =>
    {
        while (true)
        {

            //模擬耗時工作
            System.Threading.Thread.Sleep(1000);

            //進度條更新
            this.Invoke(new Action(() => //使用Invoke來更新UI
            {
                progressBar1.PerformStep();
            }));

            //進度條完成
            if (progressBar1.Value == progressBar1.Maximum)
            {
                Console.WriteLine("非同步執行結束");
                break;
            }
        }
    });
}
```

## 同時執行兩個執行續

```csharp
private async void buttonGo_Click(object sender, EventArgs e)
{
    //進度條初始化
    progressBar1.Maximum = 10;
    progressBar1.Value = 0;
    progressBar2.Maximum = 10;
    progressBar2.Value = 0;

    //非同步執行
    Console.WriteLine("非同步執行開始");

    //Task1
    Task t1 = Task.Run(() =>
    {
        Console.WriteLine("Task1開始");
        while (true)
        {

            //模擬耗時工作
            System.Threading.Thread.Sleep(1000);

            //進度條更新
            this.Invoke(new Action(() => //使用Invoke來更新UI
            {
                progressBar1.PerformStep();
            }));

            //進度條完成
            if (progressBar1.Value == progressBar1.Maximum)
            {
                Console.WriteLine("Task1結束");
                break;
            }
        }
    });
    //Task2
    Task t2 = Task.Run(() =>
    {
        Console.WriteLine("Task2開始");
        while (true)
        {

            //模擬耗時工作
            System.Threading.Thread.Sleep(500);

            //進度條更新
            this.Invoke(new Action(() => //使用Invoke來更新UI
            {
                progressBar2.PerformStep();
            }));

            //進度條完成
            if (progressBar2.Value == progressBar2.Maximum)
            {
                Console.WriteLine("Task2結束");
                break;
            }
        }
    });

    await Task.Run(() => 
        Task.WaitAll(t1, t2)
    );

    Console.WriteLine("非同步執行結束");
}
```

## 利用CancellationTokenSource達到執行續取消

```csharp
/// <summary>Task</summary>
Task t;

/// <summary>CancellationTokenSource</summary>
CancellationTokenSource cts;

private async void buttonGo_Click(object sender, EventArgs e)
{
    if (t != null && !t.IsCompleted) //Task已初始化且Task尚未完成
    {
        cts.Cancel();
    }
    else
    {
        //進度條初始化
        progressBar1.Maximum = 10;
        progressBar1.Value = 0;

        //非同步執行
        Console.WriteLine("非同步執行開始");

        cts = new CancellationTokenSource();

        t = Task.Run(() =>
        {
            try
            {
                Console.WriteLine("Task開始");

                while (true)
                {
                    cts.Token.ThrowIfCancellationRequested();

                    //模擬耗時工作
                    System.Threading.Thread.Sleep(500);

                    //進度條更新
                    this.Invoke(new Action(() => //使用Invoke來更新UI
                    {
                        progressBar1.PerformStep();
                    }));

                    //進度條完成
                    if (progressBar1.Value == progressBar1.Maximum)
                    {
                        Console.WriteLine("Task結束");
                        break;
                    }
                }
            }
            catch (OperationCanceledException)
            {
                Console.WriteLine("Task取消");
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        });

        await Task.Run(() => {
            t.Wait();
        });

        Console.WriteLine("非同步執行結束");
    }
}
```
