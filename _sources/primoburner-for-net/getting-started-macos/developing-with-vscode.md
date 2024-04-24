---
title: Developing with Visual Studio Code
metadata:
    description: This page describes the steps needed to setup Visual Studio Code for PrimoBurner .NET development on macOS
taxonomy:
    category: docs
---

# Developing with Visual Studio Code

This topic describes the steps needed to setup Visual Studio Code for PrimoBurner .NET development on macOS. These steps have been verified to work on macOS Ventura 13.5.2.

## .NET

If you don't have .NET installed follow the steps in the [Setup .NET development environment on macOS](https://blog.primosoftware.com/setup-net-development-environment-macos/) post to configure a .NET development environment. 

## Visual Studio Code

Download and install from [Visual Studio Code](https://code.visualstudio.com/download) site.

Open Visual Studio Code and press `Cmd + Shift + P`. Select `Shell Command: Install 'code' command in PATH`. 

Also install the [C# Dev Kit extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit).

Close Visual Studio Code.

## Create the .NET project 

```bash
mkdir -p ~/primoburner/net/enum-devices
cd ~/primoburner/net/enum-devices

# needed so that we can run the dotnet CLI
export PATH="$HOME/.dotnet:$PATH"

# We use 8.0 here but give the commands for other versions
# see: https://learn.microsoft.com/en-us/dotnet/core/tools/global-json 
dotnet new globaljson --sdk-version 8.0.100 --roll-forward latestPatch
# dotnet new globaljson --sdk-version 7.0.404 --roll-forward latestPatch
# dotnet new globaljson --sdk-version 6.0.417 --roll-forward latestPatch

# create new console application and project
dotnet new console --framework net6.0 --use-program-main 

# create new solution and add the project to it
dotnet new sln
dotnet sln add enum-devices.csproj

# add .gitignore
dotnet new gitignore
```

### Automate the build

Add the following Visual Studio Code specific files to the `.vscode` subdir:

#### `.vscode/tasks.json`

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "dotnet",
            "task": "build",
            "group": "build",
            "problemMatcher": [],
            "label": "dotnet: build"
        }
    ]
}
```

### Setup Debugging

Add the following Visual Studio Code specific files to the `.vscode` subdir:

#### `.vscode/launch.json`

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": ".NET Core Launch (console)",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "dotnet: build",
            "program": "${workspaceFolder}/bin/Debug/net6.0/enum-devices.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "console": "internalConsole",
            "stopAtEntry": false
        }
    ]
}
```

### Test the build

Test the build by pressing `Cmd + Shift + B`. Visual Studio Code should execute the `dotnet build` command automatically.

### Test the debugging

Set a breakpoint on the first line of `static void Main(string[] args)` inside `Program.cs`. Press `F5` to launch the debugger. It should stop at the breakpoint.

## Update for PrimoBurner .NET

1. Replace the contents of `Program.cs` with this code:

    ```csharp
    using PrimoSoftware.Burner;

    namespace EnumDevices
    {
        class Program
        {
            static void Main(string[] args)
            {
                // Initialize PrimoBurner
                Library.Initialize();

                // Set license. To run PrimoBurner in demo mode, comment the next line out
                Library.SetLicense("license-xml-string");

                // Create engine
                using (var engine = new Engine())
                {
                    // Initialize engine
                    engine.Initialize();

                    // create device enumerator
                    using (var enumerator = engine.CreateDeviceEnumerator())
                    {
                        for (int i = 0; i < enumerator.Count; i++)
                        {
                            // create a device; do not ask for exclusive access
                            var device = enumerator.CreateDevice(i, false);
                            if (null != device)
                            {
                                Console.WriteLine("({0}:) - {1}",
                                    device.DriveLetter, device.Description);
                            }
                        }
                    }

                    // terminate engine
                    engine.Shutdown();
                }

                Library.Shutdown();
            }
        }
    }
    ```

2. [Download](https://github.com/primoburner/primoburner-net-core/releases/) the Darwin version of PrimoBurner for .NET (net60). The file you need will have a name similar to `primoburner-net60-v5.0.1-demo.1-darwin.zip` - the version number may differ. 

3. Unzip in a location of your choice, then copy the file `PrimoBurner.clrcore.x64.dll` and `libPrimoBurner.dylib` to the project's directory. 

4. Add a reference to `PrimoBurner.clrcore.x64.dll` 

    Replace the contents of the `enum-devices.csproj` project file with this code:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <RootNamespace>EnumDevices</RootNamespace>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>

    <!-- This adds a reference to PrimoBurner .NET dll-->
    <ItemGroup>
        <Reference Include="PrimoBurner.clrcore.x64">
        <HintPath>PrimoBurner.clrcore.x64.dll</HintPath>
        </Reference>  
    </ItemGroup>
    
    </Project>
    ```

## Run the application

In Terminal:

```bash
 ./bin/Debug/net6.0/enum-devices
```

or in Visual Studio Code, press `F5` to debug the application.   

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'libPrimoBurner.dylib': The specified module could not be found.` To fix that, copy the file `libPrimoBurner.dylib` to the project directory.
