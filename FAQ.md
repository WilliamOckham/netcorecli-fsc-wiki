Some answers to frequently asked questions about developing for .NET Core. Some answers apply only to F#, others to all languages.

If something works with C#, it should work exactly the same for F#.

**TABLE OF CONTENTS**

- This is a FAQ, where are the real docs?
- .NET Core works with F#? Now? Without going crazy?
- Preview? RC?
- How can i install it?
- I maintain an F# library published as nupkg and i want to support .NET Core
    - Ok i want to start to convert a project?
    - a new target framework?
    - what dependencies version of libraries/tools?
    - how can i check version resolved?
    - some code need to be changed for .net core.. two version!
    - Can i create a single nuget package with multiple framework versions inside (.NET 4.6, .NET Core)?
        - But i already create the nuget package in my build, i dont want to use project.json for every framework
            - i need to merge nupkg manually?
- Tools support?
- Travis/Appveyor


## This is a FAQ, where are the real docs?

The [docs](https://dotnet.github.io/docs/getting-started/index.html) are really nice to read, and explain all scenarios.

So they are a good first read.

They're also open source, send a PR :smile:

## .NET Core works with F#? Now? Without going crazy?

Yes, even during the beta, it was already OK [with workarounds](https://github.com/enricosada/fsharp-dotnet-cli-samples/wiki/Changelog#v1.0.0-beta-002071)

Now, with `preview1`, .NET Core and F# work OOTB. For example, see [a suave sample app](https://github.com/SuaveIO/Suave-CoreCLR-sample), working cross platform (win/ubuntu/osx/docker).

It's not RTM yet, if you see a bug, open an issue in:

- For F# compiler errors, crash - https://github.com/Microsoft/visualfsharp 
- For .NET SDK (the cli) errors - https://github.com/dotnet/netcorecli-fsc/
- For .NET SDK if the same bug happens with similar C# code - https://github.com/dotnet/cli

## Is .NET Core in Preview? RC?

When .NET Core was in beta, the tooling (dotnet/cli, project.json, xproj) and the .NET Core (coreclr, corefx, bcl) was bundled together using same version (`beta` and after `rc2`).

The dotnet team [split the two parts](https://blogs.msdn.microsoft.com/dotnet/2016/05/16/announcing-net-core-rc2/), the `.NET Core` (more ready for production, RC quality => `RC2`) from tooling (renamed from `.NET CLI` to `.NET Core SDK`, now in `preview1`)

So:

- **.NET Core** (`RC2`) => Run apps with .NET Core runtime, clr and base class libraries, (dotnet/coreclr, dotnet/corefx)
- **.NET Core SDK** (`preview1`) => tooling. Develop apps with .NET Core and the SDK+CLI (Software Development Kit/Command Line Interface) tools, like dotnet/cli (the tool that runs `dotnet build`)

## How can i install it?

See the [docs](https://dotnet.github.io/docs/getting-started/installing/index.html)

1. From http://dot.net choose .NET Core. 
2. That page shows the quickstart. For F#, just add `--lang f#` to `dotnet new`.

Or follow the (a bit hidden) [Other downloads](https://www.microsoft.com/net/download#core) link for other os and types.

Both .NET Core and .NET SDK has two versions:

- **from os native installers: like msi/pkg**, install globally in the machine (see options)
- **from binaries: just a zip**, download => unzip => add to `PATH` => dotnet ready (multiple version can be installed in different directories). Zero fear about breaking the machine configuration.

## I maintain an F# library published as nupkg and I want to  support .NET Core

### Ok, How do I start converting a project?

There is not an updated tutorial **yet** (this [one](https://github.com/enricosada/fsharp-dotnet-cli-samples/wiki/Migrate-fsproj-To-.NET-CLI) works for beta, but procedure is the same)

The typical procedure is

1. Go to same directory of your .fsproj
2. Run `dotnet new --lang f#` in your command shell
3. If your project is a library, change the dependency from `Microsoft.NETCore.App` to `NETStandard.Library` inside `project.json` (see below)
4. If it's a library, remove the `emitEntryPoint` attribute in `project.json`
5. Fix versions of dependencies (see below) and of `compile-fsc` tool. 
6. Check to see if your project builds ( `dotnet restore` followed by `dotnet build` )
7. Add all source files, same list as `Compile` inside `.fsproj`
8. Add all additional nuget packages dependencies, like in `.fsproj`
9. Add all compiler directive in `buildOptions`.`defines` of `project.json`
8. Run `dotnet build` and fix any errors
9. Run `dotnet pack` to create your package

Check some alredy converted projects, like https://github.com/SuaveIO/suave 

### A new target framework?

See [docs](https://dotnet.github.io/docs/core-concepts/packages.html), it's explained better 

Usually, simplified (again read the [docs](https://dotnet.github.io/docs/core-concepts/packages.html))

| Type | Name | TFM | Latest version as preview1 |
|------|------|-----|----------------------------|
| library | .NET Standard | `netstandard` | `netstandard1.5` |
| console app |  .NET Core Application | `netcoreapp` | `netcoreapp1.0` |

TFM means target framework moniker, the short name ( ref [nugets targetframeworks docs](https://docs.nuget.org/create/targetframeworks) ) 

So, simplified:

- a library should target `netstandard1.5`
- a console app should target `netcoreapp1.0`

### What are the required dependencies (and their versions) for F# libraries/tools?

Usually is ok to check files generated by `dotnet new --lang f#` but `preview1` has some little mistake, so read about **Notes for F# in** [[.NET-Core-SDK-preview1]]. 

A F# library should depend (inside `dependencies`) on:

- `Microsoft.FSharp.Core.netcore`: contains the FSharp.Core assembly, no GAC in .NET Core, only nuget packages
- `NETStandard.Library` metapackage: read the docs, version depends on features
- additional packages as needed, but not `Microsoft.NETCore.App` metapackage

A F# console app should depend (inside `dependencies`) on:

- `Microsoft.FSharp.Core.netcore`: contains the FSharp.Core assembly
- `Microsoft.NETCore.App` metapackage: read the docs, version is `1.0.0-rc2-3002702` for `preview1`

As per `preview1`:

| Package | Version | Same default of dotnet new? |
|---------|---------|----------------------------|
| `Microsoft.FSharp.Core.netcore` | `1.0.0-alpha-160509` | NO, was updated :warning: |
| `NETStandard.Library` | depends but usually `1.5.0-rc2-24027` | YES, but `dotnet new` create a console app, no library! :warning:  |
| `Microsoft.NETCore.App` | `1.0.0-rc2-3002702` | YES |
| `dotnet-compile-fsc` | `1.0.0-preview1-*` | NO :warning:  |

### How can I check which version of a dependency was resolved?

do `dotnet restore` and open `project.lock.json`, search for package name.


### I need to change some of my code to run on .net core ... What do I do?

If some code need to be changed, it's possible to maintain *two versions* of the same code under '#if` defines.
When building, an upper case compiler define of TFM is passed to compiler.

so if 

```json
    "frameworks": {
        "netstandard1.5": { 
```

a compiler directive `NETSTANDARD1_5` (the `.` becomes `_`) is passed to compiler

**NOTE** the compiler is F# 4.0, so it's possibile to use `#if !DEF`, `#if A || B` and `#if A && B`

like

```fsharp
#if NETSTANDARD1_5
   let a = typeof<Program>.GetTypeInfo().Assembly.Name
#else
   let a = typeof<Program>.GetType().Assembly.Name
#endif
```


### Can I create a single nuget package with multiple framework versions inside (.NET 4.6, .NET Core)?

Yes, nuget packages support multiple versions, and the .NET Core framework is like another target framework (like PCL, or .NET 3.5).

Just add the [dependency group](https://docs.nuget.org/create/nuspec-reference) in your nuspec file and the assemblies under `lib`

#### But I already created the nuget package in my build, I don't want to use project.json for every framework

No need to go all in with `project.json` (it said it is not feeling well lately...)

You can use the .NET SDK cli tools (like `dotnet pack` or `dotnet build`) to build the assemblies (or the package)
and add these to your package

##### Do I need to merge nupkg manually?

`nuget pack` expects a nuspec and directory, so this works already.



Or you can use a opensource third party tool (https://github.com/enricosada/dotnet-mergenupkg) to merge the already built packages.

**NOTE BY AUTHOR** the `dotnet-mergenupkg` is an option written by me (@enricosada), so I like it and use it for some F# open source projects (suave/FCS/chessie). It tries to not change current build workflow, but uses dotnet cli to max. Feel free to ignore it.

1. Build your package `A` normally
2. Create a package `B` with .NET Core using `dotnet pack`
3. Use `dotnet mergenupkg` to merge assembly/nuspec from `B` to `A`, so `A` contains both.

## Tools support?

- Visual Studio: not yet
- Visual Studio Code: With the [Ionide](https://marketplace.visualstudio.com/items?itemName=Ionide.Ionide-fsharp) plugin for F#. After the first build, it should just work.
- Notepad: from day 0

## Travis/Appveyor

Some PRs are in progress to add it to .net core documentation, so it's easier to look at the PR files and discussion:

- Appveyor [PR](https://github.com/dotnet/core-docs/pull/461) and [appveyor.yml](https://github.com/enricosada/core-docs/blob/764dcf7c6bbb4f73c19c9286c4c7d2857bd99f17/appveyor.yml)
- TravisCI: [PR](https://github.com/dotnet/core-docs/pull/460) and [.travis.yml](https://github.com/enricosada/core-docs/blob/64f584f132f34ff6916b3d34cd85fa0c2cbc09e9/.travis.yml)

