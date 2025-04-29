# Task 與 Thread

在 C# 中，`Task` 和 `Thread` 都可用於執行並行操作，提升應用程式的效能和回應性。然而，它們在使用方式和提供的功能上有所不同。

## Task

`Task` 是基於 **任務並行庫 (Task Parallel Library, TPL)** 的抽象，更易於使用和管理非同步操作。它通常與 `async` 和 `await` 關鍵字結合使用，實現非阻塞式的等待。

**關鍵特性：**

* **非阻塞式等待：** 使用 `await` 關鍵字可以讓程式在等待 `Task` 完成時不會凍結 UI 執行緒或其他重要的執行緒。
* **async/await 關鍵字：** `async` 標記方法為非同步方法，`await` 用於等待非同步操作完成。
* **更高級的抽象：** 相較於 `Thread`，`Task` 提供了更多內建的功能，如取消、延續、以及更方便的結果處理。

執行一個 `Task` ： 執行後顯示 **Task Done**

```csharp
private async void button_Click(object sender, EventArgs e)
{
    await Task.Run(() =>
    {
        Console.WriteLine("Task started");

        Thread.Sleep(1000);
    });

    Console.WriteLine("Task Done");
}
```

同時執行兩個 `Task` ： `t1` 與 `t2` 同時執行，而不是 `t2` 不會等 `t1` 執行好才執行；皆執行好了後顯示 **Task Done**

```csharp
private async void button_Click(object sender, EventArgs e)
{
    //Task1
    Task t1 = Task.Run(() =>
    {
        Console.WriteLine("Task1 started");

        System.Threading.Thread.Sleep(1000);
    });

    //Task2
    Task t2 = Task.Run(() =>
    {
        Console.WriteLine("Task2 started");

        System.Threading.Thread.Sleep(5000);
    });

    await Task.Run(() => {
        Task.WaitAll(t1, t2);
    });

    Console.WriteLine("Task Done");
}
```

利用 **CancellationTokenSource** 取消 `Task` ： 取消後要執行的方法，可以寫在 `cts.Cancel();` 後面，或是寫在 `catch (TaskCanceledException){}`內

```csharp
/// <summary>Task</summary>
Task t;

/// <summary>CancellationTokenSource</summary>
CancellationTokenSource cts;

private async void button1_Click(object sender, EventArgs e)
{
    if (t != null && !t.IsCompleted) //Task已初始化且Task尚未完成
    {
        cts.Cancel();

    }
    else
    {
        cts = new CancellationTokenSource();

        t = Task.Run(() =>
        {
            try
            {
                for (int i = 0; i < 100; i++)
                {
                    if (cts.Token.IsCancellationRequested)
                    {
                        Console.WriteLine("Task cancelled");
                        break;
                    }

                    Console.WriteLine("Task running: " + i);

                    Thread.Sleep(1000);
                }
            }
            catch (TaskCanceledException)
            {
                Console.WriteLine("Task Cancel");
            }
            finally
            {
                Console.WriteLine("Task finish");
            }
        }, cts.Token);

        await t; // /等待 t 執行完畢
    }
}
```

## Thread

`Thread` 代表一個底層的作業系統執行緒。直接操作 `Thread` 相對較為複雜，且容易產生資源管理和同步問題。在現代 C# 開發中，通常建議使用 `Task` 來進行並行操作。

**關鍵特性：**

* **直接操作作業系統執行緒：** 提供了對執行緒的更底層控制。
* **阻塞式等待：** 使用 `Join()` 方法會導致呼叫執行緒阻塞，直到被等待的執行緒完成。

執行一個 `Thread` ： 執行後顯示 **Task Done** 。 這邊注意到 **button Click Finish** 是立即顯示，而不是等到 `t1` 結束後才顯示，
 
```csharp
private void button_Click(object sender, EventArgs e)
{
    Console.WriteLine("button Click Strat");

    Thread t1 = new Thread(ThreadMethod);
    t1.Start();

    Console.WriteLine("button Click Finish");
}


static void ThreadMethod()
{
    Console.WriteLine("Thread Start");

    Thread.Sleep(3000);

    Console.WriteLine("Thread Done");
}
```

執行兩個 `Thread` ： 注意這邊是建立一個 `t` 來處理 `t1` 與 `t2` 的同時執行與互相等待，如果把 **MainThreadMethod** 內的程式碼拉到 `button_Click` 內執行，會造成UI執行續卡死

```csharp
private void button_Click(object sender, EventArgs e)
{
    Console.WriteLine("button Click Strat");

    Thread t = new Thread(() => {
        MainThreadMethod("Apple");
    } );
    t.Start();

    Console.WriteLine("button Click Finish");
}

static void MainThreadMethod(String param)
{
    Console.WriteLine("Param Is " + param);

    Console.WriteLine("All Thread Start");

    Thread t1 = new Thread(ThreadMethod1);
    Thread t2 = new Thread(ThreadMethod2);
    t1.Start();
    t2.Start();

    Console.WriteLine("WAIT Thread Finish");
    t1.Join();
    t2.Join();

    Console.WriteLine("All Thread Done");
}


static void ThreadMethod1()
{
    Console.WriteLine("Thread 1 Start");

    Thread.Sleep(1000);

    Console.WriteLine("Thread 1 Done");
}

static void ThreadMethod2()
{
    Console.WriteLine("Thread 2 Start");

    Thread.Sleep(3000);

    Console.WriteLine("Thread 2 Done");
}

```

利用 **Abort** 取消 `Thread` ： 取消後要執行的方法，可以寫在 `t.Abort();` 後面，或是寫在 `catch (ThreadAbortException){}`內

```csharp
/// <summary>Thread</summary>
private Thread t;

private void button1_Click(object sender, EventArgs e)
{
    if (t != null && t.IsAlive)
    {
        t.Abort();
    }
    else
    {
        Console.WriteLine("button Click Strat");

        t = new Thread(ThreadMethod);
        t.Start();

        Console.WriteLine("button Click Finish");
    }
}

static void ThreadMethod()
{
    try
    {
        Console.WriteLine("Thread Start");

        Thread.Sleep(5000);

        Console.WriteLine("Thread Done");
    }
    catch (ThreadAbortException)
    {
        Console.WriteLine("Thread Abort");
    }
    finally
    {
        ThreadDoneMethod();
    }
}

static void ThreadDoneMethod()
{
    Console.WriteLine("Thread Finish");
}
```

## Invoke

當在非 UI 執行緒中執行的程式碼需要更新 UI 元素時，會因為跨執行緒存取而導致錯誤。這是因為 UI 元素只能由創建它們的執行緒（通常是主 UI 執行緒）進行修改。 `Control.Invoke()` 方法提供了一種安全地將委派封送回 UI 執行緒執行的方式。

```csharp
this.Invoke(new Action(() => //使用Invoke來更新UI
{
    progressBar1.PerformStep(); //進度條更新
}));
```
