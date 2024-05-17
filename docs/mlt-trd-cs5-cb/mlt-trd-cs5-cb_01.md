# 第一章。线程基础知识

在本章中，我们将介绍在 C#中使用线程的基本任务。您将了解到：

+   在 C#中创建线程

+   暂停线程

+   使线程等待

+   中止线程

+   确定线程状态

+   线程优先级

+   前台和后台线程

+   向线程传递参数

+   使用 C#锁定关键字进行锁定

+   使用监视器构造进行锁定

+   处理异常

# 介绍

在过去的某个时刻，普通计算机只有一个计算单元，无法同时执行多个计算任务。然而，操作系统已经可以同时处理多个程序，实现了多任务的概念。为了防止一个程序永远控制 CPU，导致其他应用程序和操作系统本身挂起，操作系统必须以某种方式将物理计算单元分割成几个虚拟处理器，并为每个执行程序分配一定量的计算能力。此外，操作系统必须始终具有对 CPU 的优先访问权，并且应该能够为不同的程序优先访问 CPU。线程是这一概念的实现。它可以被认为是分配给独立运行的特定程序的虚拟处理器。

### 注意

请记住，线程会消耗大量的操作系统资源。试图在许多线程之间共享一个物理处理器将导致操作系统忙于管理线程而无法运行程序的情况。

因此，虽然可以增强计算机处理器，使其每秒执行更多命令，但处理线程通常是操作系统的任务。在单核 CPU 上尝试并行计算某些任务是没有意义的，因为这比按顺序运行这些计算需要更多时间。然而，当处理器开始拥有更多的计算核心时，旧程序无法利用这一点，因为它们只使用一个处理器核心。

为了有效地利用现代处理器的计算能力，非常重要的是能够以多线程通信和同步的方式组织程序，从而使用多个计算核心。

本章的示例将重点介绍在 C#语言中使用线程执行一些非常基本的操作。我们将涵盖线程的生命周期，包括创建、挂起、使线程等待和中止线程，然后我们将介绍基本的同步技术。

# 在 C#中创建线程

在接下来的示例中，我们将使用 Visual Studio 2012 作为编写 C#多线程程序的主要工具。本示例将向您展示如何创建一个新的 C#程序并在其中使用线程。

### 注意

有免费的 Visual Studio 2012 Express 版本，可以从微软网站下载。我们将需要 Visual Studio 2012 Express for Windows Desktop 来进行大多数示例，以及 Visual Studio 2012 Express for Windows 8 来进行 Windows 8 特定的示例。

## 准备工作

要完成本示例，您将需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter1\Recipe1`中找到。

### 提示

**下载示例代码**

您可以通过您在[`www.packtpub.com`](http://www.packtpub.com)的帐户下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

## 如何做...

要了解如何创建一个新的 C#程序并在其中使用线程，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  确保项目使用.NET Framework 4.0 或更高版本。![如何做...](img/7644OT_01_01.jpg)

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void PrintNumbers()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Console.WriteLine(i);
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Thread t = new Thread(PrintNumbers);
t.Start();
PrintNumbers();
```

1.  运行程序。输出将会是这样的：![如何做...](img/7644OT_01_02.jpg)

## 它是如何工作的...

在步骤 1 和 2 中，我们使用.Net Framework 版本 4.0 创建了一个简单的 C#控制台应用程序。然后在第 3 步中，我们包含了包含程序所需的所有类型的`System.Threading`命名空间。

### 注意

正在执行程序的实例可以称为进程。一个进程由一个或多个线程组成。这意味着当我们运行一个程序时，我们总是有一个执行程序代码的主线程。

在第 4 步中，我们定义了`PrintNumbers`方法，该方法将在主线程和新创建的线程中使用。然后在第 5 步中，我们创建了一个运行`PrintNumbers`的线程。当我们构造一个线程时，`ThreadStart`或`ParameterizedThreadStart`委托的实例被传递给构造函数。当我们只需输入要在不同线程中运行的方法的名称时，C#编译器在幕后创建了这个对象。然后我们启动一个线程，并在主线程上以通常的方式运行`PrintNumbers`。

结果将会有两个范围从 1 到 10 的数字范围随机交叉。这说明`PrintNumbers`方法同时在主线程和另一个线程上运行。

# 暂停一个线程

这个示例将向您展示如何使一个线程在一段时间内等待，而不浪费操作系统资源。

## 准备好了

要完成本示例，您需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter1\Recipe2`中找到。

## 如何做...

要理解如何使一个线程等待而不浪费操作系统资源，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void PrintNumbers()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Console.WriteLine(i);
  }
}
static void PrintNumbersWithDelay()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(2));
    Console.WriteLine(i);
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Thread t = new Thread(PrintNumbersWithDelay);
t.Start();
PrintNumbers();
```

1.  运行程序。

## 它是如何工作的...

当程序运行时，它创建一个线程，该线程将在`PrintNumbersWithDelay`方法中执行代码。在那之后，它立即运行`PrintNumbers`方法。这里的关键特点是在`PrintNumbersWithDelay`方法中添加`Thread.Sleep`方法调用。它会导致执行此代码的线程在打印每个数字之前等待指定的时间（在我们的例子中为两秒）。当一个线程正在睡眠时，它尽可能少地使用 CPU 时间。因此，我们将看到通常稍后运行的`PrintNumbers`方法中的代码将在单独的线程中的`PrintNumbersWithDelay`方法中的代码之前执行。

# 使一个线程等待

这个示例将向您展示程序如何等待另一个线程中的某些计算完成，以便稍后在代码中使用其结果。仅使用`Thread.Sleep`是不够的，因为我们不知道计算需要多长时间。

## 准备好了

要完成本示例，您需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter1\Recipe3`中找到。

## 如何做...

要理解程序如何等待另一个线程中的某些计算完成，以便稍后使用其结果，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void PrintNumbersWithDelay()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(2));
    Console.WriteLine(i);
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Console.WriteLine("Starting...");
Thread t = new Thread(PrintNumbersWithDelay);
t.Start();
t.Join();
Console.WriteLine("Thread completed");
```

1.  运行程序。

## 它是如何工作的...

当程序运行时，它运行一个长时间运行的线程，打印出数字，并在打印每个数字之前等待两秒。但在主程序中，我们调用了`t.Join`方法，这允许我们等待线程`t`完成。当它完成时，主程序继续运行。借助这种技术，可以在两个线程之间同步执行步骤。第一个线程等待另一个完成，然后继续工作。在第一个线程等待时，它处于阻塞状态（就像在之前的示例中调用`Thread.Sleep`时一样）。

# 中止一个线程

在本示例中，我们将描述如何中止另一个线程的执行。

## 准备工作

要完成本示例，您需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter1\Recipe4`中找到。

## 如何做...

要了解如何中止另一个线程的执行，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static void PrintNumbersWithDelay()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(2));
    Console.WriteLine(i);
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Console.WriteLine("Starting program...");
Thread t = new Thread(PrintNumbersWithDelay);
t.Start();
Thread.Sleep(TimeSpan.FromSeconds(6));
t.Abort();
Console.WriteLine("A thread has been aborted");
Thread t = new Thread(PrintNumbers);
t.Start();
PrintNumbers();
```

1.  运行程序。

## 它是如何工作的...

当主程序和一个单独的打印数字的线程运行时，我们等待 6 秒，然后在一个线程上调用`t.Abort`方法。这会向线程注入一个`ThreadAbortException`方法，导致线程终止。这是非常危险的，通常因为这个异常可能在任何时候发生，可能会完全破坏应用程序。此外，并不总是可能使用这种技术终止线程。目标线程可能拒绝通过处理此异常并调用`Thread.ResetAbort`方法来中止。因此，不建议使用`Abort`方法来关闭线程。有不同的方法更受推荐，比如提供一个`CancellationToken`方法来取消线程执行。这种方法将在第三章*使用线程池*中描述。

# 确定线程状态

本示例将描述线程可能具有的可能状态。了解线程是否已启动或是否处于阻塞状态非常有用。请注意，因为线程独立运行，其状态可能随时改变。

## 准备工作

要完成本示例，您需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter1\Recipe5`中找到。

## 如何做...

要了解如何确定线程状态并获取有用的信息，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static void DoNothing()
{
  Thread.Sleep(TimeSpan.FromSeconds(2));
}

static void PrintNumbersWithStatus()
{
  Console.WriteLine("Starting...");
  Console.WriteLine(Thread.CurrentThread
  .ThreadState.ToString());
  for (int i = 1; i < 10; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(2));
    Console.WriteLine(i);
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Console.WriteLine("Starting program...");
Thread t = new Thread(PrintNumbersWithStatus);
Thread t2 = new Thread(DoNothing);
Console.WriteLine(t.ThreadState.ToString());
t2.Start();
t.Start();
for (int i = 1; i < 30; i++)
{
  Console.WriteLine(t.ThreadState.ToString());
}
Thread.Sleep(TimeSpan.FromSeconds(6));
t.Abort();
Console.WriteLine("A thread has been aborted");
Console.WriteLine(t.ThreadState.ToString());
Console.WriteLine(t2.ThreadState.ToString());
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它定义了两个不同的线程；其中一个将被中止，另一个成功运行。线程状态位于`Thread`对象的`ThreadState`属性中，这是一个 C#枚举。一开始，线程处于`ThreadState.Unstarted`状态。然后我们运行它，并假设在 30 次循环迭代的过程中，线程将从`ThreadState.Running`状态变为`ThreadState.WaitSleepJoin`状态。

### 提示

请注意，当前的`Thread`对象始终可以通过`Thread.CurrentThread`静态属性访问。

如果没有发生，只需增加迭代次数。然后我们中止第一个线程，并看到现在它有一个`ThreadState.Aborted`状态。程序也可能打印出`ThreadState.AbortRequested`状态。这很好地说明了同步两个线程的复杂性。请记住，您不应该在程序中使用线程中止。我在这里只是为了展示相应的线程状态。

最后，我们可以看到我们的第二个线程`t2`成功完成，现在有一个`ThreadState.Stopped`状态。还有其他几种状态，但它们部分已被弃用，部分不如我们检查的那些有用。

# 线程优先级

本示例将描述线程优先级的不同可能选项。设置线程优先级确定线程将获得多少 CPU 时间。

## 准备就绪

要完成这个示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter1\Recipe6`中找到。

## 如何做...

要理解线程优先级的工作原理，请执行以下步骤：

1.  开始 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Diagnostics;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void RunThreads()
{
  var sample = new ThreadSample();

  var threadOne = new Thread(sample.CountNumbers);
  threadOne.Name = "ThreadOne";
  var threadTwo = new Thread(sample.CountNumbers);
  threadTwo.Name = "ThreadTwo";

  threadOne.Priority = ThreadPriority.Highest;
  threadTwo.Priority = ThreadPriority.Lowest;
  threadOne.Start();
  threadTwo.Start();

  Thread.Sleep(TimeSpan.FromSeconds(2));
  sample.Stop();
}
class ThreadSample
{
  private bool _isStopped = false;
  public void Stop()
  {
    _isStopped = true;
  }

  public void CountNumbers()
  {
    long counter = 0;

    while (!_isStopped)
    {
      counter++;
    }

    Console.WriteLine("{0} with {1,11} priority " +"has a count = {2,13}", Thread.CurrentThread.Name, Thread.CurrentThread.Priority,counter.ToString("N0"));
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Console.WriteLine("Current thread priority: {0}", Thread.CurrentThread.Priority);
Console.WriteLine("Running on all cores available");
RunThreads();
Thread.Sleep(TimeSpan.FromSeconds(2));
Console.WriteLine("Running on a single core");
Process.GetCurrentProcess().ProcessorAffinity = new IntPtr(1);
RunThreads();
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它定义了两个不同的线程。第一个是`ThreadPriority.Highest`，将具有最高的线程优先级，而第二个是`ThreadPriority.Lowest`，将具有最低的优先级。我们打印出主线程优先级值，然后在所有可用的核心上启动这两个线程。如果我们有多个计算核心，我们应该在两秒内得到一个初始结果。最高优先级线程通常应该计算更多迭代，但两个值应该接近。但是，如果有其他程序运行并加载所有 CPU 核心，情况可能会大不相同。

为了模拟这种情况，我们设置了`ProcessorAffinity`选项，指示操作系统在单个 CPU 核心（编号为一）上运行所有线程。现在结果应该非常不同，计算将花费超过 2 秒。这是因为 CPU 核心将主要运行高优先级线程，给其他线程很少的时间。

请注意，这是操作系统如何处理线程优先级的示例。通常，您不应该编写依赖于此行为的程序。

# 前台和后台线程

本示例将描述前台和后台线程是什么，以及设置此选项如何影响程序行为。

## 准备就绪

要完成这个示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter1\Recipe7`中找到。

## 如何做...

要理解前台和后台线程对程序的影响，请执行以下操作：

1.  开始 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
class ThreadSample
{
  private readonly int _iterations;

  public ThreadSample(int iterations)
  {
    _iterations = iterations;
  }
  public void CountNumbers()
  {
    for (int i = 0; i < _iterations; i++)
    {
      Thread.Sleep(TimeSpan.FromSeconds(0.5));
      Console.WriteLine("{0} prints {1}", Thread.CurrentThread.Name, i);
    }
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var sampleForeground = new ThreadSample(10);
var sampleBackground = new ThreadSample(20);

var threadOne = new Thread(sampleForeground.CountNumbers);
threadOne.Name = "ForegroundThread";
var threadTwo = new Thread(sampleBackground.CountNumbers);
threadTwo.Name = "BackgroundThread";
threadTwo.IsBackground = true;

threadOne.Start();
threadTwo.Start();
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它定义了两个不同的线程。默认情况下，我们显式创建的线程是前台线程。要创建后台线程，我们手动将`threadTwo`对象的`IsBackground`属性设置为`true`。我们以第一个线程将更快完成的方式配置这些线程，然后运行程序。

第一个线程完成后，程序关闭，后台线程终止。这是两者之间的主要区别：进程在完成工作之前等待所有前台线程完成，但如果有后台线程，它们只是关闭。

还要注意的是，如果程序定义了一个前台线程，而这个线程没有完成，主程序将无法正常结束。

# 向线程传递参数

这个示例将描述如何向在另一个线程中运行的代码提供所需的数据。我们将介绍不同的方式来完成这个任务，并审查常见的错误。

## 准备工作

要完成这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter1\Recipe8`中找到。

## 如何做...

要了解如何向线程传递参数，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void Count(object iterations)
{
  CountNumbers((int)iterations);
}

static void CountNumbers(int iterations)
{
  for (int i = 1; i <= iterations; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(0.5));
    Console.WriteLine("{0} prints {1}", Thread.CurrentThread.Name, i);
  }
}
static void PrintNumber(int number)
{
  Console.WriteLine(number);
}

class ThreadSample
{
  private readonly int _iterations;

  public ThreadSample(int iterations)
  {
    _iterations = iterations;
  }
  public void CountNumbers()
  {
    for (int i = 1; i <= _iterations; i++)
    {
      Thread.Sleep(TimeSpan.FromSeconds(0.5));
      Console.WriteLine("{0} prints {1}", Thread.CurrentThread.Name, i);
    }
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
var sample = new ThreadSample(10);

var threadOne = new Thread(sample.CountNumbers);
threadOne.Name = "ThreadOne";
threadOne.Start();
threadOne.Join();
Console.WriteLine("--------------------------");

var threadTwo = new Thread(Count);
threadTwo.Name = "ThreadTwo";
threadTwo.Start(8);
threadTwo.Join();
Console.WriteLine("--------------------------");

var threadThree = new Thread(() => CountNumbers(12));
threadThree.Name = "ThreadThree";
threadThree.Start();
threadThree.Join();
Console.WriteLine("--------------------------");

int i = 10;
var threadFour = new Thread(() => PrintNumber(i));
i = 20;
var threadFive = new Thread(() => PrintNumber(i));
threadFour.Start(); 
threadFive.Start();
```

1.  运行程序。

## 工作原理...

当主程序启动时，首先创建一个`ThreadSample`类的对象，并为其提供一定数量的迭代次数。然后我们使用对象的方法`CountNumbers`启动一个线程。这个方法在另一个线程中运行，但它使用数字 10，这是我们传递给对象构造函数的值。因此，我们只是以同样间接的方式将这个迭代次数传递给另一个线程。

## 还有更多...

另一种传递数据的方式是使用`Thread.Start`方法，接受一个可以传递给另一个线程的对象。为了以这种方式工作，我们在另一个线程中启动的方法必须接受一个类型为 object 的单个参数。通过创建一个`threadTwo`线程来说明这个选项。我们将`8`作为一个对象传递给`Count`方法，在那里它被转换为`integer`类型。

下一个选项涉及使用 lambda 表达式。lambda 表达式定义了一个不属于任何类的方法。我们创建这样一个方法，调用另一个方法所需的参数，并在另一个线程中启动它。当我们启动`threadThree`线程时，它打印出 12 个数字，这些数字正是我们通过 lambda 表达式传递给它的数字。

使用 lambda 表达式涉及另一个名为`闭包`的 C#构造。当我们在 lambda 表达式中使用任何局部变量时，C#会生成一个类，并将这个变量作为这个类的属性。因此，实际上，我们做的事情与`threadOne`线程中的一样，但我们不是自己定义这个类；C#编译器会自动完成这个工作。

这可能会导致几个问题；例如，如果我们从几个 lambda 中使用相同的变量，它们实际上会共享这个变量的值。这可以通过前面的例子来说明；当我们启动`threadFour`和`threadFive`时，它们都会打印出`20`，因为在启动这两个线程之前，变量已经被更改为持有值`20`。

# 使用 C#锁定关键字进行锁定

这个示例将描述如何确保一个线程使用某个资源时，另一个线程不会同时使用它。我们将看到为什么需要这样做，以及线程安全概念是什么。

## 准备工作

要完成这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter1\Recipe9`中找到。

## 如何做...

要了解如何使用 C#锁定关键字，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void TestCounter(CounterBase c)
{
  for (int i = 0; i < 100000; i++)
  {
    c.Increment();
    c.Decrement();
  }
}

class Counter : CounterBase
{
  public int Count { get; private set; }
  public override void Increment()
  {
    Count++;
  }

  public override void Decrement()
  {
    Count--;
  }
}

class CounterWithLock : CounterBase
{
  private readonly object _syncRoot = new Object();

  public int Count { get; private set; }

  public override void Increment()
  {
    lock (_syncRoot)
    {
      Count++;
    }
  }

  public override void Decrement()
  {
    lock (_syncRoot)
    {
      Count--;
    }
  }
}

abstract class CounterBase
{
  public abstract void Increment();
  public abstract void Decrement();
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Console.WriteLine("Incorrect counter");

var c = new Counter();

var t1 = new Thread(() => TestCounter(c));
var t2 = new Thread(() => TestCounter(c));
var t3 = new Thread(() => TestCounter(c));
t1.Start();
t2.Start();
t3.Start();
t1.Join();
t2.Join();
t3.Join();

Console.WriteLine("Total count: {0}",c.Count);
Console.WriteLine("--------------------------");
Console.WriteLine("Correct counter");

var c1 = new CounterWithLock();

t1 = new Thread(() => TestCounter(c1));
t2 = new Thread(() => TestCounter(c1));
t3 = new Thread(() => TestCounter(c1));
t1.Start();
t2.Start();
t3.Start();
t1.Join();
t2.Join();
t3.Join();
Console.WriteLine("Total count: {0}", c1.Count);
```

1.  运行程序。

## 工作原理...

当主程序启动时，首先创建一个`Counter`类的对象。这个类定义了一个简单的计数器，可以进行增加和减少。然后我们启动三个线程，它们共享同一个计数器实例，并在一个循环中执行增加和减少操作。这会导致不确定的结果。如果我们多次运行程序，会打印出几个不同的计数器值。它可能是零，但大多数情况下不会是。

这是因为`Counter`类不是线程安全的。当多个线程同时访问计数器时，第一个线程获取计数器值为`10`并将其增加到 11。然后第二个线程获取值 11 并将其增加到 12。第一个线程获取计数器值 12，但在减少之前，第二个线程也获取了计数器值 12。然后第一个线程将 12 减少到 11 并保存到计数器中，而第二个线程同时也做同样的操作。结果是我们有两次增加和只有一次减少，这显然是不对的。这种情况被称为竞争条件，是多线程环境中错误的一个常见原因。

为了确保这种情况不会发生，我们必须确保当一个线程使用计数器时，所有其他线程必须等待，直到第一个线程完成工作。我们可以使用`lock`关键字来实现这种行为。如果我们`lock`一个对象，所有需要访问这个对象的其他线程将会处于阻塞状态，直到它被解锁。这可能会导致严重的性能问题，稍后在第二章中，*线程同步*，我们将学到更多关于这个的知识。

# 使用 Monitor 构造锁定

这个示例说明了另一个常见的多线程错误，称为死锁。由于死锁会导致程序停止工作，所以这个示例的第一部分是一个新的`Monitor`构造，它允许我们避免死锁。然后，之前描述的`lock`关键字被用来产生死锁。

## 准备就绪

要完成这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter1\Recipe10`中找到。

## 如何做...

要理解多线程错误死锁，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void LockTooMuch(object lock1, object lock2)
{
  lock (lock1)
  {
    Thread.Sleep(1000);
    lock (lock2);
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
object lock1 = new object();
object lock2 = new object();

new Thread(() => LockTooMuch(lock1, lock2)).Start();

lock (lock2)
{
  Thread.Sleep(1000);
  Console.WriteLine("Monitor.TryEnter allows not to get stuck, returning false after a specified timeout is elapsed");
  if (Monitor.TryEnter(lock1, TimeSpan.FromSeconds(5)))
  {
    Console.WriteLine("Acquired a protected resource succesfully");
  }
  else
  {
    Console.WriteLine("Timeout acquiring a resource!");
  }
}
new Thread(() => LockTooMuch(lock1, lock2)).Start();

Console.WriteLine("----------------------------------");
lock (lock2)
{
  Console.WriteLine("This will be a deadlock!");
  Thread.Sleep(1000);
  lock (lock1)
  {
    Console.WriteLine("Acquired a protected resource succesfully");
  }
}
```

1.  运行程序。

## 它是如何工作的...

让我们从`LockTooMuch`方法开始。在这个方法中，我们只是锁定第一个对象，等待一秒，然后锁定第二个对象。然后我们在另一个线程中启动这个方法，并尝试从主线程锁定第二个对象，然后锁定第一个对象。

如果我们像示例的第二部分那样使用`lock`关键字，就会发生死锁。第一个线程持有`lock1`对象的`lock`并等待`lock2`对象释放；主线程持有`lock2`对象的`lock`并等待`lock1`对象释放，而在这种情况下永远不会发生。

实际上，`lock`关键字是对`Monitor`类使用的一种语法糖。如果我们反汇编带有`lock`的代码，我们会看到它转换成以下代码片段：

```cs
bool acquiredLock = false;
try
{
  Monitor.Enter(lockObject, ref acquiredLock);

// Code that accesses resources that are protected by the lock.

}
finally
{
  if (acquiredLock)
  {
    Monitor.Exit(lockObject);
  }
}
```

因此，我们可以直接使用`Monitor`类；它有`TryEnter`方法，接受一个超时参数，并在我们无法获取由`lock`保护的资源之前超时返回`false`。

# 处理异常

这个示例将描述如何正确处理其他线程中的异常。在线程内部始终放置一个`try/catch`块非常重要，因为在线程代码外部无法捕获异常。

## 准备就绪

要完成这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter1\Recipe11`中找到。

## 如何做到…

要理解其他线程中异常的处理，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void BadFaultyThread()
{
  Console.WriteLine("Starting a faulty thread...");
  Thread.Sleep(TimeSpan.FromSeconds(2));
  throw new Exception("Boom!");
}

static void FaultyThread()
{
  try
  {
    Console.WriteLine("Starting a faulty thread...");
    Thread.Sleep(TimeSpan.FromSeconds(1));
    throw new Exception("Boom!");
  }
  catch (Exception ex)
  {
    Console.WriteLine("Exception handled: {0}", ex.Message);
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
var t = new Thread(FaultyThread);
t.Start();
t.Join();

try
{
  t = new Thread(BadFaultyThread);
  t.Start();
}
catch (Exception ex)
{
  Console.WriteLine("We won't get here!");
}
```

1.  运行程序。

## 它是如何工作的…

当主程序启动时，它定义了两个将抛出异常的线程。其中一个线程处理异常，而另一个不处理。你可以看到第二个异常没有被`try/catch`块捕获，而是在启动线程的代码周围。因此，如果直接使用线程，一般规则是不要从线程中抛出异常，而是在线程代码内部使用`try/catch`块。

在较旧版本的.NET Framework（1.0 和 1.1）中，这种行为是不同的，未捕获的异常不会强制应用程序关闭。可以通过添加一个应用程序配置文件（如`app.config`）来使用此策略，其中包含以下代码片段：

```cs
<configuration>
  <runtime>
    <legacyUnhandledExceptionPolicy enabled="1" />
  </runtime>
</configuration>
```
