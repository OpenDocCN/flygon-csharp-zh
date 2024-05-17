# 第九章：处理文件、流和序列化

本章是关于读写文件和流、文本编码和序列化的。

我们将涵盖以下主题：

+   管理文件系统

+   使用流进行读写

+   编码和解码文本

+   序列化对象图

+   控制 JSON 处理

# 管理文件系统

您的应用程序通常需要在不同的环境中对文件和目录执行输入和输出操作。`System`和`System.IO`命名空间包含为此目的的类。

## 处理跨平台环境和文件系统

让我们探讨如何处理跨平台环境，例如 Windows 与 Linux 或 macOS 之间的差异。Windows、macOS 和 Linux 的路径不同，因此我们将从探索.NET 如何处理这一点开始：

1.  使用您喜欢的代码编辑器创建一个名为`Chapter09`的新解决方案/工作区。

1.  添加一个控制台应用程序项目，如下表所定义：

    1.  项目模板：**控制台应用程序**/`console`

    1.  工作区/解决方案文件和文件夹：`Chapter09`

    1.  项目文件和文件夹：`WorkingWithFileSystems`

1.  在`Program.cs`中，添加语句以静态导入`System.Console`、`System.IO.Directory`、`System.Environment`和`System.IO.Path`类型，如下所示：

    ```cs
    using static System.Console; 
    using static System.IO.Directory; 
    using static System.IO.Path; 
    using static System.Environment; 
    ```

1.  在`Program.cs`中，创建一个静态`OutputFileSystemInfo`方法，并在其中编写语句以执行以下操作：

    +   输出路径和目录分隔符字符。

    +   输出当前目录的路径。

    +   输出一些特殊路径，用于系统文件、临时文件和文档。

    ```cs
    static void OutputFileSystemInfo()
    {
      WriteLine("{0,-33} {1}", arg0: "Path.PathSeparator",
        arg1: PathSeparator);
      WriteLine("{0,-33} {1}", arg0: "Path.DirectorySeparatorChar",
        arg1: DirectorySeparatorChar);
      WriteLine("{0,-33} {1}", arg0: "Directory.GetCurrentDirectory()",
        arg1: GetCurrentDirectory());
      WriteLine("{0,-33} {1}", arg0: "Environment.CurrentDirectory", 
        arg1: CurrentDirectory);
      WriteLine("{0,-33} {1}", arg0: "Environment.SystemDirectory", 
        arg1: SystemDirectory);
      WriteLine("{0,-33} {1}", arg0: "Path.GetTempPath()", 
        arg1: GetTempPath());
      WriteLine("GetFolderPath(SpecialFolder");
      WriteLine("{0,-33} {1}", arg0: " .System)", 
        arg1: GetFolderPath(SpecialFolder.System));
      WriteLine("{0,-33} {1}", arg0: " .ApplicationData)", 
        arg1: GetFolderPath(SpecialFolder.ApplicationData));
      WriteLine("{0,-33} {1}", arg0: " .MyDocuments)", 
        arg1: GetFolderPath(SpecialFolder.MyDocuments));
      WriteLine("{0,-33} {1}", arg0: " .Personal)", 
        arg1: GetFolderPath(SpecialFolder.Personal));
    } 
    ```

    `Environment`类型有许多其他有用的成员，我们在此代码中未使用，包括`GetEnvironmentVariables`方法以及`OSVersion`和`ProcessorCount`属性。

1.  在`Program.cs`中，在函数上方调用`OutputFileSystemInfo`方法，如下所示：

    ```cs
    OutputFileSystemInfo(); 
    ```

1.  运行代码并查看结果，如图*9.1*所示：![文本描述自动生成](img/B17442_09_01.png)

    图 9.1：运行应用程序以在 Windows 上显示文件系统信息

当使用 Visual Studio Code 中的`dotnet run`运行控制台应用程序时，`CurrentDirectory`将是项目文件夹，而不是`bin`内的文件夹。

**最佳实践**：Windows 使用反斜杠`\`作为目录分隔符。macOS 和 Linux 使用正斜杠`/`作为目录分隔符。在组合路径时，不要假设代码中使用的是哪种字符。

## 管理驱动器

要管理驱动器，请使用`DriveInfo`类型，该类型有一个静态方法，返回有关连接到计算机的所有驱动器的信息。每个驱动器都有一个驱动器类型。

让我们探索驱动器：

1.  创建一个`WorkWithDrives`方法，并编写语句以获取所有驱动器并输出其名称、类型、大小、可用自由空间和格式，但仅当驱动器就绪时，如下所示：

    ```cs
    static void WorkWithDrives()
    {
      WriteLine("{0,-30} | {1,-10} | {2,-7} | {3,18} | {4,18}",
        "NAME", "TYPE", "FORMAT", "SIZE (BYTES)", "FREE SPACE");
      foreach (DriveInfo drive in DriveInfo.GetDrives())
      {
        if (drive.IsReady)
        {
          WriteLine(
            "{0,-30} | {1,-10} | {2,-7} | {3,18:N0} | {4,18:N0}",
            drive.Name, drive.DriveType, drive.DriveFormat,
            drive.TotalSize, drive.AvailableFreeSpace);
        }
        else
        {
          WriteLine("{0,-30} | {1,-10}", drive.Name, drive.DriveType);
        }
      }
    } 
    ```

    **最佳实践**：在读取`TotalSize`等属性之前，检查驱动器是否就绪，否则对于可移动驱动器，您将看到抛出的异常。

1.  在`Program.cs`中，注释掉之前的方法调用，并添加对`WorkWithDrives`的调用，如下面的代码中突出显示的那样：

    ```cs
    **// OutputFileSystemInfo();**
    **WorkWithDrives();** 
    ```

1.  运行代码并查看结果，如*图 9.2*所示：![](img/B17442_09_02.png)

    图 9.2：在 Windows 上显示驱动器信息

## 管理目录

要管理目录，请使用`Directory`、`Path`和`Environment`静态类。这些类型包含许多用于处理文件系统的成员。

在构建自定义路径时，必须小心编写代码，使其不依赖于平台，例如，不假设使用哪种目录分隔符字符。

1.  创建一个`WorkWithDirectories`方法，并编写语句以执行以下操作：

    +   通过为用户的主目录下创建一个字符串数组来定义自定义路径，然后使用`Path`类型的`Combine`方法正确组合它们。

    +   使用`Directory`类的`Exists`方法检查自定义目录路径是否存在。

    +   使用`Directory`类的`CreateDirectory`和`Delete`方法创建然后删除目录，包括其中的文件和子目录：

    ```cs
    static void WorkWithDirectories()
    {
      // define a directory path for a new folder
      // starting in the user's folder
      string newFolder = Combine(
        GetFolderPath(SpecialFolder.Personal),
        "Code", "Chapter09", "NewFolder");
      WriteLine($"Working with: {newFolder}");
      // check if it exists
      WriteLine($"Does it exist? {Exists(newFolder)}");
      // create directory 
      WriteLine("Creating it...");
      CreateDirectory(newFolder);
      WriteLine($"Does it exist? {Exists(newFolder)}");
      Write("Confirm the directory exists, and then press ENTER: ");
      ReadLine();
      // delete directory 
      WriteLine("Deleting it...");
      Delete(newFolder, recursive: true);
      WriteLine($"Does it exist? {Exists(newFolder)}");
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法调用，并添加对`WorkWithDirectories`的调用。

1.  运行代码并查看结果，并使用您喜欢的文件管理工具确认目录已创建，然后按 Enter 键删除它，如下面的输出所示：

    ```cs
    Working with: /Users/markjprice/Code/Chapter09/NewFolder Does it exist? False
    Creating it...
    Does it exist? True
    Confirm the directory exists, and then press ENTER:
    Deleting it...
    Does it exist? False 
    ```

## 管理文件

在处理文件时，可以像我们为目录类型所做的那样静态导入文件类型，但对于下一个示例，我们将不会这样做，因为它与目录类型有一些相同的方法，并且它们会发生冲突。文件类型的名称足够短，在这种情况下不会造成影响。步骤如下：

1.  创建一个`WorkWithFiles`方法，并编写语句以执行以下操作：

    1.  检查文件是否存在。

    1.  创建文本文件。

    1.  向文件写入一行文本。

    1.  关闭文件以释放系统资源和文件锁（这通常在`try-finally`语句块内部完成，以确保即使写入文件时发生异常，文件也会关闭）。

    1.  将文件复制到备份。

    1.  删除原始文件。

    1.  读取备份文件的内容，然后关闭它：

    ```cs
    static void WorkWithFiles()
    {
      // define a directory path to output files
      // starting in the user's folder
      string dir = Combine(
        GetFolderPath(SpecialFolder.Personal), 
        "Code", "Chapter09", "OutputFiles");
      CreateDirectory(dir);
      // define file paths
      string textFile = Combine(dir, "Dummy.txt");
      string backupFile = Combine(dir, "Dummy.bak");
      WriteLine($"Working with: {textFile}");
      // check if a file exists
      WriteLine($"Does it exist? {File.Exists(textFile)}");
      // create a new text file and write a line to it
      StreamWriter textWriter = File.CreateText(textFile);
      textWriter.WriteLine("Hello, C#!");
      textWriter.Close(); // close file and release resources
      WriteLine($"Does it exist? {File.Exists(textFile)}");
      // copy the file, and overwrite if it already exists
      File.Copy(sourceFileName: textFile,
        destFileName: backupFile, overwrite: true);
      WriteLine(
        $"Does {backupFile} exist? {File.Exists(backupFile)}");
      Write("Confirm the files exist, and then press ENTER: ");
      ReadLine();
      // delete file
      File.Delete(textFile);
      WriteLine($"Does it exist? {File.Exists(textFile)}");
      // read from the text file backup
      WriteLine($"Reading contents of {backupFile}:");
      StreamReader textReader = File.OpenText(backupFile); 
      WriteLine(textReader.ReadToEnd());
      textReader.Close();
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法调用，并添加对`WorkWithFiles`的调用。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Working with: /Users/markjprice/Code/Chapter09/OutputFiles/Dummy.txt 
    Does it exist? False
    Does it exist? True
    Does /Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak exist? True 
    Confirm the files exist, and then press ENTER:
    Does it exist? False
    Reading contents of /Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak:
    Hello, C#! 
    ```

## 管理路径

有时，您需要处理路径的一部分；例如，您可能只想提取文件夹名称、文件名或扩展名。有时，您需要生成临时文件夹和文件名。您可以使用`Path`类的静态方法来完成此操作：

1.  在`WorkWithFiles`方法的末尾添加以下语句：

    ```cs
    // Managing paths
    WriteLine($"Folder Name: {GetDirectoryName(textFile)}"); 
    WriteLine($"File Name: {GetFileName(textFile)}"); 
    WriteLine("File Name without Extension: {0}",
      GetFileNameWithoutExtension(textFile)); 
    WriteLine($"File Extension: {GetExtension(textFile)}"); 
    WriteLine($"Random File Name: {GetRandomFileName()}"); 
    WriteLine($"Temporary File Name: {GetTempFileName()}"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Folder Name: /Users/markjprice/Code/Chapter09/OutputFiles 
    File Name: Dummy.txt
    File Name without Extension: Dummy 
    File Extension: .txt
    Random File Name: u45w1zki.co3 
    Temporary File Name:
    /var/folders/tz/xx0y_wld5sx0nv0fjtq4tnpc0000gn/T/tmpyqrepP.tmp 
    ```

    `GetTempFileName`创建一个零字节文件并返回其名称，供你使用。`GetRandomFileName`仅返回一个文件名；它不创建文件。

## 获取文件信息

要获取有关文件或目录的更多信息，例如其大小或上次访问时间，你可以创建`FileInfo`或`DirectoryInfo`类的实例。

`FileInfo`和`DirectoryInfo`都继承自`FileSystemInfo`，因此它们都具有`LastAccessTime`和`Delete`等成员，以及它们自己的特定成员，如下表所示：

| 类 | 成员 |
| --- | --- |
| `FileSystemInfo` | 字段：`FullPath`、`OriginalPath`属性：`Attributes`、`CreationTime`、`CreationTimeUtc`、`Exists`、`Extension`、`FullName`、`LastAccessTime`、`LastAccessTimeUtc`、`LastWriteTime`、`LastWriteTimeUtc`、`Name`方法：`Delete`、`GetObjectData`、`Refresh` |
| `DirectoryInfo` | 属性：`Parent`、`Root`方法：`Create`、`CreateSubdirectory`、`EnumerateDirectories`、`EnumerateFiles`、`EnumerateFileSystemInfos`、`GetAccessControl`、`GetDirectories`、`GetFiles`、`GetFileSystemInfos`、`MoveTo`、`SetAccessControl` |
| `FileInfo` | 属性：`Directory`、`DirectoryName`、`IsReadOnly`、`Length`方法：`AppendText`、`CopyTo`、`Create`、`CreateText`、`Decrypt`、`Encrypt`、`GetAccessControl`、`MoveTo`、`Open`、`OpenRead`、`OpenText`、`OpenWrite`、`Replace`、`SetAccessControl` |

让我们编写一些代码，使用`FileInfo`实例高效地对文件执行多项操作：

1.  在`WorkWithFiles`方法末尾添加语句，为备份文件创建一个`FileInfo`实例，并将有关它的信息写入控制台，如下面的代码所示：

    ```cs
    FileInfo info = new(backupFile); 
    WriteLine($"{backupFile}:"); 
    WriteLine($"Contains {info.Length} bytes");
    WriteLine($"Last accessed {info.LastAccessTime}"); 
    WriteLine($"Has readonly set to {info.IsReadOnly}"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    /Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak: 
    Contains 11 bytes
    Last accessed 26/10/2021 09:08:26 
    Has readonly set to False 
    ```

字节数可能因操作系统而异，因为操作系统可以使用不同的行结束符。

## 控制你处理文件的方式

处理文件时，你经常需要控制它们的打开方式。`File.Open`方法有重载，可以使用`enum`值指定额外的选项。

`enum`类型如下：

+   `FileMode`：这控制你想要对文件执行的操作，例如`CreateNew`、`OpenOrCreate`或`Truncate`。

+   `FileAccess`：这控制你需要什么级别的访问权限，例如`ReadWrite`。

+   `FileShare`：这控制文件上的锁定，以允许其他进程指定级别的访问权限，例如`Read`。

你可能想要打开一个文件并从中读取，并允许其他进程也读取它，如下面的代码所示：

```cs
FileStream file = File.Open(pathToFile,
  FileMode.Open, FileAccess.Read, FileShare.Read); 
```

文件属性也有一个`enum`，如下所示：

+   `FileAttributes`：这是为了检查`FileSystemInfo`派生类型的`Attributes`属性，例如`Archive`和`Encrypted`。

你可以检查文件或目录的属性，如下面的代码所示：

```cs
FileInfo info = new(backupFile); 
WriteLine("Is the backup file compressed? {0}",
  info.Attributes.HasFlag(FileAttributes.Compressed)); 
```

# 使用流进行读写

一个**流**是一系列字节，可以从中读取和写入。尽管文件可以像数组一样处理，通过知道文件中字节的位置提供随机访问，但将文件作为流处理，其中字节可以按顺序访问，可能会有用。

流还可用于处理终端输入输出和网络资源，如套接字和端口，这些资源不提供随机访问，也不能查找（即移动）到某个位置。您可以编写代码来处理一些任意字节，而无需知道或关心它来自哪里。您的代码只是读取或写入流，而另一段代码处理字节实际存储的位置。

## 理解抽象流和具体流

存在一个名为`Stream`的`抽象`类，它代表任何类型的流。记住，`抽象`类不能使用`new`实例化；它们只能被继承。

有许多具体类继承自这个基类，包括`FileStream`、`MemoryStream`、`BufferedStream`、`GZipStream`和`SslStream`，因此它们都以相同的方式工作。所有流都实现`IDisposable`，因此它们有一个`Dispose`方法来释放非托管资源。

以下表格描述了`Stream`类的一些常见成员：

| 成员 | 描述 |
| --- | --- |
| `CanRead`, `CanWrite` | 这些属性确定是否可以从流中读取和写入。 |
| `Length`, `Position` | 这些属性确定流中的总字节数和当前位置。对于某些类型的流，这些属性可能会引发异常。 |
| `Dispose` | 此方法关闭流并释放其资源。 |
| `Flush` | 如果流有缓冲区，则此方法将缓冲区中的字节写入流，并清除缓冲区。 |
| `CanSeek` | 此属性确定是否可以使用`Seek`方法。 |
| `Seek` | 此方法将其参数指定的新位置移动当前位置。 |
| `Read`, `ReadAsync` | 这些方法从流中读取指定数量的字节到字节数组中，并推进位置。 |
| `ReadByte` | 此方法从流中读取下一个字节并推进位置。 |
| `Write`, `WriteAsync` | 这些方法将字节数组的内容写入流中。 |
| `WriteByte` | 此方法将一个字节写入流中。 |

### 理解存储流

以下表格描述了一些代表字节存储位置的存储流：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.IO` | `FileStream` | 文件系统中存储的字节。 |
| `System.IO` | `MemoryStream` | 当前进程内存中存储的字节。 |
| `System.Net.Sockets` | `NetworkStream` | 网络位置存储的字节。 |

`FileStream` 在 .NET 6 中被重写，以在 Windows 上具有更高的性能和可靠性。

### 理解功能流

某些功能流无法独立存在，只能“附加到”其他流以添加功能，如下表所述：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.Security.Cryptography` | `CryptoStream` | 此流用于加密和解密。 |
| `System.IO.Compression` | `GZipStream`, `DeflateStream` | 这些类用于压缩和解压缩流。 |
| `System.Net.Security` | `AuthenticatedStream` | 此流用于跨流发送凭据。 |

### 理解流辅助类

尽管有时您需要以低级别处理流，但大多数情况下，您可以将辅助类插入链中以简化操作。所有流辅助类型均实现 `IDisposable`，因此它们具有 `Dispose` 方法以释放非托管资源。

处理常见场景的一些辅助类如下表所述：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.IO` | `StreamReader` | 此读取器以纯文本形式从底层流读取数据。 |
| `System.IO` | `StreamWriter` | 此写入器以纯文本形式向底层流写入数据。 |
| `System.IO` | `BinaryReader` | 此读取器以 .NET 类型从流中读取数据。例如，`ReadDecimal` 方法从底层流读取接下来的 16 字节作为 `decimal` 值，而 `ReadInt32` 方法读取接下来的 4 字节作为 `int` 值。 |
| `System.IO` | `BinaryWriter` | 此写入器以 .NET 类型向流写入数据。例如，带有 `decimal` 参数的 `Write` 方法向底层流写入 16 字节，而带有 `int` 参数的 `Write` 方法写入 4 字节。 |
| `System.Xml` | `XmlReader` | 此读取器使用 XML 格式从底层流读取数据。 |
| `System.Xml` | `XmlWriter` | 此写入器使用 XML 格式向底层流写入数据。 |

## 写入文本流

让我们编写一些代码将文本写入流：

1.  使用您偏好的代码编辑器，在 `Chapter09` 解决方案/工作区中添加一个名为 `WorkingWithStreams` 的新控制台应用：

    1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选定项。

    1.  在 Visual Studio Code 中，选择 `WorkingWithStreams` 作为活动 OmniSharp 项目。

1.  在 `WorkingWithStreams` 项目中，在 `Program.cs` 中，导入 `System.Xml` 命名空间并静态导入 `System.Console`、`System.Environment` 和 `System.IO.Path` 类型。

1.  在 `Program.cs` 底部，定义一个名为 `Viper` 的静态类，其中包含一个名为 `Callsigns` 的静态 `string` 数组，如下所示：

    ```cs
    static class Viper
    {
      // define an array of Viper pilot call signs
      public static string[] Callsigns = new[]
      {
        "Husker", "Starbuck", "Apollo", "Boomer",
        "Bulldog", "Athena", "Helo", "Racetrack"
      };
    } 
    ```

1.  在 `Viper` 类上方，定义一个名为 `WorkWithText` 的方法，该方法枚举 Viper 呼号，将每个呼号写入单个文本文件中的一行，如下所示：

    ```cs
    static void WorkWithText()
    {
      // define a file to write to
      string textFile = Combine(CurrentDirectory, "streams.txt");
      // create a text file and return a helper writer
      StreamWriter text = File.CreateText(textFile);
      // enumerate the strings, writing each one
      // to the stream on a separate line
      foreach (string item in Viper.Callsigns)
      {
        text.WriteLine(item);
      }
      text.Close(); // release resources
      // output the contents of the file
      WriteLine("{0} contains {1:N0} bytes.",
        arg0: textFile,
        arg1: new FileInfo(textFile).Length);
      WriteLine(File.ReadAllText(textFile));
    } 
    ```

1.  在命名空间导入下方，调用 `WorkWithText` 方法。

1.  运行代码并查看结果，如下所示：

    ```cs
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.txt contains
    60 bytes. 
    Husker 
    Starbuck 
    Apollo 
    Boomer 
    Bulldog 
    Athena 
    Helo 
    Racetrack 
    ```

1.  打开创建的文件并检查其是否包含呼号列表。

## 写入 XML 流

编写 XML 元素有两种方式，如下所示：

+   `WriteStartElement`和`WriteEndElement`：当元素可能有子元素时使用这一对方法。

+   `WriteElementString`：当元素没有子元素时使用此方法。

现在，让我们尝试将 Viper 飞行员呼号数组`string`值存储在 XML 文件中：

1.  创建一个`WorkWithXml`方法，该方法枚举呼号，并将每个呼号作为单个 XML 文件中的元素写入，如下面的代码所示：

    ```cs
    static void WorkWithXml()
    {
      // define a file to write to
      string xmlFile = Combine(CurrentDirectory, "streams.xml");
      // create a file stream
      FileStream xmlFileStream = File.Create(xmlFile);
      // wrap the file stream in an XML writer helper
      // and automatically indent nested elements
      XmlWriter xml = XmlWriter.Create(xmlFileStream,
        new XmlWriterSettings { Indent = true });
      // write the XML declaration
      xml.WriteStartDocument();
      // write a root element
      xml.WriteStartElement("callsigns");
      // enumerate the strings writing each one to the stream
      foreach (string item in Viper.Callsigns)
      {
        xml.WriteElementString("callsign", item);
      }
      // write the close root element
      xml.WriteEndElement();
      // close helper and stream
      xml.Close();
      xmlFileStream.Close();
      // output all the contents of the file
      WriteLine("{0} contains {1:N0} bytes.",
        arg0: xmlFile,
        arg1: new FileInfo(xmlFile).Length);
      WriteLine(File.ReadAllText(xmlFile));
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法调用，并添加对`WorkWithXml`方法的调用。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.xml contains
    310 bytes.
    <?xml version="1.0" encoding="utf-8"?>
    <callsigns>
      <callsign>Husker</callsign>
      <callsign>Starbuck</callsign>
      <callsign>Apollo</callsign>
      <callsign>Boomer</callsign>
      <callsign>Bulldog</callsign>
      <callsign>Athena</callsign>
      <callsign>Helo</callsign>
      <callsign>Racetrack</callsign>
    </callsigns> 
    ```

## 释放文件资源

当你打开一个文件进行读取或写入时，你正在使用.NET 之外的资源。这些被称为**非托管资源**，并且在完成与它们的工作后必须被释放。为了确定性地控制何时释放它们，我们可以在`finally`块中调用`Dispose`方法。

让我们改进之前处理 XML 的代码，以正确释放其非托管资源：

1.  修改`WorkWithXml`方法，如下面的代码中突出显示的那样：

    ```cs
    static void WorkWithXml()
    {
     **FileStream? xmlFileStream =** **null****;** 
     **XmlWriter? xml =** **null****;**
    **try**
     **{**
        // define a file to write to
        string xmlFile = Combine(CurrentDirectory, "streams.xml");
        // create a file stream
     **xmlFileStream = File.Create(xmlFile);**
        // wrap the file stream in an XML writer helper
        // and automatically indent nested elements
     **xml = XmlWriter.Create(xmlFileStream,**
    **new** **XmlWriterSettings { Indent =** **true** **});**
        // write the XML declaration
        xml.WriteStartDocument();
        // write a root element
        xml.WriteStartElement("callsigns");
        // enumerate the strings writing each one to the stream
        foreach (string item in Viper.Callsigns)
        {
          xml.WriteElementString("callsign", item);
        }
        // write the close root element
        xml.WriteEndElement();
        // close helper and stream
        xml.Close();
        xmlFileStream.Close();
        // output all the contents of the file
        WriteLine($"{0} contains {1:N0} bytes.",
          arg0: xmlFile,
          arg1: new FileInfo(xmlFile).Length);
        WriteLine(File.ReadAllText(xmlFile));
     **}**
     **catch (Exception ex)**
     **{**
    **// if the path doesn't exist the exception will be caught**
     **WriteLine(****$"****{ex.GetType()}** **says** **{ex.Message}****"****);**
     **}**
    **finally**
     **{**
    **if** **(xml !=** **null****)**
     **{** 
     **xml.Dispose();**
     **WriteLine(****"The XML writer's unmanaged resources have been disposed."****);**
    **if** **(xmlFileStream !=** **null****)**
     **{**
     **xmlFileStream.Dispose();**
     **WriteLine(****"The file stream's unmanaged resources have been disposed."****);**
     **}**
     **}**
     **}**
    } 
    ```

    你也可以回去修改你之前创建的其他方法，但我会将其留作可选练习。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    The XML writer's unmanaged resources have been disposed. 
    The file stream's unmanaged resources have been disposed. 
    ```

**最佳实践**：在调用`Dispose`方法之前，检查对象是否不为 null。

### 通过使用`using`语句简化释放

你可以通过使用`using`语句简化需要检查`null`对象然后调用其`Dispose`方法的代码。一般来说，我建议使用`using`而不是手动调用`Dispose`，除非你需要更高级别的控制。

令人困惑的是，`using`关键字有两种用途：导入命名空间和生成一个`finally`语句，该语句在实现`IDisposable`接口的对象上调用`Dispose`。

编译器将`using`语句块转换为没有`catch`语句的`try`-`finally`语句。你可以使用嵌套的`try`语句；因此，如果你确实想要捕获任何异常，你可以这样做，如下面的代码示例所示：

```cs
using (FileStream file2 = File.OpenWrite(
  Path.Combine(path, "file2.txt")))
{
  using (StreamWriter writer2 = new StreamWriter(file2))
  {
    try
    {
      writer2.WriteLine("Welcome, .NET!");
    }
    catch(Exception ex)
    {
      WriteLine($"{ex.GetType()} says {ex.Message}");
    }
  } // automatically calls Dispose if the object is not null
} // automatically calls Dispose if the object is not null 
```

你甚至可以通过不明确指定`using`语句的大括号和缩进来进一步简化代码，如下面的代码所示：

```cs
using FileStream file2 = File.OpenWrite(
  Path.Combine(path, "file2.txt"));
using StreamWriter writer2 = new(file2);
try
{
  writer2.WriteLine("Welcome, .NET!");
}
catch(Exception ex)
{
  WriteLine($"{ex.GetType()} says {ex.Message}");
} 
```

## 压缩流

XML 相对冗长，因此在字节中占用的空间比纯文本多。让我们看看如何使用称为 GZIP 的常见压缩算法来压缩 XML：

1.  在`Program.cs`的顶部，导入用于处理压缩的命名空间，如下面的代码所示：

    ```cs
    using System.IO.Compression; // BrotliStream, GZipStream, CompressionMode 
    ```

1.  添加一个`WorkWithCompression`方法，该方法使用`GZipStream`实例创建一个包含与之前相同 XML 元素的压缩文件，然后在读取时解压缩并输出到控制台，如下面的代码所示：

    ```cs
    static void WorkWithCompression()
    {
      string fileExt = "gzip";
      // compress the XML output
      string filePath = Combine(
        CurrentDirectory, $"streams.**{fileExt}**");
      FileStream file = File.Create(filePath);
      Stream compressor = new GZipStream(file, CompressionMode.Compress);
      using (compressor)
      {
        using (XmlWriter xml = XmlWriter.Create(compressor))
        {
          xml.WriteStartDocument();
          xml.WriteStartElement("callsigns");
          foreach (string item in Viper.Callsigns)
          {
            xml.WriteElementString("callsign", item);
          }
          // the normal call to WriteEndElement is not necessary
          // because when the XmlWriter disposes, it will
          // automatically end any elements of any depth
        }
      } // also closes the underlying stream
      // output all the contents of the compressed file
      WriteLine("{0} contains {1:N0} bytes.",
        filePath, new FileInfo(filePath).Length);
      WriteLine($"The compressed contents:");
      WriteLine(File.ReadAllText(filePath));
      // read a compressed file
      WriteLine("Reading the compressed XML file:");
      file = File.Open(filePath, FileMode.Open);
      Stream decompressor = new GZipStream(file,
        CompressionMode.Decompress);
      using (decompressor)
      {
        using (XmlReader reader = XmlReader.Create(decompressor))
        {
          while (reader.Read()) // read the next XML node
          {
            // check if we are on an element node named callsign
            if ((reader.NodeType == XmlNodeType.Element)
              && (reader.Name == "callsign"))
            {
              reader.Read(); // move to the text inside element
              WriteLine($"{reader.Value}"); // read its value
            }
          }
        }
      }
    } 
    ```

1.  在`Program.cs`中，保留对`WorkWithXml`的调用，并添加对`WorkWithCompression`的调用，如下面的代码中突出显示的那样：

    ```cs
    // WorkWithText();
    **WorkWithXml();**
    **WorkWithCompression();** 
    ```

1.  运行代码并比较 XML 文件和压缩后的 XML 文件的大小。如以下编辑后的输出所示，压缩后的文件大小不到未压缩 XML 文件的一半。

    ```cs
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.xml contains 310 bytes.
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.gzip contains 150 bytes. 
    ```

## 使用 Brotli 算法压缩

在.NET Core 2.1 中，微软引入了 Brotli 压缩算法的实现。在性能上，Brotli 类似于 DEFLATE 和 GZIP 中使用的算法，但输出密度大约高出 20%。步骤如下：

1.  修改`WorkWithCompression`方法，使其具有一个可选参数来指示是否应使用 Brotli，并默认使用 Brotli，如下面的代码中突出显示的那样：

    ```cs
    static void WorkWithCompression(**bool** **useBrotli =** **true**)
    {
      string fileExt = **useBrotli ?** **"brotli"** **:** **"gzip"****;**
      // compress the XML output
      string filePath = Combine(
        CurrentDirectory, $"streams.{fileExt}");
      FileStream file = File.Create(filePath);
     **Stream compressor;**
    **if** **(useBrotli)**
     **{**
     **compressor =** **new** **BrotliStream(file, CompressionMode.Compress);**
     **}**
    **else**
     **{**
     **compressor =** **new** **GZipStream(file, CompressionMode.Compress);**
     **}**
      using (compressor)
      {
        using (XmlWriter xml = XmlWriter.Create(compressor))
        {
          xml.WriteStartDocument();
          xml.WriteStartElement("callsigns");
          foreach (string item in Viper.Callsigns)
          {
            xml.WriteElementString("callsign", item);
          }
        }
      } // also closes the underlying stream
      // output all the contents of the compressed file
      WriteLine("{0} contains {1:N0} bytes.",
        filePath, new FileInfo(filePath).Length);
      WriteLine($"The compressed contents:");
      WriteLine(File.ReadAllText(filePath));
      // read a compressed file
      WriteLine("Reading the compressed XML file:");
      file = File.Open(filePath, FileMode.Open);
     **Stream decompressor;**
    **if** **(useBrotli)**
     **{**
     **decompressor =** **new** **BrotliStream(**
     **file, CompressionMode.Decompress);**
     **}**
    **else**
     **{**
     **decompressor =** **new** **GZipStream(**
     **file, CompressionMode.Decompress);**
     **}**
      using (decompressor)
      {
        using (XmlReader reader = XmlReader.Create(decompressor))
        {
          while (reader.Read())
          {
            // check if we are on an element node named callsign
            if ((reader.NodeType == XmlNodeType.Element)
              && (reader.Name == "callsign"))
            {
              reader.Read(); // move to the text inside element
              WriteLine($"{reader.Value}"); // read its value
            }
          }
        }
      }
    } 
    ```

1.  在`Program.cs`顶部附近，调用`WorkWithCompression`两次，一次使用默认的 Brotli，一次使用 GZIP，如下面的代码所示：

    ```cs
    WorkWithCompression(); 
    WorkWithCompression(useBrotli: false); 
    ```

1.  运行代码并比较两个压缩后的 XML 文件的大小。如以下编辑后的输出所示，Brotli 的密度高出 21%以上。

    ```cs
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.brotli contains 118 bytes.
    /Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.gzip contains 150 bytes. 
    ```

# 编码和解码文本

文本字符可以用不同的方式表示。例如，字母表可以用摩尔斯电码编码成一系列点和划，以便通过电报线路传输。

类似地，计算机内部的文本以位（1 和 0）的形式存储，代表代码空间内的一个码位。大多数码位代表一个字符，但它们也可以有其他含义，如格式化。

例如，ASCII 有一个包含 128 个码位的代码空间。.NET 使用名为**Unicode**的标准来内部编码文本。Unicode 拥有超过一百万个码位。

有时，您需要将文本移出.NET，以便在不使用 Unicode 或使用 Unicode 变体的系统中使用，因此学习如何在编码之间转换非常重要。

下表列出了计算机常用的一些替代文本编码：

| 编码方式 | 描述 |
| --- | --- |
| ASCII | 此编码使用字节的低七位对有限范围的字符进行编码。 |
| UTF-8 | 此方式将每个 Unicode 码位表示为一个至四个字节的序列。 |
| UTF-7 | 与 UTF-8 相比，此方式在 7 位通道上更高效，但它存在安全性和健壮性问题，因此建议使用 UTF-8 而非 UTF-7。 |
| UTF-16 | 此方式将每个 Unicode 码位表示为一个或两个 16 位整数的序列。 |
| UTF-32 | 此方式将每个 Unicode 码位表示为一个 32 位整数，因此是一种固定长度编码，与其他所有变长 Unicode 编码不同。 |
| ANSI/ISO 编码 | 这提供了对各种代码页的支持，这些代码页用于支持特定的语言或一组语言。 |

**最佳实践**：在当今大多数情况下，UTF-8 是一个好的默认选择，这也是它实际上是默认编码的原因，即`Encoding.Default`。

## 将字符串编码为字节数组

让我们来探讨文本编码：

1.  使用您喜欢的代码编辑器，在`Chapter09`解决方案/工作区中添加一个名为`WorkingWithEncodings`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithEncodings`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，导入`System.Text`命名空间并静态导入`Console`类。

1.  添加语句以使用用户选择的编码对`string`进行编码，循环遍历每个字节，然后将其解码回`string`并输出，如下面的代码所示：

    ```cs
    WriteLine("Encodings"); 
    WriteLine("[1] ASCII");
    WriteLine("[2] UTF-7");
    WriteLine("[3] UTF-8");
    WriteLine("[4] UTF-16 (Unicode)");
    WriteLine("[5] UTF-32"); 
    WriteLine("[any other key] Default");
    // choose an encoding
    Write("Press a number to choose an encoding: "); 
    ConsoleKey number = ReadKey(intercept: false).Key; 
    WriteLine();
    WriteLine();
    Encoding encoder = number switch
    {
      ConsoleKey.D1 => Encoding.ASCII,
      ConsoleKey.D2 => Encoding.UTF7,
      ConsoleKey.D3 => Encoding.UTF8,
      ConsoleKey.D4 => Encoding.Unicode,
      ConsoleKey.D5 => Encoding.UTF32,
      _             => Encoding.Default
    };
    // define a string to encode
    string message = "Café cost: £4.39";
    // encode the string into a byte array
    byte[] encoded = encoder.GetBytes(message);
    // check how many bytes the encoding needed
    WriteLine("{0} uses {1:N0} bytes.",
      encoder.GetType().Name, encoded.Length);
    WriteLine();
    // enumerate each byte 
    WriteLine($"BYTE HEX CHAR"); 
    foreach (byte b in encoded)
    {
      WriteLine($"{b,4} {b.ToString("X"),4} {(char)b,5}");
    }
    // decode the byte array back into a string and display it
    string decoded = encoder.GetString(encoded); 
    WriteLine(decoded); 
    ```

1.  运行代码并注意避免使用`Encoding.UTF7`的警告，因为它不安全。当然，如果您需要使用该编码生成文本以与其他系统兼容，它需要在.NET 中保持为选项。

1.  按 1 选择 ASCII，并注意当输出字节时，英镑符号（£）和带重音的 e（é）无法在 ASCII 中表示，因此它使用问号代替。

    ```cs
    BYTE  HEX  CHAR
      67   43     C
      97   61     a
     102   66     f
      63   3F     ?
      32   20      
     111   6F     o
     115   73     s
     116   74     t
      58   3A     :
      32   20      
      63   3F     ?
      52   34     4
      46   2E     .
      51   33     3
      57   39     9
    Caf? cost: ?4.39 
    ```

1.  重新运行代码并按 3 选择 UTF-8，注意 UTF-8 为需要两个字节的两个字符额外需要两个字节（总共 18 字节而不是 16 字节），但它可以编码和解码é和£字符。

    ```cs
    UTF8EncodingSealed uses 18 bytes.
    BYTE  HEX  CHAR
      67   43     C
      97   61     a
     102   66     f
     195   C3     Ã
     169   A9     ©
      32   20      
     111   6F     o
     115   73     s
     116   74     t
      58   3A     :
      32   20      
     194   C2     Â
     163   A3     £
      52   34     4
      46   2E     .
      51   33     3
      57   39     9
    Café cost: £4.39 
    ```

1.  重新运行代码并按 4 选择 Unicode（UTF-16），注意 UTF-16 为每个字符需要两个字节，总共 32 字节，并且能够编码和解码é和£字符。这种编码被.NET 内部用于存储`char`和`string`值。

## 文件中的文本编码和解码

当使用流辅助类，如`StreamReader`和`StreamWriter`时，您可以指定要使用的编码。当您向辅助类写入时，文本将自动编码，当您从辅助类读取时，字节将自动解码。

要指定编码，请将编码作为第二个参数传递给辅助类型的构造函数，如下面的代码所示：

```cs
StreamReader reader = new(stream, Encoding.UTF8); 
StreamWriter writer = new(stream, Encoding.UTF8); 
```

**良好实践**：通常，您无法选择使用哪种编码，因为您将生成的文件供另一个系统使用。但是，如果可以，请选择使用最少字节数但能存储您所需所有字符的编码。

# 序列化对象图

**序列化**是将活动对象转换为使用指定格式的字节序列的过程。**反序列化**则是其逆过程。这样做是为了保存活动对象的当前状态，以便将来可以重新创建它。例如，保存游戏的当前状态，以便明天可以从同一位置继续。序列化的对象通常存储在文件或数据库中。

有数十种格式可供指定，但最常见的两种是**可扩展标记语言**（**XML**）和**JavaScript 对象表示法**（**JSON**）。

**最佳实践**：JSON 更紧凑，最适合 Web 和移动应用程序。XML 更冗长，但在更多遗留系统中得到更好的支持。使用 JSON 来最小化序列化对象图的大小。当向 Web 应用程序和移动应用程序发送对象图时，JSON 也是一个不错的选择，因为 JSON 是 JavaScript 的原生序列化格式，而移动应用程序通常通过有限的带宽进行调用，因此字节数很重要。

.NET 有多个类可以序列化和反序列化为 XML 和 JSON。我们将从`XmlSerializer`和`JsonSerializer`开始。

## 序列化为 XML

让我们从 XML 开始，这可能是目前世界上最常用的序列化格式。为了展示一个典型的例子，我们将定义一个自定义类来存储有关人员的信息，然后使用嵌套的`Person`实例列表创建一个对象图：

1.  使用您喜欢的代码编辑器，在`Chapter09`解决方案/工作区中添加一个名为`WorkingWithSerialization`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`WorkingWithSerialization`作为活动 OmniSharp 项目。

1.  添加一个名为`Person`的类，其中包含一个`protected`的`Salary`属性，这意味着它只能由自身和派生类访问。为了填充薪水，该类有一个构造函数，它有一个参数来设置初始薪水，如下所示：

    ```cs
    namespace Packt.Shared;
    public class Person
    {
      public Person(decimal initialSalary)
      {
        Salary = initialSalary;
      }
      public string? FirstName { get; set; }
      public string? LastName { get; set; }
      public DateTime DateOfBirth { get; set; }
      public HashSet<Person>? Children { get; set; }
      protected decimal Salary { get; set; }
    } 
    ```

1.  在`Program.cs`中，导入用于 XML 序列化的命名空间，并静态导入`Console`、`Environment`和`Path`类，如下所示：

    ```cs
    using System.Xml.Serialization; // XmlSerializer
    using Packt.Shared; // Person 
    using static System.Console; 
    using static System.Environment; 
    using static System.IO.Path; 
    ```

1.  添加语句以创建`Person`实例的对象图，如下所示：

    ```cs
    // create an object graph
    List<Person> people = new()
    {
      new(30000M) 
      {
        FirstName = "Alice",
        LastName = "Smith",
        DateOfBirth = new(1974, 3, 14)
      },
      new(40000M) 
      {
        FirstName = "Bob",
        LastName = "Jones",
        DateOfBirth = new(1969, 11, 23)
      },
      new(20000M)
      {
        FirstName = "Charlie",
        LastName = "Cox",
        DateOfBirth = new(1984, 5, 4),
        Children = new()
        {
          new(0M)
          {
            FirstName = "Sally",
            LastName = "Cox",
            DateOfBirth = new(2000, 7, 12)
          }
        }
      }
    };
    // create object that will format a List of Persons as XML
    XmlSerializer xs = new(people.GetType());
    // create a file to write to
    string path = Combine(CurrentDirectory, "people.xml");
    using (FileStream stream = File.Create(path))
    {
      // serialize the object graph to the stream
      xs.Serialize(stream, people);
    }
    WriteLine("Written {0:N0} bytes of XML to {1}",
      arg0: new FileInfo(path).Length,
      arg1: path);
    WriteLine();
    // Display the serialized object graph
    WriteLine(File.ReadAllText(path)); 
    ```

1.  运行代码，查看结果，并注意抛出了一个异常，如下所示：

    ```cs
    Unhandled Exception: System.InvalidOperationException: Packt.Shared.Person cannot be serialized because it does not have a parameterless constructor. 
    ```

1.  在`Person`中，添加一个语句来定义一个无参数构造函数，如下所示：

    ```cs
    public Person() { } 
    ```

    构造函数无需执行任何操作，但它必须存在，以便`XmlSerializer`在反序列化过程中调用它来实例化新的`Person`实例。

1.  重新运行代码并查看结果，并注意对象图被序列化为 XML 元素，如`<FirstName>Bob</FirstName>`，并且`Salary`属性未被包含，因为它不是`public`属性，如下所示：

    ```cs
    Written 752 bytes of XML to
    /Users/markjprice/Code/Chapter09/WorkingWithSerialization/people.xml
    <?xml version="1.0"?>
    <ArrayOfPerson  >
      <Person>
        <FirstName>Alice</FirstName>
        <LastName>Smith</LastName>
        <DateOfBirth>1974-03-14T00:00:00</DateOfBirth>
      </Person>
      <Person>
        <FirstName>Bob</FirstName>
        <LastName>Jones</LastName>
        <DateOfBirth>1969-11-23T00:00:00</DateOfBirth>
      </Person>
      <Person>
        <FirstName>Charlie</FirstName>
        <LastName>Cox</LastName>
        <DateOfBirth>1984-05-04T00:00:00</DateOfBirth>
        <Children>
          <Person>
            <FirstName>Sally</FirstName>
            <LastName>Cox</LastName>
            <DateOfBirth>2000-07-12T00:00:00</DateOfBirth>
          </Person>
        </Children>
      </Person>
    </ArrayOfPerson> 
    ```

## 生成紧凑的 XML

我们可以使用属性而不是元素来使 XML 更紧凑：

1.  在`Person`中，导入`System.Xml.Serialization`命名空间，以便您可以为某些属性装饰`[XmlAttribute]`属性。

1.  使用`[XmlAttribute]`属性修饰名字、姓氏和出生日期属性，并为每个属性设置一个简短名称，如下所示：

    ```cs
    **[****XmlAttribute(****"fname"****)****]**
    public string FirstName { get; set; }
    **[****XmlAttribute(****"lname"****)****]**
    public string LastName { get; set; }
    **[****XmlAttribute(****"dob"****)****]**
    public DateTime DateOfBirth { get; set; } 
    ```

1.  运行代码并注意文件大小已从 752 字节减少到 462 字节，通过将属性值输出为 XML 属性，节省了超过三分之一的存储空间，如下所示：

    ```cs
    Written 462 bytes of XML to /Users/markjprice/Code/Chapter09/ WorkingWithSerialization/people.xml
    <?xml version="1.0"?>
    <ArrayOfPerson  >
      <Person fname="Alice" lname="Smith" dob="1974-03-14T00:00:00" />
      <Person fname="Bob" lname="Jones" dob="1969-11-23T00:00:00" />
      <Person fname="Charlie" lname="Cox" dob="1984-05-04T00:00:00">
        <Children>
          <Person fname="Sally" lname="Cox" dob="2000-07-12T00:00:00" />
        </Children>
      </Person>
    </ArrayOfPerson> 
    ```

## 反序列化 XML 文件

现在让我们尝试将 XML 文件反序列化回内存中的活动对象：

1.  添加语句以打开 XML 文件，然后对其进行反序列化，如下面的代码所示：

    ```cs
    using (FileStream xmlLoad = File.Open(path, FileMode.Open))
    {
      // deserialize and cast the object graph into a List of Person
      List<Person>? loadedPeople =
        xs.Deserialize(xmlLoad) as List<Person>;
      if (loadedPeople is not null)
      {
        foreach (Person p in loadedPeople)
        {
          WriteLine("{0} has {1} children.", 
            p.LastName, p.Children?.Count ?? 0);
        }
      }
    } 
    ```

1.  运行代码并注意，人员成功地从 XML 文件加载并进行了枚举，如下面的输出所示：

    ```cs
    Smith has 0 children. 
    Jones has 0 children. 
    Cox has 1 children. 
    ```

还有许多其他属性可用于控制生成的 XML。

如果不使用任何注解，`XmlSerializer`在反序列化时会使用属性名称进行大小写不敏感匹配。

**良好实践**：使用`XmlSerializer`时，请记住只有公共字段和属性会被包含，且类型必须有一个无参构造函数。你可以通过属性来自定义输出。

## 使用 JSON 进行序列化

处理 JSON 序列化格式的最流行的.NET 库之一是 Newtonsoft.Json，也称为 Json.NET。它成熟且功能强大。让我们看看它的实际应用：

1.  在`WorkingWithSerialization`项目中，添加对`Newtonsoft.Json`最新版本的包引用，如下面的标记所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="Newtonsoft.Json" 
        Version="13.0.1" />
    </ItemGroup> 
    ```

1.  构建`WorkingWithSerialization`项目以恢复包。

1.  在`Program.cs`中，添加语句以创建一个文本文件，然后将人员序列化为 JSON 文件，如下面的代码所示：

    ```cs
    // create a file to write to
    string jsonPath = Combine(CurrentDirectory, "people.json");
    using (StreamWriter jsonStream = File.CreateText(jsonPath))
    {
      // create an object that will format as JSON
      Newtonsoft.Json.JsonSerializer jss = new();
      // serialize the object graph into a string
      jss.Serialize(jsonStream, people);
    }
    WriteLine();
    WriteLine("Written {0:N0} bytes of JSON to: {1}",
      arg0: new FileInfo(jsonPath).Length,
      arg1: jsonPath);
    // Display the serialized object graph
    WriteLine(File.ReadAllText(jsonPath)); 
    ```

1.  运行代码并注意，与包含元素的 XML 相比，JSON 所需的字节数不到一半。它甚至比使用属性的 XML 文件还要小，如下面的输出所示：

    ```cs
    Written 366 bytes of JSON to: /Users/markjprice/Code/Chapter09/ WorkingWithSerialization/people.json [{"FirstName":"Alice","LastName":"Smith","DateOfBirth":"1974-03-
    14T00:00:00","Children":null},{"FirstName":"Bob","LastName":"Jones","Date
    OfBirth":"1969-11-23T00:00:00","Children":null},{"FirstName":"Charlie","L astName":"Cox","DateOfBirth":"1984-05-04T00:00:00","Children":[{"FirstNam e":"Sally","LastName":"Cox","DateOfBirth":"2000-07-12T00:00:00","Children ":null}]}] 
    ```

## 高性能的 JSON 处理

.NET Core 3.0 引入了一个新的命名空间来处理 JSON，`System.Text.Json`，它通过利用`Span<T>`等 API 优化了性能。

此外，像 Json.NET 这样的旧库是基于读取 UTF-16 实现的。使用 UTF-8 读写 JSON 文档将更高效，因为大多数网络协议，包括 HTTP，都使用 UTF-8，你可以避免将 UTF-8 从 Json.NET 的 Unicode `string`值进行转码。

根据不同场景，微软通过新 API 实现了 1.3 倍至 5 倍的性能提升。

Json.NET 的原作者 James Newton-King 加入了微软，并与他们合作开发了新的 JSON 类型。正如他在讨论新 JSON API 的评论中所说，“Json.NET 不会消失”，如图*9.3*所示：

![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B17442_09_03.png)

图 9.3：Json.NET 原作者的一条评论

让我们看看如何使用新的 JSON API 来反序列化 JSON 文件：

1.  在`WorkingWithSerialization`项目中，在`Program.cs`中，导入新的 JSON 类以进行序列化，使用别名以避免与我们之前使用的 Json.NET 名称冲突，如下面的代码所示：

    ```cs
    using NewJson = System.Text.Json.JsonSerializer; 
    ```

1.  添加语句以打开 JSON 文件，反序列化它，并输出人员的姓名和子女数量，如下面的代码所示：

    ```cs
    using (FileStream jsonLoad = File.Open(jsonPath, FileMode.Open))
    {
      // deserialize object graph into a List of Person
      List<Person>? loadedPeople = 
        await NewJson.DeserializeAsync(utf8Json: jsonLoad,
          returnType: typeof(List<Person>)) as List<Person>;
      if (loadedPeople is not null)
      {
        foreach (Person p in loadedPeople)
        {
          WriteLine("{0} has {1} children.",
            p.LastName, p.Children?.Count ?? 0);
        }
      }
    } 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Smith has 0 children. 
    Jones has 0 children. 
    Cox has 1 children. 
    ```

**良好实践**：为提高开发效率和丰富功能集选择 Json.NET，或为性能选择`System.Text.Json`。

# 控制 JSON 处理

控制 JSON 处理的方式有很多选项，如下所示：

+   包括与排除字段。

+   设置大小写策略。

+   选择大小写敏感策略。

+   选择紧凑与美化空白。

让我们看看一些实际操作：

1.  使用您偏好的代码编辑器，在`Chapter09`解决方案/工作区中添加一个名为`WorkingWithJson`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithJson`作为活动 OmniSharp 项目。

1.  在`WorkingWithJson`项目中，在`Program.cs`中，删除现有代码，导入用于处理 JSON 的两个主要命名空间，然后静态导入`System.Console`、`System.Environment`和`System.IO.Path`类型，如下所示：

    ```cs
    using System.Text.Json; // JsonSerializer
    using System.Text.Json.Serialization; // [JsonInclude]
    using static System.Console;
    using static System.Environment;
    using static System.IO.Path; 
    ```

1.  在`Program.cs`底部，定义一个名为`Book`的类，如下所示：

    ```cs
    public class Book
    {
      // constructor to set non-nullable property
      public Book(string title)
      {
        Title = title;
      }
      // properties
      public string Title { get; set; }
      public string? Author { get; set; }
      // fields
      [JsonInclude] // include this field
      public DateOnly PublishDate;
      [JsonInclude] // include this field
      public DateTimeOffset Created;
      public ushort Pages;
    } 
    ```

1.  在`Book`类上方，添加语句以创建`Book`类的一个实例并将其序列化为 JSON，如下所示：

    ```cs
    Book csharp10 = new(title: 
      "C# 10 and .NET 6 - Modern Cross-platform Development")
    { 
      Author = "Mark J Price",
      PublishDate = new(year: 2021, month: 11, day: 9),
      Pages = 823,
      Created = DateTimeOffset.UtcNow,
    };
    JsonSerializerOptions options = new()
    {
      IncludeFields = true, // includes all fields
      PropertyNameCaseInsensitive = true,
      WriteIndented = true,
      PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    };
    string filePath = Combine(CurrentDirectory, "book.json");
    using (Stream fileStream = File.Create(filePath))
    {
      JsonSerializer.Serialize<Book>(
        utf8Json: fileStream, value: csharp10, options);
    }
    WriteLine("Written {0:N0} bytes of JSON to {1}",
      arg0: new FileInfo(filePath).Length,
      arg1: filePath);
    WriteLine();
    // Display the serialized object graph 
    WriteLine(File.ReadAllText(filePath)); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Written 315 bytes of JSON to C:\Code\Chapter09\WorkingWithJson\bin\Debug\net6.0\book.json
    {
      "title": "C# 10 and .NET 6 - Modern Cross-platform Development",
      "author": "Mark J Price",
      "publishDate": {
        "year": 2021,
        "month": 11,
        "day": 9,
        "dayOfWeek": 2,
        "dayOfYear": 313,
        "dayNumber": 738102
      },
      "created": "2021-08-20T08:07:02.3191648+00:00",
      "pages": 823
    } 
    ```

    注意以下事项：

    +   JSON 文件大小为 315 字节。

    +   成员名称使用驼峰式大小写，例如`publishDate`。这对后续在 JavaScript 浏览器中处理最为有利。

    +   由于设置的选项，所有字段均被包含，包括`pages`。

    +   JSON 被美化以提高人类可读性。

    +   `DateTimeOffset`值以单一标准字符串格式存储。

    +   `DateOnly`值存储为具有`year`和`month`等日期部分的子属性的对象。

1.  在`Program.cs`中，设置`JsonSerializerOptions`时，注释掉大小写策略设置，写入缩进，并包含字段。

1.  运行代码并查看结果，如下所示：

    ```cs
    Written 230 bytes of JSON to C:\Code\Chapter09\WorkingWithJson\bin\Debug\net6.0\book.json
    {"Title":"C# 10 and .NET 6 - Modern Cross-platform Development","Author":"Mark J Price","PublishDate":{"Year":2021,"Month":11,"Day":9,"DayOfWeek":2,"DayOfYear":313,"DayNumber":738102},"Created":"2021-08-20T08:12:31.6852484+00:00"} 
    ```

    注意以下事项：

    +   JSON 文件大小为 230 字节，减少了超过 25%。

    +   成员名称使用正常大小写，例如`PublishDate`。

    +   `Pages`字段缺失。其他字段因`PublishDate`和`Created`字段上的`[JsonInclude]`属性而被包含。

    +   JSON 紧凑，空白字符最少，以节省传输或存储的带宽。

## 用于处理 HTTP 响应的新 JSON 扩展方法

在.NET 5 中，微软对`System.Text.Json`命名空间中的类型进行了改进，例如为`HttpResponse`添加了扩展方法，您将在*第十六章*，*构建和消费 Web 服务*中看到。

## 从 Newtonsoft 迁移到新 JSON

如果您现有的代码使用了 Newtonsoft Json.NET 库，并希望迁移到新的`System.Text.Json`命名空间，那么微软有专门的文档指导，您可以在以下链接找到：

[`docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to`](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)

# 实践与探索

通过回答问题、进行实践操作并深入研究本章主题，测试你的知识和理解。

## 练习 9.1 – 测试你的知识

回答以下问题：

1.  使用`File`类和`FileInfo`类有何不同？

1.  流（stream）的`ReadByte`方法与`Read`方法有何区别？

1.  何时使用`StringReader`、`TextReader`和`StreamReader`类？

1.  `DeflateStream`类型有何作用？

1.  UTF-8 编码每个字符使用多少字节？

1.  对象图（object graph）是什么？

1.  哪种序列化格式最适合用于最小化空间需求？

1.  哪种序列化格式最适合用于跨平台兼容性？

1.  为何使用`string`值如`"\Code\Chapter01"`来表示路径是不妥的，应如何替代？

1.  关于 NuGet 包及其依赖关系的信息在哪里可以找到？

## 练习 9.2 – 实践 XML 序列化

在`Chapter09`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，该程序创建一个形状列表，使用序列化将其保存到文件系统中（使用 XML 格式），然后进行反序列化：

```cs
// create a list of Shapes to serialize
List<Shape> listOfShapes = new()
{
  new Circle { Colour = "Red", Radius = 2.5 },
  new Rectangle { Colour = "Blue", Height = 20.0, Width = 10.0 },
  new Circle { Colour = "Green", Radius = 8.0 },
  new Circle { Colour = "Purple", Radius = 12.3 },
  new Rectangle { Colour = "Blue", Height = 45.0, Width = 18.0 }
}; 
```

形状（Shapes）应有一个名为`Area`的只读属性，以便在反序列化时，能输出包含面积的形状列表，如下所示：

```cs
List<Shape> loadedShapesXml = 
  serializerXml.Deserialize(fileXml) as List<Shape>;
foreach (Shape item in loadedShapesXml)
{
  WriteLine("{0} is {1} and has an area of {2:N2}",
    item.GetType().Name, item.Colour, item.Area);
} 
```

运行控制台应用程序时，输出应如下所示：

```cs
Loading shapes from XML:
Circle is Red and has an area of 19.63 
Rectangle is Blue and has an area of 200.00 
Circle is Green and has an area of 201.06 
Circle is Purple and has an area of 475.29 
Rectangle is Blue and has an area of 810.00 
```

## 练习 9.3 – 探索主题

利用以下页面上的链接，深入了解本章涉及的主题：

[第九章 - 文件、流和序列化](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-9---working-with-files-streams-and-serialization)

# 总结

本章中，你学习了如何读写文本文件和 XML 文件，如何压缩和解压缩文件，如何编码和解码文本，以及如何将对象序列化为 JSON 和 XML（并反序列化回来）。

下一章，你将学习如何使用 Entity Framework Core 操作数据库。
