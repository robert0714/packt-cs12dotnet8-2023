# 7 Packaging and Distributing .NET Types
## The road to .NET 8
### Checking your .NET SDKs for updates
Microsoft introduced a command with .NET 6 to check the versions of .NET SDKs and runtimes that you have installed and warn you if any need updating. For example, enter the following command:
```bash
dotnet sdk check
```
You will see results, including the status of available updates, as shown in the following partial output:
```bash
.NET SDK:
版本           狀態
------------------
8.0.414      最新資訊。

使用 .NET 9.0.305 試用最新的 .NET SDK 功能。

.NET 執行階段:
名稱                                版本          狀態
-------------------------------------------------------------
Microsoft.NETCore.App             6.0.36      .NET 6.0 已不再支援。
Microsoft.WindowsDesktop.App      6.0.36      .NET 6.0 已不再支援。
Microsoft.NETCore.App             7.0.20      .NET 7.0 已不再支援。
Microsoft.WindowsDesktop.App      7.0.20      .NET 7.0 已不再支援。
Microsoft.AspNetCore.App          8.0.20      最新資訊。
Microsoft.NETCore.App             8.0.20      最新資訊。
Microsoft.WindowsDesktop.App      8.0.20      最新資訊。
```
## Understanding .NET components
### Assemblies, NuGet packages, and namespaces
An assembly is where a type is stored in the filesystem. Assemblies are a mechanism for deploying code. For example, the `System.Data.dll` assembly contains types for managing data. To use types in other assemblies, they must be referenced. Assemblies can be static (pre-created) or dynamic (generated at runtime). Dynamic assemblies are an advanced feature that we will not cover in this book. Assemblies can be compiled into a single file as a DLL (class library) or an EXE (console app).

Assemblies are distributed as **NuGet packages**, which are files that are downloadable from public online feeds and can contain multiple assemblies and other resources. You will also hear about **project SDKs**, **workloads**, and **platforms**, which are combinations of NuGet packages.

Microsoft’s NuGet feed is found here: https://www.nuget.org/.

### Dependent assemblies
If an assembly is compiled as a class library and provides types for other assemblies to use, then it has the file extension `.dll` (**dynamic link library**), and it cannot be executed standalone.

Likewise, if an assembly is compiled as an application, then it has the file extension `.exe` (**executable**) and can be executed standalone. Before .NET Core 3, console apps were compiled to `.dll` files and had to be executed by the `dotnet run` command or a host executable.

Any assembly can reference one or more class library assemblies as dependencies, but you cannot have circular references. So, assembly **B** cannot reference assembly **A** if assembly **A** already references assembly **B**. The compiler will warn you if you attempt to add a dependency reference that would cause a circular reference. Circular references are often a warning sign of poor code design. If you are sure that you need a circular reference, then use an interface to solve it.

#### Microsoft .NET project SDKs
By default, console applications have a dependency reference on the Microsoft .NET project SDK. This platform contains thousands of types in NuGet packages that almost all applications would need, such as the System.Int32 and System.String types.

When using .NET, you reference the dependency assemblies, NuGet packages, and platforms that [your application needs in a project file](./AssembliesAndNamespaces/).

### Namespaces and types in assemblies
Many common .NET types are in the `System.Runtime.dll` assembly. There is not always a one-to-one mapping between assemblies and namespaces. A single assembly can contain many namespaces and a namespace can be defined in many assemblies. You can see the relationship between some assemblies and the namespaces that they supply types for in *Table 7.1*:
| Assembly                 | Example namespaces                                     | Example types                 |
|--------------------------|--------------------------------------------------------|-------------------------------|
| System.Runtime.dll       | System, System.Collections, System.Collections.Generic | Int32, String, IEnumerable<T> |
| System.Console.dll       | System                                                 | Console                       |
| System.Threading.dll     | System.Threading                                       | Interlocked, Monitor, Mutex   |
| System.Xml.XDocument.dll | System.Xml.Linq                                        | XDocument, XElement, XNode    |

### NuGet packages
.NET is split into a set of packages, distributed using a Microsoft-supported package management technology named NuGet. Each of these packages represents a single assembly of the same name. For example, the System.Collections package contains the System.Collections.dll assembly.

The following are the benefits of packages:

* Packages can be easily distributed on public feeds.
* Packages can be reused.
* Packages can ship on their own schedule.
* Packages can be tested independently of other packages.
* Packages can support different OSes and CPUs by including multiple versions of the same assembly built for different OSes and CPUs.
* Packages can have dependencies specific to only one library.
* Apps are smaller because unreferenced packages aren’t part of the distribution. Table 7.2 lists some of the more important packages and their important types:
  
| Package              | Important types                   | 
|----------------------|-----------------------------------|
| System.Runtime       | Object, String, Int32, Array      |
| System.Collections   | List<T>, Dictionary<TKey, TValue> | 
| System.Net.Http      | HttpClient, HttpResponseMessage   | 
| System.IO.FileSystem | File, Directory                   |
| System.Reflection    | Assembly, TypeInfo, MethodInfo    | 

Table 7.2: Some important packages and their important types

### Understanding frameworks
There is a two-way relationship between frameworks and packages. Packages define the APIs, while frameworks group packages. A framework without any packages would not define any APIs.

.NET packages each support a set of frameworks. For example, the `System.IO.FileSystem` package version 4.3.0 supports the following frameworks:

* .NET Standard, version 1.3 or later
* .NET Framework, version 4.6 or later
* Six Mono and Xamarin platforms (for example, Xamarin.iOS)

> More Information: You can read the details at the following link: https://www.nuget.org/packages/System.IO.FileSystem/#supportedframeworks-body-tab.

### Importing a namespace to use a type
Let’s explore how namespaces are related to assemblies and types:
1. In the [AssembliesAndNamespaces project](./AssembliesAndNamespaces/Program.cs), in Program.cs, delete the existing statements and then enter the following code:
   ```chsarp
   XDocument doc = new();
   ```
   > Recent versions of code editors will often automatically add a namespace import statement to fix the problem I want you to see. Please delete the `using` statement that your code editor writes for you.
2. Build the project and note the compiler error message, as shown in the following output:
   ```bash
   CS0246 The type or namespace name 'XDocument' could not be found (are you missing a using directive or an assembly reference?)
   ```
   > The `XDocument` type is not recognized because we have not told the compiler what the namespace of the type is. Although this project already has a reference to the assembly that contains the type, we also need to either prefix the type name with its namespace, for example, `System.Xml.Linq.XDocument`, or import the namespace.

3. Click inside the `XDocument` class name. Your code editor displays a light bulb, showing that it recognizes the type and can automatically fix the problem for you.
4. Click the light bulb, and select `using System.Xml.Linq`; from the menu.
   
This will *import the namespace* by adding a `using` statement to the top of the file. Once a namespace is imported at the top of a code file, then all the types within the namespace are available for use in that code file by just typing their name, without the type name needing to be fully qualified by prefixing it with its namespace.

I like to add a comment after importing a namespace to remind me why I need to import that namespace, as shown in the following code:
```csharp
using System.Xml.Linq; // To use XDocument.
```

If you do not comment your namespaces, you or other developers will not know why they are imported and might delete them, breaking the code. Or they might never delete imported namespaces “just in case” they are needed, potentially cluttering the code unnecessarily. This is why most modern code editors have features to remove unused namespaces. This technique also subconsciously trains you, while you are learning, to remember which namespace you need to import to use a particular type or extension method.   

### Relating C# keywords to .NET types
One of the common questions I get from new C# programmers is, “What is the difference between string with a lowercase s and `String` with an uppercase S?”

The short answer is easy: none. The long answer is that all C# keywords that represent types like `string` or `int` are aliases for a .NET type in a class library assembly.

When you use the `string` keyword, the compiler recognizes it as a `System.String` type. When you use the `int` type, the compiler recognizes it as a `System.Int32` type.

Let’s see this in action with some code:

1. In Program.cs, declare two variables to hold string values, one using lowercase string and one using uppercase String, as shown in the following code:
   ```csharp
   string s1 = "Hello"; 
   String s2 = "World";
   WriteLine($"{s1} {s2}");
   ```
2. Run the AssembliesAndNamespaces project and note that string and String both work and literally mean the same thing.
3. In AssembliesAndNamespaces.csproj, add an entry to prevent the System namespace from being globally imported, as shown in the following markup:
   ```xml
   <ItemGroup>
       <Using Remove="System" />
   </ItemGroup>
   ```
4. In `Program.cs`, and in the **Error List** or **PROBLEMS** window, note the compiler error message, as shown in the following output:
   ```bash
   CS0246 The type or namespace name 'String' could not be found (are you missing a using directive or an assembly reference?)
   ```
5. At the top of Program.cs, import the System namespace with a using statement that will fix the error, as shown in the following code:
   ```csharp
   using System; // To use String.
   ```
> **Good Practice**: When you have a choice, use the C# keyword instead of the actual type because the keywords do not need a namespace to be imported.

### Mapping C# aliases to .NET types
Table 7.3 shows the 18 C# type keywords along with their actual .NET types:
| Keyword | .NET type      | Keyword | .NET type                    |
|---------|----------------|---------|------------------------------|
| string  | System.String  | char    | System.Char                  |
| sbyte   | System.SByte   | byte    | System.Byte                  |
| short   | System.Int16   | ushort  | System.UInt16                |
| int     | System.Int32   | uint    | System.UInt32                |
| long    | System.Int64   | ulong   | System.UInt64                |
| nint    | System.IntPtr  | nuint   | System.UIntPtr               |
| float   | System.Single  | double  | System.Double                |
| decimal | System.Decimal | bool    | System.Boolean               |
| object  | System.Object  | dynamic | System.Dynamic.DynamicObject |


Table 7.3: C# type keywords and their actual .NET types

Other .NET programming language compilers can do the same thing. For example, the Visual Basic .NET language has a type named Integer, which is its alias for System.Int32.

### Understanding native-sized integers
C# 9 introduced the `nint` and `nuint` keyword aliases for native-sized integers, meaning that the storage size for the integer value is platform-specific. They store a 32-bit integer in a 32-bit process and `sizeof()` returns 4 bytes; they store a 64-bit integer in a 64-bit process and `sizeof()` returns 8 bytes. The aliases represent pointers to the integer value in memory, which is why their .NET names are `IntPtr` and `UIntPtr`. The actual storage type will be either `System.Int32` or `System.Int64`, depending on the process.

In a 64-bit process, the following code:
```csharp
WriteLine($"Environment.Is64BitProcess = {Environment.Is64BitProcess}");
WriteLine($"int.MaxValue = {int.MaxValue:N0}");
WriteLine($"nint.MaxValue = {nint.MaxValue:N0}");
```
produces this output:
```bash
Environment.Is64BitProcess = True
int.MaxValue = 2,147,483,647
nint.MaxValue = 9,223,372,036,854,775,807
```

### Sharing code with legacy platforms using .NET Standard
Before .NET Standard, there were **Portable Class Libraries (PCLs)**. With PCLs, you could create a library of code and explicitly specify which platforms you want the library to support, such as Xamarin, Silverlight, and Windows 8. Your library could then use the intersection of APIs that are supported by the specified platforms.

Microsoft realized that this was unsustainable, so they created .NET Standard—a single API that all future .NET platforms would support. There are older versions of .NET Standard, but .NET Standard 2.0 was an attempt to unify all important recent .NET platforms. .NET Standard 2.1 was released in late 2019 but only .NET Core 3.0 and that year’s version of Xamarin support its new features. For the rest of this book, I will use the term .NET Standard to mean .NET Standard 2.0.

.NET Standard is like HTML5 in that they are both standards that a platform should support. Just as Google’s Chrome browser and Microsoft’s Edge browser implement the HTML5 standard, .NET Core, .NET Framework, and Xamarin all implement .NET Standard. If you want to create a library of types that will work across variants of legacy .NET, you can do so most easily with .NET Standard.

> **Good Practice**: Since many of the API additions in .NET Standard 2.1 required runtime changes, and .NET Framework is Microsoft’s legacy platform, which needs to remain as unchanging as possible, .NET Framework 4.8 remained on .NET Standard 2.0 rather than implementing .NET Standard 2.1. If you need to support .NET Framework customers, then you should create class libraries on .NET Standard 2.0, even though it is not the latest and does not support all the recent language and BCL new features.

Your choice of which .NET Standard version to target comes down to a balance between maximizing platform support and available functionality. A lower version supports more platforms but has a smaller set of APIs. A higher version supports fewer platforms but has a larger set of APIs. Generally, you should choose the lowest version that supports all the APIs that you need.

### Understanding defaults for class libraries with different SDKs
When using the `dotnet` SDK tool to create a class library, it might be useful to know which target framework will be used by default, as shown in Table 7.4:
| SDK           | Default target framework for new class libraries | 
|---------------|--------------------------------------------------|
| .NET Core 3.1 | netstandard2.0                                   |
| .NET 6        | net6.0                                           |
| .NET 7        | net7.0                                           |
| .NET 8        | net8.0                                           | 
 
Table 7.4: .NET SDKs and their default target framework for new class libraries

Of course, just because a class library targets a specific version of .NET by default, it does not mean you cannot change it after creating a class library project using the default template.

You can manually set the target framework to a value that supports the projects that need to reference that library, as shown in Table 7.5:

| Class library target framework | Can be used by projects that target                                                                                                                  |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| netstandard2.0                 | .NET Framework 4.6.1 or later, .NET Core 2 or later, .NET 5 or later, Mono 5.4 or later, Xamarin.Android 8 or later, and Xamarin.iOS 10.14 or later. |
| netstandard2.1                 | .NET Core 3 or later, .NET 5 or later, Mono 6.4 or later, Xamarin.Android 10 or later, and Xamarin.iOS 12.16 or later.                               |
| net6.0                         | .NET 6 or later.                                                                                                                                     |
| net7.0                         | .NET 7 or later.                                                                                                                                     |
| net8.0                         | .NET 8 or later.                                                                                                                                     |
Table 7.5: Class library target frameworks and the projects that can use them

> **Good Practice**: Always check the target framework of a class library and then manually change it to something more appropriate if necessary. Make a conscious decision about what it should be rather than accepting the default.

### Creating a .NET Standard class library
We will create a class library using .NET Standard 2.0 so that it can be used across all important .NET legacy platforms and cross-platform on Windows, macOS, and Linux operating systems, while also having access to a wide set of .NET APIs:

1. Use your preferred code editor to add a new **Class Library** / `classlib` project named `SharedLibrary` that targets .NET Standard 2.0 to the `Chapter07` solution:
   * If you use Visual Studio 2022, when prompted for the **Target Framework**, select **.NET Standard 2.0**, and then configure the startup project for the solution to the current selection.
   * If you use `Visual Studio Code`, include a switch to target .NET Standard 2.0, as shown in the following command:
     ```bash  
     dotnet new classlib -f netstandard2.0 # existed project add referneces
     dotnet new classlib -f netstandard2.0 -n SharedLibrary # creat new project named 'SharedLibrary' to be referneced
     ```
> **Good Practice**: If you need to create types that use new features in .NET 8, as well as types that only use .NET Standard 2.0 features, then you can create two separate class libraries: one targeting .NET Standard 2.0 and one targeting .NET 8.

An alternative to manually creating two class libraries is to create one that supports **multi-targeting**. If you would like me to add a section about multi-targeting to the next edition, please let me know. You can read about multi-targeting here: https://learn.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting.

### Controlling the .NET SDK
By default, executing `dotnet` commands uses the highest version installed .NET SDK. There may be times when you want to control which SDK is used.

For example, once .NET 9 becomes available in preview, starting in February 2024, or the final version becomes available in November 2024, you might install it. But you would probably want your experience to match the book steps, which use the .NET 8 SDK. But once you install a .NET 9 SDK, it will be used by default.

You can control the .NET SDK used by default by using a `global.json` file, which contains the version to use. The `dotnet` command searches the current folder and then each ancestor folder in turn for a `global.json` file to see if it should use a different .NET SDK version.

You do not need to complete the following steps, but if you want to try and do not already have .NET 6 SDK installed, then you can install it from the following link:

https://dotnet.microsoft.com/download/dotnet/6.0

1. Create a subdirectory/folder in the `Chapter07` folder named [`ControlSDK`](./ControlSDK/).
2. On Windows, start Command Prompt or Windows Terminal. On macOS, start Terminal. If you are using Visual Studio Code, then you can use the integrated terminal.
3. In the `ControlSDK` folder, at the command prompt or terminal, enter a command to list the installed .NET SDKs, as shown in the following command:
   ```bash
   dotnet --list-sdks
   ```
4. Note the results and the version number of the latest .NET 6 SDK installed, as shown highlighted in the following output:
   ```bash
   6.0.314 [C:\Program Files\dotnet\sdk]
   7.0.304 [C:\Program Files\dotnet\sdk]
   8.0.100 [C:\Program Files\dotnet\sdk]
   ```  
5. Create a global.json file that forces the use of the latest .NET Core 6.0 SDK that you have installed (which might be later than mine), as shown in the following command:
   ```bash
   dotnet new globaljson --sdk-version 6.0.314
   ```
6. Note the result, as shown in the following output:
   ```bash
   The template "global.json file" was created successfully.
   ```
7. Use your preferred code editor to open the global.json file and review its contents, as shown in the following markup:
   ```json
   {
      "sdk": {
         "version": "6.0.314"
      }
   }
   ```
   > For example, to open it with Visual Studio Code, enter the command code global.json.
8. In the `ControlSDK` folder, at the command prompt or terminal, enter a command to create a class library project, as shown in the following command:
   ```bash 
   dotnet new classlib
   ```
9. If you do not have the .NET 6 SDK installed, then you will see an error, as shown in the following output:
   ```bash
   Could not execute because the application was not found or a compatible .NET SDK is not installed.
   ```
10. If you do have the .NET 6 SDK installed, then a class library project will be created that targets .NET 6 by default, as shown highlighted in the following markup:
    ```xml
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
         <TargetFramework>net6.0</TargetFramework>
         <!-- <TargetFramework>net6.0</TargetFramework> -->
         <ImplicitUsings>enable</ImplicitUsings>
         <Nullable>enable</Nullable>
      </PropertyGroup>
    </Project>
    ```


## Publishing your code for deployment
There are three ways to publish and deploy a .NET application. They are:
* Framework-dependent deployment (FDD)
* Framework-dependent executable (FDE)
* Self-contained

FDD means you deploy a DLL that must be executed by the `dotnet` command-line tool. FDE means you deploy an EXE that can be run directly from the command line. Both require the appropriate version of the .NET runtime to be installed on the system.

Sometimes, you want to be able to give someone a USB stick containing your application built for their operating system and know that it can execute on their computer. You would want to perform a self-contained deployment. While the size of the deployment files will be larger, you’ll know that it will work.

### Creating a console app to publish
Let’s explore how to publish a console app:
1. Use your preferred code editor to add a new **Console App** / `console` project named [`DotNetEverywhere`](./DotNetEverywhere/) to the `Chapter07` solution. Make sure you target .NET 8.
2. Modify the project file to statically import the `System.Console` class in all C# files.
3. In [Program.cs](./DotNetEverywhere/Program.cs), delete the existing statements, and then add a statement to output a message saying the console app can run everywhere and some information about the operating system, as shown in the following code.
4. Run the `DotNetEverywhere` project and note the results when run on Windows 11, as shown in the following output:
   ```bash
   I can run everywhere!
   OS Version is Microsoft Windows NT 10.0.22000.0.
   I am Windows 11.
   Press any key to stop me.
   ```
5. In [`DotNetEverywhere.csproj`](./DotNetEverywhere/DotNetEverywhere.csproj), add the **runtime identifiers (RIDs)** to target five operating systems inside the `<PropertyGroup>` element, as shown highlighted in the following markup:
   * The `win10-x64` RID value means Windows 10 or Windows Server 2016 64-bit. You could also use the `win10-arm64` RID value to deploy to a Microsoft Surface Pro X.
   * The `osx-x64` RID value means macOS Sierra 10.12 or later. You can also specify version-specific RID values like `osx.10.15-x64` (Catalina), `osx.11.0-x64` (Big Sur on Intel), or `osx.11.0-arm64` (Big Sur on Apple Silicon).
   * The l`inux-x64` RID value means most desktop distributions of Linux, like Ubuntu, CentOS, Debian, or Fedora. Use `linux-arm` for Raspbian or Raspberry Pi OS 32-bit. Use `linux-arm64` for a Raspberry Pi running Ubuntu 64-bit.
> There are two elements that you can use to specify runtime identifiers. Use `<RuntimeIdentifier>` if you only need to specify one. Use `<RuntimeIdentifiers>` if you need to specify multiple, as we did in the preceding example. If you use the wrong one, then the compiler will give an error and it can be difficult to understand why with only one character difference!

### Understanding dotnet commands
When you install the .NET SDK, it includes a **command-line interface (CLI)** named dotnet.

The .NET CLI has commands that work on the current folder to create a new project using templates:

1. On Windows, start **Command Prompt** or **Windows Terminal**. On macOS, start **Terminal**. If you prefer to use Visual Studio 2022 or Visual Studio Code, then you can use the integrated terminal.
2. Enter the `dotnet new list` (or `dotnet new -l` or `dotnet new --list` with older SDKs) command to list your currently installed templates, the most common of which are shown in Table 7.6:  
   
   | Template Name                                | Short Name   | Language   |
   |----------------------------------------------|--------------|------------|
   | .NET MAUI App                                | maui         | C#         |
   | .NET MAUI Blazor App                         | maui-blazor  | C#         |
   | ASP.NET Core Empty                           | web          | C#, F#     |
   | ASP.NET Core gRPC Service                    | grpc         | C#         |
   | ASP.NET Core Web API                         | webapi       | C#, F#     |
   | ASP.NET Core Web API (native AOT)            | webapiaot    | C#         |
   | ASP.NET Core Web App (Model-View-Controller) | mvc          | C#, F#     |
   | Blazor Web App                               | blazor       | C#         |
   | Class Library                                | classlib     | C#, F#, VB |
   | Console App                                  | console      | C#, F#, VB |
   | EditorConfig File                            | editorconfig |            |
   | global.json File                             | globaljson   |            |
   | Solution File                                | sln          |            |
   | xUnit Test Project                           | xunit        |            |
   
   Table 7.6: Project template full and short names
> .NET MAUI projects are not supported for Linux. The team has said they have left that work to the open source community. If you need to create a truly cross-platform graphical app, then take a look at Avalonia at the following link: https://avaloniaui.net/.

### Getting information about .NET and its environment
It is useful to see what .NET SDKs and runtimes are currently installed, alongside information about the operating system, as shown in the following command:
```bash
dotnet --info
```
Note the results, as shown in the following partial output:
```bash
.NET SDK (reflecting any global.json):
 Version:   8.0.100
 Commit:    3fe444af72
Runtime Environment:
 OS Name:     Windows
 OS Version:  10.0.22621
 OS Platform: Windows
 RID:         win10-x64
 Base Path:   C:\Program Files\dotnet\sdk\8.0.100\
.NET workloads installed:
There are no installed workloads to display.
Host (useful for support):
  Version: 8.0.0
  Commit:  bc78804f5d
.NET SDKs installed:
  5.0.214 [C:\Program Files\dotnet\sdk]
  6.0.317 [C:\Program Files\dotnet\sdk]
  7.0.401 [C:\Program Files\dotnet\sdk]
  8.0.100 [C:\Program Files\dotnet\sdk]
.NET runtimes installed:
  Microsoft.AspNetCore.App 5.0.17 [...\dotnet\shared\Microsoft.AspNetCore.All]
...
```
### Managing projects using the dotnet CLI
The .NET CLI has the following commands that work on the project in the current folder, to manage the project:
* `dotnet help`: This shows the command-line help.
* `dotnet new`: This creates a new .NET project or file.
* `dotnet tool`: This installs or manages tools that extend the .NET experience.
* `dotnet workload`: This manages optional workloads like .NET MAUI.
* `dotnet restore`: This downloads dependencies for the project.
* `dotnet build`: This builds, aka compiles, a .NET project. A new switch introduced with .NET 8 is `--tl` (meaning terminal logger), which provides a modern output. For example, it provides real-time information about what the build is doing. You can learn more at the following link: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build#options.
* `dotnet build-server`: This interacts with servers started by a build.
* `dotnet msbuild`: This runs MS Build Engine commands.
* `dotnet clean`: This removes the temporary outputs from a build.
* `dotnet test`: This builds and then runs unit tests for the project.
* `dotnet run`: This builds and then runs the project.
* `dotnet pack`: This creates a NuGet package for the project.
* `dotnet publish`: This builds and then publishes the project, either with dependencies or as a self-contained application. In .NET 7 and earlier, this published the `Debug` configuration by default. In .NET 8 and later, it now publishes the `Release` configuration by default.
* `dotnet add`: This adds a reference to a package or class library to the project.
* `dotnet remove`: This removes a reference to a package or class library from the project.
* `dotnet list`: This lists the package or class library references for the project.

### Publishing a self-contained app
Now that you have seen some example `dotnet` tool commands, we can publish our cross-platform console app:

1. At the command prompt or terminal, make sure that you are in the [`DotNetEverywhere`](./DotNetEverywhere/) folder.
2. Enter a command to build and publish the self-contained release version of the console application for Windows 10, as shown in the following command:
   ```bash
   dotnet publish -c Release -r win10-x64 --self-contained
   ```
3. Note the build engine restores any needed packages, compiles the project source code into an assembly DLL, and creates a `publish` folder, as shown in the following output:
   ```bash   
   MSBuild version 17.8.0+14c24b2d3 for .NET
   Determining projects to restore...
   All projects are up-to-date for restore.
   DotNetEverywhere -> C:\cs12dotnet8\Chapter07\DotNetEverywhere\bin\Release\net8.0\win10-x64\DotNetEverywhere.dll
   DotNetEverywhere -> C:\cs12dotnet8\Chapter07\DotNetEverywhere\bin\Release\net8.0\win10-x64\publish\
   ```
4. Enter the following commands to build and publish the release versions for the macOS and Linux variants:
   ```bash
   dotnet publish -c Release -r osx-x64 --self-contained
   dotnet publish -c Release -r osx.11.0-arm64 --self-contained
   dotnet publish -c Release -r linux-x64 --self-contained
   dotnet publish -c Release -r linux-arm64 --self-contained
   ```
   > **Good Practice**: You could automate these commands by using a scripting language like PowerShell and execute the script file on any operating system using the cross-platform PowerShell Core. I have done this for you at the following link: https://github.com/markjprice/cs12dotnet8/tree/main/scripts/publish-scripts.

5. Open Windows File Explorer or a macOS Finder window, navigate to `DotNetEverywhere\bin\Release\net8.0`, and note the output folders for the five operating systems.
6. In the `win10-x64` folder, open the `publish` folder, and note all the supporting assemblies, like `Microsoft.CSharp.dll`.
7. Select the `DotNetEverywhere` executable file, and note it is 154 KB.
8. If you are on Windows, then double-click to execute the program and note the result, as shown in the following output:
   ```bash 
   I can run everywhere!
   OS Version is Microsoft Windows NT 10.0.22621.0.
   I am Windows 11.
   Press any key to stop me.
   ```
9. Press any key to close the console app and its window.
10. Note that the total size of the `publish` folder and all its files is 68.3 MB.
11. In the `osx.11.0-arm64` folder, select the `publish` folder, note all the supporting assemblies, and then select the `DotNetEverywhere` executable file. Note that the executable is 125 KB, and the `publish` folder is about 73.9 MB. There is no `.exe` file extension for published applications on macOS, so the filename will not have an extension.

If you copy any of those `publish` folders to the appropriate operating system, the console app will run; this is because it is a self-contained deployable .NET application. For example, here it is on macOS with Intel:
```bash      
I can run everywhere!
OS Version is Unix 13.5.2
I am macOS.
Press any key to stop me.
```
This example used a console app, but you could just as easily create an ASP.NET Core website or web service, or a Windows Forms or WPF app. Of course, you can only deploy Windows desktop apps to Windows computers, not Linux or macOS.

### Publishing a single-file app
If you can assume that .NET is already installed on the computer on which you want to run your app, then you can use the extra flags when you publish your app for release to say that it does not need to be self-contained and that you want to publish it as a single file (if possible), as shown in the following command (which must be entered on a single line):
```bash
dotnet publish -r win10-x64 -c Release --no-self-contained
/p:PublishSingleFile=true
```
This will generate two files: `DotNetEverywhere.exe` and `DotNetEverywhere.pdb`. The `.exe` file is the executable. The `.pdb` file is a program debug database file that stores debugging information.

If you prefer the .pdb file to be embedded in the .exe file, for example, to ensure it is deployed with its assembly, then add a `<DebugType>` element to the `<PropertyGroup>` element in your `.csproj` file and set it to embedded, as shown highlighted in the following markup:
```xml
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>net8.0</TargetFramework>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <RuntimeIdentifiers>
    win10-x64;osx-x64;osx.11.0-arm64;linux-x64;linux-arm64
  </RuntimeIdentifiers>
  <DebugType>embedded</DebugType>
</PropertyGroup>
```
If you cannot assume that .NET is already installed on a computer, then although Linux also only generates the two files, expect the following additional files for Windows: `coreclr.dll`, `clrjit.dll`, `clrcompression.dll`, and `mscordaccore.dll`.

Let’s see an example for Windows:
1. At the command prompt or terminal, in the DotNetEverywhere folder, enter the command to build the self-contained release version of the console app for Windows 10, as shown in the following command:
  ```bash
  dotnet publish -c Release -r win10-x64 --self-contained /p:PublishSingleFile=true
  ```
2. Navigate to the `DotNetEverywhere\bin\Release\net8.0\win10-x64\publish` folder and select the `DotNetEverywhere` executable file. Note that the executable is now 62.6 MB, and there is also a `.pdb` file that is 11 KB. The sizes of these files on your system will vary.

### Reducing the size of apps using app trimming
One of the problems with deploying a .NET app as a self-contained app is that the .NET libraries take up a lot of space. One of the biggest needs is to reduce the size of Blazor WebAssembly components because all the .NET libraries need to be downloaded to the browser.

Luckily, you can reduce this size by not packaging unused assemblies with your deployments. Introduced with .NET Core 3, the app trimming system can identify the assemblies needed by your code and remove those that are not needed. This was known as `copyused` trim mode.

With .NET 5, the trimming went further by removing individual types, and even members, like methods from within an assembly if they are not used. For example, with a **Hello World** console app, the `System.Console.dll` assembly is trimmed from 61.5 KB to 31.5 KB. This was known as `link` trim mode, but it was not enabled by default.

With .NET 6, Microsoft added annotations to their libraries to indicate how they can be safely trimmed, so the trimming of types and members was made the default.

With .NET 7, Microsoft renamed `link` to `full` and `copyused` to `partial`.

The catch is how well the trimming identifies unused assemblies, types, and members. If your code is dynamic, perhaps using reflection, then it might not work correctly, so Microsoft also allows manual control.

There are two ways to enable type-level and member-level, aka `full`, trimming. Since this level of trimming is the default with .NET 6 or later, all we need to do is enable trimming without setting a trim level or mode.

The first way is to add an element in the project file, as shown in the following markup:
```xml
<PublishTrimmed>true</PublishTrimmed> <!--Enable trimming.-->
```
The second way is to add a flag when publishing, as shown highlighted in the following command:
```bash
dotnet publish ... 
-p:PublishTrimmed=True
```
There are two ways to enable assembly-level, aka `partial`, trimming.

The first way is to add two elements in the project file, as shown in the following markup:
```xml
<PublishTrimmed>true</PublishTrimmed> <!--Enable trimming.-->
<TrimMode>partial</TrimMode> <!--Set assembly-level trimming.-->
```
The second way is to add two flags when publishing, as shown highlighted in the following command:
```bash
dotnet publish ... 
-p:PublishTrimmed=True -p:TrimMode=partial
```
### Controlling where build artifacts are created
Traditionally, each project has its own `bin` and `obj` subfolders where temporary files are created during the build process. When you publish, the files are created in the `bin` folder.

You might prefer to put all these temporary files and folders somewhere else. Introduced with .NET 8 is the ability to control where build artifacts are created. Let’s see how:

1. At the command prompt or terminal for the `Chapter07` folder, enter the following command:
   ```bash
   dotnet new buildprops --use-artifacts
   ```
2. Note the success message, as shown in the following output:
   ```bash
   The template "MSBuild Directory.Build.props file" was created successfully.
   ```
   > We could have created this file in the cs12dotnet8 folder, and it would then affect all projects in all chapters.
3. In the Chapter07 folder, open the Directory.Build.props file, as shown in the following markup:
   ```xml   
   <Project>
      <!-- See https://aka.ms/dotnet/msbuild/customize for more details on customizing your build -->
      <PropertyGroup>
         <ArtifactsPath>$(MSBuildThisFileDirectory)artifacts</ArtifactsPath>
      </PropertyGroup>
   </Project>
   ```
4. Build any project or the whole solution.
5. In the `Chapter07` folder, note there is now an `artifacts` folder that contains subfolders for any recently built projects.
6. Optionally, but recommended, is to delete this file or rename it to something like `Directory.Build.props.disabled` so that it does not affect the rest of this chapter.
   
> **Warning!** If you leave this build configuration enabled, then remember that your build artifacts are now created in this new folder structure.

## Native ahead-of-time compilation
Native AOT produces assemblies that are:
* **Self-contained**, meaning they can run on systems that do not have the .NET runtime installed.
* **Ahead-of-time (AOT) compiled to native code**, meaning a faster startup time and a potentially smaller memory footprint.

Native AOT compiles IL code to native code at the time of publishing, rather than at runtime using the **Just In Time (JIT)** compiler. But native AOT assemblies must target a specific runtime environment like Windows x64 or Linux Arm.

Since native AOT happens at publish time, you should remember that while you are debugging and working live on a project in your code editor, it is still using the runtime JIT compiler, not native AOT, even if you have AOT enabled in the project!

However, some features that are incompatible with native AOT will be disabled or throw exceptions, and a source analyzer is enabled to show warnings about potential code incompatibilities.

### Limitations of native AOT
Native AOT has limitations, some of which are shown in the following list:
* No dynamic loading of assemblies.
* No runtime code generation, for example, using `System.Reflection.Emit`.
* It requires trimming, which has its own limitations, as we covered in the previous section.
* They must be self-contained, so they must embed any libraries they call, which increases their size.


Although your own assemblies might not use the features listed above, major parts of .NET itself do. For example, ASP.NET Core MVC (including Web API services that use controllers) and EF Core do runtime code generation to implement their functionality.

The .NET teams are hard at work making as much of .NET compatible with native AOT as possible, as soon as possible. But .NET 8 only includes basic support for ASP.NET Core if you use Minimal APIs, and no support for EF Core.

My guess is that .NET 9 will include support for ASP.NET Core MVC and some parts of EF Core, but it could take until .NET 10 before we can all confidently use most of .NET and know we can build our assemblies with native AOT to gain the benefits.

The native AOT publishing process includes code analyzers to warn you if you use any features that are not supported, but not all packages have been annotated to work well with these yet.

The most common annotation used to indicate that a type or member does not support AOT is the `[RequiresDynamicCode]` attribute.

> **More Information**: You can learn more about AOT warnings at the following link: https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/fixing-warnings.

### Reflection and native AOT
Reflection is frequently used for runtime inspection of type `metadata`, dynamic invocation of members, and code generation.

Native AOT does allow some reflection features, but the trimming performed during the native AOT compilation process cannot statically determine when a type has members that might be only accessed via reflection. These members would be removed by AOT, which would then cause a runtime exception.

> Good Practice: Developers must annotate their types with `[DynamicallyAccessedMembers]` to indicate a member that is only dynamically accessed via reflection and should therefore be left untrimmed.

### Requirements for native AOT
There are additional requirements for different operating systems:

* On Windows, you must install the Visual Studio 2022 **Desktop development with C++** workload with all default components.
* On Linux, you must install the compiler toolchain and developer packages for libraries that the .NET runtime depends on. For example, for Ubuntu 18.04 or later: `sudo apt-get install clang zlib1g-dev`.
  
> **Warning!** Cross-platform native AOT publishing is not supported. This means that you must run the publish on the operating system that you will deploy to. For example, you cannot publish a native AOT project on Linux to later run on Windows, and vice versa.

### Enabling native AOT for a project
To enable native AOT publishing in a project, add the `<PublishAot>` element to the project file, as shown highlighted in the following markup:
```xml
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <PublishAot>true</PublishAot>
```
### Building a native AOT project
Now let’s see a practical example using the new AOT option for a console app:
1. In the solution named Chapter07, add a native AOT-compatible console app project, as defined in the following list:
   * Project template: Console App / `console --aot`
   * Solution file and folder: `Chapter07`
   * Project file and folder: [`AotConsole`](./AotConsole/)
   * Do not use top-level statements: Cleared
   * Enable native AOT publish: Selected
   > If your code editor does not yet provide the option for AOT, then create a traditional console app and then you will need to manually enable AOT as shown in step 2, or use the `dotnet` CLI.
2. In the project file, note that native AOT publishing is enabled, as well as invariant globalization, as shown highlighted in the following markup:
   ```xml
   <Project Sdk="Microsoft.NET.Sdk.Web">
      <PropertyGroup>
         <TargetFramework>net8.0</TargetFramework>
         <Nullable>enable</Nullable>
         <ImplicitUsings>enable</ImplicitUsings>
         <PublishAot>true</PublishAot>
         <InvariantGlobalization>true</InvariantGlobalization>
      </PropertyGroup>
   </Project>
   ```
   > Explicitly setting invariant globalization to `true` is new in the **Console App** project template with .NET 8. It is designed to make a console app non-culture-specific so it can be deployed anywhere in the world and have the same behavior. If you set this property to false, or if the element is missing, then the console app will default to the culture of the current computer it is hosted on. You can read more about invariant globalization mode at the following link: https://github.com/dotnet/runtime/blob/main/docs/design/features/globalization-invariant-mode.md.

3. Modify the project file to statically import the `System.Console` class in all C# files.
4. In [`Program.cs`](./AotConsole/Program.cs), delete any existing statements, and then add statements to output the current culture and OS version, as shown in the following code:
   ```csharp 
   using System.Globalization; // To use CultureInfo.
   WriteLine("This is an ahead-of-time (AOT) compiled console app.");
   WriteLine("Current culture: {0}", CultureInfo.CurrentCulture.DisplayName);
   WriteLine("OS version: {0}", Environment.OSVersion);
   Write("Press any key to exit.");
   ReadKey(intercept: true); // Do not output the key that was pressed.
   ```
5. Run the console app project and note the culture is invariant, as shown in the following output:
   ```bash
   This is an ahead-of-time (AOT) compiled console app.
   Current culture: Invariant Language (Invariant Country)
   OS version: Microsoft Windows NT 10.0.22621.0
   ```
> Warning! Actually, the console app is not being AOT compiled; it is still currently JIT-compiled because we have not yet published it.

### Publishing a native AOT project
A console app that functions correctly during development when the code is untrimmed and JIT-compiled could still fail once you publish it using native AOT because then the code is trimmed and JIT-compiled and, therefore, it is a different code with different behavior. You should, therefore, perform a publish before assuming your project will work.

If your project does not produce any AOT warnings at publish time, you can then be confident that your service will work after publishing for AOT.

Let’s publish our console app:
1. At the command prompt or terminal for the `AotConsole` project, publish the console app using native AOT, as shown in the following command:
   ```bash
   dotnet publish
   ```
2. Note the message about generating native code, as shown in the following output:
   ```bash
   MSBuild version 17.8.0+4ce2ff1f8 for .NET
   Determining projects to restore...
   Restored C:\cs12dotnet8\Chapter07\AotConsole\AotConsole.csproj (in 173 ms).
   AotConsole -> C:\cs12dotnet8\Chapter07\AotConsole\bin\Release\net8.0\win-x64\AotConsole.dll
   Generating native code
   AotConsole -> C:\cs12dotnet8\Chapter07\AotConsole\bin\Release\net8.0\win-x64\publish\
   ```  
3. Start **File Explorer**, open the `bin\Release\net8.0\win-x64\publish` folder, and note the `AotConsole.exe` file is about 1.2 MB. The `AotConsole.pdb` file is only needed for debugging.
4. Run the `AotConsole.exe` and note the console app has the same behavior as before.
5. In [`Program.cs`](./AotConsole/Program.cs), import namespaces to work with dynamic code assemblies, as shown in the following code:
   ```csharp
   using System.Reflection; // To use AssemblyName.
   using System.Reflection.Emit; // To use AssemblyBuilder.
   ```
6. In [`Program.cs`](./AotConsole/Program.cs), create a dynamic assembly builder, as shown in the following code:
   ```csharp
   AssemblyBuilder ab = AssemblyBuilder.DefineDynamicAssembly(
   new AssemblyName("MyAssembly"), AssemblyBuilderAccess.Run);
   ```
7. At the command prompt or terminal for the `AotConsole` project, publish the console app using native AOT, as shown in the following command:
   ```bash   
   dotnet publish
   ```
8. Note the warning about calling the `DefineDynamicAssembly` method, which the .NET team has decorated with the [RequiresDynamicCode] attribute, as shown in the following output:
   ```bash   
   C:\cs12dotnet8\Chapter07\AotConsole\Program.cs(9,22): warning IL3050: Using member 'System.Reflection.Emit.AssemblyBuilder.DefineDynamicAssembly(AssemblyName, AssemblyBuilderAccess)' which has 'RequiresDynamicCodeAttribute' can break functionality when AOT compiling. Defining a dynamic assembly requires dynamic code. [C:\cs12dotnet8\Chapter07\AotConsole\AotConsole.csproj]
   ```
9. Comment out the statement that we cannot use in an AOT project.
> More Information: You can learn more about native AOT at the following link: https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/.

## Decompiling .NET assemblies
One of the best ways to learn how to code for .NET is to see how professionals do it. Most code editors have an extension for decompiling .NET assemblies. Visual Studio 2022 and Visual Studio Code can use the **ILSpy** extension. JetBrains Rider has a built-in **IL Viewer** tool.

> **Good Practice**: You could decompile someone else’s assemblies for non-learning purposes, like copying their code for use in your own production library or application, but remember that you are viewing their intellectual property, so please respect that.

## Packaging your libraries for NuGet distribution
Before we learn how to create and package our own libraries, we will review how a project can use an existing package.

### Referencing a NuGet package
Let’s say that you want to add a package created by a third-party developer, for example, `Newtonsoft.Json`, a popular package for working with the **JavaScript Object Notation (JSON)** serialization format:

1. In the [`AssembliesAndNamespaces`](./AssembliesAndNamespaces/) project, add a reference to the Newtonsoft.Json NuGet package, either using the GUI for Visual Studio 2022 or the `dotnet add package` command for Visual Studio Code.
2. Open the `AssembliesAndNamespaces.csproj` file and note that a package reference has been added, as shown in the following markup:
   ```xml
   <ItemGroup>
      <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
   </ItemGroup>
   ```
If you have a more recent version of the `Newtonsoft.Json` package, then it has been updated since this chapter was written.

### Fixing dependencies
To consistently restore packages and write reliable code, it’s important that you **fix dependencies**. Fixing dependencies means you are using the same family of packages released for a specific version of .NET, for example, SQLite for .NET 8, as shown highlighted in the following markup:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Version="8.0.0"
      Include="Microsoft.EntityFrameworkCore.Sqlite" />
  </ItemGroup>
</Project>
```
To fix dependencies, every package should have a single version with no additional qualifiers. Additional qualifiers include betas (`beta1`), release candidates (`rc4`), and wildcards (`*`).

Wildcards allow future versions to be automatically referenced and used because they always represent the most recent release. But wildcards are, therefore, dangerous because they could result in the use of future incompatible packages that break your code.

This can be worth the risk while writing a book where new preview versions are released every month and you do not want to keep updating the preview package references, as I did during 2023, and as shown in the following markup:
```xml
<PackageReference Version="8.0.0-preview.*"
  Include="Microsoft.EntityFrameworkCore.Sqlite" />
```  
To also automatically use the release candidates that arrive in September and October each year, you can make the pattern even more flexible, as shown in the following markup:
```xml
<PackageReference Version="8.0-*"
  Include="Microsoft.EntityFrameworkCore.Sqlite" />
```  
If you use the `dotnet add package` command or Visual Studio’s **Manage NuGet Packages**, then it will by default use the latest specific version of a package. But if you copy and paste configuration from a blog article or manually add a reference yourself, you might include wildcard qualifiers.

The following dependencies are examples of NuGet package references that are not fixed and, therefore, should be avoided unless you know the implications:
```xml
<PackageReference Include="System.Net.Http" Version="4.1.0-*" />
<PackageReference Include="Newtonsoft.Json" Version="13.0.2-beta1" />
```
> **Good Practice**: Microsoft guarantees that if you fix your dependencies to what ships with a specific version of .NET, for example, 8.0.0, those packages will all work together. Almost always fix your dependencies, especially in production deployments.

### Packaging a library for NuGet
Now, let’s package the `SharedLibrary` project that you created earlier:

1. In the `SharedLibrary` project, note that the class library targets .NET Standard 2.0 and, therefore, by default, uses the C# 7.3 compiler. Explicitly specify the C# 12 compiler, as shown in the following markup:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <LangVersion>12</LangVersion>
  </PropertyGroup>
</Project>
```
2. In the `SharedLibrary` project, rename the `Class1.cs` file `StringExtensions.cs`.
3. Modify its contents to provide some useful extension methods for validating various text values using regular expressions, as shown in the following code:
```csharp 
using System.Text.RegularExpressions; // To use Regex.
namespace Packt.Shared;
public static class StringExtensions
{
  public static bool IsValidXmlTag(this string input)
  {
    return Regex.IsMatch(input,
      @"^<([a-z]+)([^<]+)*(?:>(.*)<\/\1>|\s+\/>)$");
  }
  public static bool IsValidPassword(this string input)
  {
    // Minimum of eight valid characters.
return Regex.IsMatch(input, "^[a-zA-Z0-9_-]{8,}$");
  }
  public static bool IsValidHex(this string input)
  {
    // Three or six valid hex number characters.
return Regex.IsMatch(input,
      "^#?([a-fA-F0-9]{3}|[a-fA-F0-9]{6})$");
  }
}
```
> You will learn how to write regular expressions in Chapter 8, Working with Common .NET Types.

4. In [`SharedLibrary.csproj`](./SharedLibrary/SharedLibrary.csproj), modify its contents, as shown highlighted in the following markup, and note the following:
   * `PackageId` must be globally unique, so you must use a different value if you want to publish this NuGet package to the https://www.nuget.org/ public feed for others to reference and download.
   * `PackageLicenseExpression` must be a value from https://spdx.org/licenses/, or you could specify a custom license.
   > **Warning!** If you rely on IntelliSense to edit the file, then it could mislead you to use deprecated tag names. For example, <PackageIconUrl> is deprecated in favor of <PackageIcon>. Sometimes, you cannot trust automated tools to help you correctly! The recommended tag names are documented in the MSBuild Property column in the table found at the following link: https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target.
   * All the other elements are self-explanatory:
      ```xml
      <Project Sdk="Microsoft.NET.Sdk">
         <PropertyGroup>
            <TargetFramework>netstandard2.0</TargetFramework>
            <LangVersion>12</LangVersion>
            <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
            <PackageId>Packt.CSdotnet.SharedLibrary</PackageId>
            <PackageVersion>8.0.0.0</PackageVersion>
            <Title>C# 12 and .NET 8 Shared Library</Title>
            <Authors>Mark J Price</Authors>
            <PackageLicenseExpression>
               MS-PL
            </PackageLicenseExpression>
            <PackageProjectUrl>
               https://github.com/markjprice/cs12dotnet8
            </PackageProjectUrl>
            <PackageReadmeFile>readme.md</PackageReadmeFile>
            <PackageIcon>packt-csdotnet-sharedlibrary.png</PackageIcon>
            <PackageRequireLicenseAcceptance>true</PackageRequireLicenseAcceptance>
            <PackageReleaseNotes>
               Example shared library packaged for NuGet.
            </PackageReleaseNotes>
            <Description>
               Three extension methods to validate a string value.
            </Description>
            <Copyright>
               Copyright © 2016-2023 Packt Publishing Limited
            </Copyright>
            <PackageTags>string extensions packt csharp dotnet</PackageTags>
         </PropertyGroup>
         <ItemGroup>
            <None Include="packt-csdotnet-sharedlibrary.png" 
                  PackagePath="\" Pack="true" />
            <None Include="readme.md"
                  PackagePath="\" Pack="true" />
         </ItemGroup>
      </Project>
      ```
      > `<None>` represents a file that does not participate in the build process. `Pack="true"` means the file will be included in the NuGet package created in the specified package path location. You can learn more at the following link: https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets#packing-an-icon-image-file.
   > **Good Practice**: Configuration property values that are `true` or `false` values cannot have any whitespace, so the `<PackageRequireLicenseAcceptance>` entry cannot have a carriage return and indentation, as shown in the preceding markup.
5. Download the icon file and save it in the `SharedLibrary` project folder from the following link: https://github.com/markjprice/cs12dotnet8/blob/main/code/Chapter07/SharedLibrary/packt-csdotnet-sharedlibrary.png.
6. In the `SharedLibrary` project folder, create a file named readme.md, with some basic information about the package, as shown in the following markup:
```markdown
# README for C# 12 and .NET 8 Shared Library
This is a shared library that readers build in the book, 
*C# 12 and .NET 8 - Modern Cross-Platform Development Fundamentals*.
```
7. Build the release assembly:
   * In Visual Studio 2022, select Release in the toolbar, and then navigate to **Build | Build SharedLibrary**.
   * In Visual Studio Code, in Terminal, enter `dotnet build -c Release`.

If we had not set `<GeneratePackageOnBuild>` to `true` in the project file, then we would have to create a NuGet package manually using the following additional steps:
* In Visual Studio 2022, navigate to **Build | Pack SharedLibrary**.
* In Visual Studio Code, in **Terminal**, enter `dotnet pack -c Release`.

## Publishing a package to a public NuGet feed
If you want everyone to be able to download and use your NuGet package, then you must upload it to a public NuGet feed like Microsoft’s:
1. Start your favorite browser and navigate to the following link: https://www.nuget.org/packages/manage/upload.
2. You will need to sign up for, and then sign in with, a Microsoft account at https://www.nuget.org/ if you want to upload a NuGet package for other developers to reference as a dependency package.
3. Click the Browse... button and select the `.nupkg` file that was created by generating the NuGet package. The folder path should be `cs12dotnet8\Chapter07\SharedLibrary\bin\Release` and the file is named `Packt.CSdotnet.SharedLibrary.8.0.0.nupkg`.
4. Verify that the information you entered in the `SharedLibrary.csproj` file has been correctly filled in, and then click **Submit**.
5. Wait a few seconds, and you will see a success message showing that your package has been uploaded
   > Good Practice: If you get an error, then review the project file for mistakes, or read more information about the PackageReference format at https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets.

6. Click the **Frameworks** tab, and note that because we targeted .NET Standard 2.0, our class library can be used by every .NET platform   
### Publishing a package to a private NuGet feed
Organizations can host their own private NuGet feeds. This can be a handy way for many developer teams to share work. You can read more at the following link:

https://learn.microsoft.com/en-us/nuget/hosting-packages/overview

### Exploring NuGet packages with a tool
A handy tool named NuGet Package Explorer for opening and reviewing more details about a NuGet package was created by Uno Platform. As well as being a website, it can be installed as a cross-platform app. Let’s see what it can do:
1. Start your favorite browser and navigate to the following link: https://nuget.info.
2. In the search box, enter `Packt.CSdotnet.SharedLibrary`.
3. Select the package v8.0.0 published by **Mark J Price** and then click the **Open** button.
4. In the Contents section, expand the `lib` folder and the `netstandard2.0` folder.
5. Select `SharedLibrary.dll`, and note the details
6. If you want to use this tool locally in the future, click the install button in your browser.
7. Close your browser.
   
Not all browsers support installing web apps like this. I recommend Chrome for testing and development.

### Testing your class library package
You will now test your uploaded package by referencing it in the `AssembliesAndNamespaces` project:
1. In the [`AssembliesAndNamespaces`](./AssembliesAndNamespaces/) project, add a reference to your (or my) package, as shown highlighted in the following markup:
   ```xml
   <ItemGroup>
      <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
      <PackageReference Include="Packt.CSdotnet.SharedLibrary" 
                        Version="8.0.0" />
   </ItemGroup>
   ```
2. Build the `AssembliesAndNamespaces` project.
3. In `Program.cs`, import the `Packt.Shared` namespace.
4. In `Program.cs`, prompt the user to enter some `string` values, and then validate them using the extension methods in the package, as shown in the following code:
   ```csharp
   Write("Enter a color value in hex: "); 
   string? hex = ReadLine();
   WriteLine("Is {0} a valid color value? {1}",
   arg0: hex, arg1: hex.IsValidHex());
   Write("Enter a XML element: "); 
   string? xmlTag = ReadLine();
   WriteLine("Is {0} a valid XML element? {1}", 
   arg0: xmlTag, arg1: xmlTag.IsValidXmlTag());
   Write("Enter a password: "); 
   string? password = ReadLine();
   WriteLine("Is {0} a valid password? {1}",
   arg0: password, arg1: password.IsValidPassword());
   ```
5. Run the `AssembliesAndNamespaces` project, enter some values as prompted, and view the results, as shown in the following output:
   ```bash
   Enter a color value in hex: 00ffc8 
   Is 00ffc8 a valid color value? True
   Enter an XML element: <h1 class="<" />
   Is <h1 class="<" /> a valid XML element? False 
   Enter a password: secretsauce
   Is secretsauce a valid password? True
   ```