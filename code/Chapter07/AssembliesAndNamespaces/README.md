### Microsoft .NET project SDKs

Let’s explore the relationship between assemblies and namespaces:
1. Use your preferred code editor to create a new project, as defined in the following list:
   * Project template: **Console App** / `console`
   * Project file and folder: `AssembliesAndNamespaces`
     ```bash
     dotnet new console -n AssembliesAndNamespaces
     ```
  
2. Open `AssembliesAndNamespaces.csproj` and note that it is a typical project file for a .NET application, as shown in the following markup:
    ```xml
    <Project Sdk="Microsoft.NET.Sdk">
        <PropertyGroup>
            <OutputType>Exe</OutputType>
            <TargetFramework>net8.0</TargetFramework>
            <ImplicitUsings>enable</ImplicitUsings>
            <Nullable>enable</Nullable>
        </PropertyGroup>
    </Project>
    ```
3. After the `<PropertyGroup>` section, add a new `<ItemGroup>` section to statically import `System.Console` for all C# files using the implicit usings .NET SDK feature, as shown in the following markup:
```xml   
<ItemGroup>
  <Using Include="System.Console" Static="true" />
</ItemGroup>    
```