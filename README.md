<div align="center">
  <h1><code>wasmtime-dotnet</code></h1>

  <p>
    <strong>.NET embedding of
    <a href="https://github.com/bytecodealliance/wasmtime">Wasmtime</a></strong>
  </p>

  <strong>A <a href="https://bytecodealliance.org/">Bytecode Alliance</a> project</strong>

  <p>
    <a href="https://github.com/bytecodealliance/wasmtime-dotnet/actions?query=workflow%3ACI">
      <img src="https://github.com/bytecodealliance/wasmtime-dotnet/workflows/CI/badge.svg" alt="CI status"/>
    </a>
    <a href="https://www.nuget.org/packages/Wasmtime">
      <img src="https://img.shields.io/nuget/v/wasmtime" alt="Latest Version"/>
    </a>
    <a href="https://bytecodealliance.github.io/wasmtime-dotnet/">
      <img src="https://img.shields.io/badge/docs-main-green" alt="Documentation"/>
    </a>
  </p>

</div>

## Installation

You can add a package reference with the [.NET SDK](https://dotnet.microsoft.com/):

```text
$ dotnet add package --version 0.25.0-preview1 wasmtime
```

_Note that the `--version` option is required because the package is currently prerelease._

## Introduction

For this introduction, we'll be using a simple WebAssembly module that imports a `hello` function and exports a `run` function:

```wat
(module
  (func $hello (import "" "hello"))
  (func (export "run") (call $hello))
)
```

To use this module from .NET, create a new console project:

```
$ mkdir wasmintro
$ cd wasmintro
$ dotnet new console
```

Next, add a reference to the [Wasmtime package](https://www.nuget.org/packages/Wasmtime):

```
$ dotnet add package --version 0.25.0-preview1 wasmtime
```

Replace the contents of `Program.cs` with the following code:

```c#
using System;
using Wasmtime;

namespace Tutorial
{
    class Program
    {
        static void Main(string[] args)
        {
            using var engine = new Engine();

            using var module = Module.FromText(
              engine,
              "hello",
              "(module (func $hello (import \"\" \"hello\")) (func (export \"run\") (call $hello)))"
            );

            using var host = new Host(engine);

            using var function = host.DefineFunction(
                "",
                "hello",
                () => Console.WriteLine("Hello from C#!")
            );

            using dynamic instance = host.Instantiate(module);
            instance.run();
        }
    }
}
```

An `Engine` is created and then a WebAssembly module is loaded from a string in WebAssembly text format.

A `Host` defines a function called `hello` that simply prints a hello message.

The module is instantiated into the host and the module's `run` export is invoked.

To run the application, simply use `dotnet`:

```
$ dotnet run
```

This should print `Hello from C#!`.

## Contributing

### Building

Use `dotnet` to build the repository:

```
$ dotnet build
```

This will download the latest development snapshot of Wasmtime for your platform.

### Testing

Use `dotnet` to run the unit tests:

```
$ dotnet test
```

### Creating the NuGet package

Use `dotnet` to create a NuGet package:

```
$ cd src
$ dotnet pack -c Release
```

This will create a `.nupkg` file in `src/bin/Release`.

By default, local builds will use a `-dev` suffix for the package to differentiate between official packages and development packages.
