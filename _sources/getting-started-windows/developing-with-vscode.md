---
title: Developing with Visual Studio Code
metadata:
    description: This page describes the steps needed to setup Visual Studio Code for AVBlocks .NET development on macOS
taxonomy:
    category: docs
---

# Developing with Visual Studio Code

This topic describes the steps needed to setup Visual Studio Code for AVBlocks .NET development on macOS. These steps have been verified to work on macOS Ventura 13.5.2.

## .NET

If you don't have .NET installed follow the steps in the [Setup .NET development environment on Windows](https://blog.primosoftware.com/setup-net-development-environment-windows/) post to configure a .NET development environment. 

## Visual Studio Code

Download and install from [Visual Studio Code](https://code.visualstudio.com/download) site.

Open Visual Studio Code and press `Cmd + Shift + p`. Select `Shell Command: Install 'code' command in PATH`. 

Also install the [C# Dev Kit extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit).

Close Visual Studio Code.

## Create the .NET project 

```powershell
mkdir ~/avblocks/net/simple-converter
cd ~/avblocks/net/simple-converter

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
dotnet sln add simple-converter.csproj

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
            "program": "${workspaceFolder}/bin/Debug/net6.0/simple-converter.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "console": "internalConsole",
            "stopAtEntry": false
        }
    ]
}
```

### Test the build

Test the build by pressing `Cmd + B`. Visual Studio Code should execute the `dotnet build` command automatically.

### Test the debugging

Set a breakpoint on the first line of `static void Main(string[] args)` inside `Program.cs`. Press `F5` to launch the debugger. It should stop at the breakpoint.

## Update for AVBlocks .NET

1. Replace the contents of `Program.cs` with this code:

    ```csharp
    using PrimoSoftware.AVBlocks;

    namespace SimpleConverter
    {
        class Program
        {
            static void Main(string[] args)
            {
                Library.Initialize();

                var inputInfo = new MediaInfo();

                inputInfo.Inputs[0].File = "Wildlife.wmv";
                if (inputInfo.Open())
                {
                    var inputSocket = MediaSocket.FromMediaInfo(inputInfo);
                    var outputSocket = MediaSocket.FromPreset(Preset.Video.Generic.MP4.Base_H264_AAC);
                    outputSocket.File = "Wildlife.mp4";

                    using (var transcoder = new Transcoder())
                    {
                        transcoder.Inputs.Add(inputSocket);
                        transcoder.Outputs.Add(outputSocket);

                        if (transcoder.Open())
                        {
                            transcoder.Run();
                            transcoder.Close();
                        }
                    }
                }

                Library.Shutdown();
            }
        }
    }
    ```

2. [Download](https://github.com/avblocks/avblocks-net-core/releases/) the Windows version of AVBlocks for .NET (net60). The file you need will have a name similar to `avblocks-net60-v3.0.0-demo.1-windows.zip` - the version number may differ. 

3. Unzip in a location of your choice, then copy the file `AVBlocks.clrcore.x64.dll` and `AVBlocks64.dll` to the project's directory.

4. Add a reference to `AVBlocks.clrcore.x64.dll` 

    Replace the contents of the `simple-converter.csproj` project file with this code:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <RootNamespace>SimpleConverter</RootNamespace>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>

    <!-- This adds a reference to AVBlocks .NET dll-->
    <ItemGroup>
        <Reference Include="AVBlocks.clrcore.x64">
        <HintPath>AVBlocks.clrcore.x64.dll</HintPath>
        </Reference>  
    </ItemGroup>
    
    </Project>
    ```

## Run the application

1. Download the `Wildlife.wmv` HD movie from the [Internet Archive](https://archive.org/download/WildlifeHd/Wildlife.wmv) and save it in the project directory.

2. Run the application in Visual Studio Code. Wait a few seconds for the Transcoder to finish. The converted file `Wildlife.mp4` will be in the project directory.   

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'AVBlocks64.dll': The specified module could not be found.` To fix that, copy the file `AVBlocks64.dll` to the project directory.

* `Transcoder.Open` may fail if there is already a file `Wildlife.mp4` in the project directory. Delete `Wildlife.mp4` to solve that.         
