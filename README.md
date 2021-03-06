# Frosting

[![AppVeyor branch](https://img.shields.io/appveyor/ci/cakebuild/frosting/develop.svg)](https://ci.appveyor.com/project/cakebuild/frosting/branch/develop) 
[![MyGet](https://img.shields.io/myget/cake/vpre/Cake.Frosting.svg?label=myget)](https://www.myget.org/feed/cake/package/nuget/Cake.Frosting)

A .NET Core host for Cake, that allows you to write your build scripts as a 
portable console application (`netstandard1.0`). Frosting is currently 
in alpha, but more information, documentation and samples will be added soon.

**Expect things to move around initially. Especially naming of things.**

## Table of Contents

1. [Example](https://github.com/cake-build/frosting#example)
2. [Acknowledgement](https://github.com/cake-build/frosting#acknowledgement)
3. [License](https://github.com/cake-build/frosting#license)
4. [Thanks](https://github.com/cake-build/frosting#thanks)
5. [Code of Conduct](https://github.com/cake-build/frosting#code-of-conduct)
6. [.NET Foundation](https://github.com/cake-build/frosting#net-foundation)

## Example

### 1. Add NuGet.Config

Start by adding a `NuGet.Config` file to your project.  
The reason for this is that Cake.Frosting is in preview at the moment and not
available on [nuget.org](https://nuget.org).

```xml
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="myget.org" value="https://www.myget.org/F/cake/api/v3/index.json" protocolVersion="3" />
  </packageSources>
</configuration>
```

### 2. Add Build.csproj

Now create a `Build.csproj` file that will tell `dotnet` what dependencies are required
to build our program. The `Cake.Frosting` package will decide what version of `Cake.Core` and `Cake.Common`
you will get.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp1.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Cake.Frosting" Version="0.1.0-alpha0034" />
  </ItemGroup>

</Project>

```

### 3. Add global.json

To make sure that we use the correct version of the .NET Core SDK when Building
and running our build script, we will need to add a `global.json`.

```json
{
  "sdk": {
    "version": "1.0.0-rc4-004771"
  }
}
```

### 3. Add Program.cs

For the sake of keeping the example simple, all classes are listed after each other, 
but you should of course treat the source code of your build scripts like any other
code and divide them up in individual files.

```csharp
using Cake.Common.Diagnostics;
using Cake.Core;
using Cake.Core.Diagnostics;
using Cake.Frosting;

public class Program
{
    public static int Main(string[] args)
    {
        // Create the host.
        var host = new CakeHostBuilder()
            .WithArguments(args)
            .ConfigureServices(services =>
            {
                // Use a custom settings class.
                services.UseContext<MySettings>();

                // Use a custom lifetime to initialize the context.
                services.UseLifetime<MyLifetime>();

                // Use the parent directory as the working directory.
                services.UseWorkingDirectory("..");
            })
            .Build();

        // Run the host.
        return host.Run();
    }
}

public class MySettings : FrostingContext
{
    public bool Magic { get; set; }

    public MySettings(ICakeContext context)
        : base(context)
    {
        // You could initialize the context here,
        // but it's recommended to use a FrostingLifetime
        // since the context will be created when running
        // a dry run of the script as well.
    }
}

public class MyLifetime : FrostingLifetime<MySettings>
{
    public override void Setup(MySettings context)
    {
        context.Magic = context.Arguments.HasArgument("magic");
    }
}

[TaskName("Provide-Another-Name-Like-This")]
public class Build : FrostingTask<MySettings>
{
    public override bool ShouldRun(MySettings context)
    {
        // Don't run this task on OSX.
        return context.Environment.Platform.Family != PlatformFamily.OSX;
    }

    public override void Run(MySettings context)
    {
        context.Information("Magic: {0}", context.Magic);
    }
}

[Dependency(typeof(Build))]
public class Default : FrostingTask
{
    // If you don't inherit from FrostinTask<MySettings>
    // the standard ICakeContext will be provided.
}
``` 

### 4. Run it!

To execute the build, simply run it like any .NET Core application.  
In the example we provide the custom `--magic` argument. Notice that
we use `--` to separate the arguments to the `dotnet` command.

```powershell
> dotnet restore
> dotnet run -- --magic
```

**NOTE:** You're not supposed to commit the produced binaries to your repository.  
The above command is what you're expected to run from your bootstrapper.

## Building from source

### .NET Core SDK

To build from source, you will need to have 
[.NET Core SDK 1.0 rc4 build 004771](https://github.com/dotnet/core/blob/master/release-notes/rc4-download.md)
installed on your machine.

### Visual Studio (optional)

If you want to develop using Visual Studio, then you need to use Visual Studio 2017 RC 4 or higher.

## Acknowledgement

The API for configuring and running the host have been heavily influenced by 
the [ASP.NET Core hosting API](https://github.com/aspnet/Hosting).

## License

Copyright © [.NET Foundation](http://dotnetfoundation.org/) and contributors.  
Frosting is provided as-is under the MIT license. For more information see 
[LICENSE](https://github.com/cake-build/frosting/blob/develop/LICENSE).

## Thanks

A big thank you has to go to [JetBrains](https://www.jetbrains.com) who provide 
each of the Cake developers with an 
[Open Source License](https://www.jetbrains.com/support/community/#section=open-source) 
for [ReSharper](https://www.jetbrains.com/resharper/) that helps with the development of Cake.

The Cake Team would also like to say thank you to the guys at
[MyGet](https://www.myget.org/) for their support in providing a Professional 
subscription which allows us to continue to push all of our pre-release 
editions of Cake NuGet packages for early consumption by the Cake community.

## Code of Conduct

This project has adopted the code of conduct defined by the 
[Contributor Covenant](http://contributor-covenant.org/) to clarify expected behavior 
in our community. For more information see the [.NET Foundation Code of Conduct](http://www.dotnetfoundation.org/code-of-conduct).

## .NET Foundation

This project is supported by the [.NET Foundation](http://www.dotnetfoundation.org).
