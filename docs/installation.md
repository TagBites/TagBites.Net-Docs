# Installation

## NuGet

```
dotnet add package TagBites.Net
```

or via Package Manager Console:

```
Install-Package TagBites.Net
```

NuGet Package: https://www.nuget.org/packages/TagBites.Net/

## Requirements

- Targets **`netstandard2.0`** and **`net7.0`** (multi-targeted package — your project can be older/newer as long as it's compatible with `netstandard2.0`, e.g. .NET Framework 4.6.1+, .NET Core 2.0+, or any modern .NET version).
- [Json.NET (Newtonsoft.Json)](https://www.newtonsoft.com/json) 13.0.1+ — used internally as the default serializer (see [Configuration](configuration.md)). Pulled in automatically as a NuGet dependency.
- `System.Reflection.DispatchProxy` 4.3.0+ — only required (and only pulled in) on `netstandard2.0`; not needed on `net7.0`, which has it built in. This powers the RMI controller proxies (see [RMI advanced](rmi-advanced.md)).

Always double-check the *Dependencies*/*Frameworks* tabs on the [NuGet page](https://www.nuget.org/packages/TagBites.Net/) for the specific version you install, in case this changes in a future release.

## Basic usage

Once installed, reference the library and create a `Client` or `Server` instance — see [Quick start](README.md) for a complete example.

```csharp
using TagBites.Net;
```
