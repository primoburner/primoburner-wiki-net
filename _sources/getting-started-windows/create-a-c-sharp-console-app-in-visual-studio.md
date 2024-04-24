---
title: Create a C# Console App in Visual Studio
metadata:
    description: This article describes the steps needed to configure a C# console application in Visual Studio.
taxonomy:
    category: docs
---

# Create a C# Console App in Visual Studio

This topic describes the steps needed to configure a C# console application in Visual Studio. These steps have been verified to work with Visual Studio 2022 Community Edition, on Windows 11 Pro (22H2).

## Create the Visual Studio project

1. Create a C# console application in Visual Studio.
    * Name the project `simple-converter`. 
    * Select `Place solution and project in the same directory`
    * Choose `.NET 6.0 (Long-term support)` for Framework
2. In Visual Studio, select `Build | Configuration Manager` from the menu, then add the `x64` platform to the solution platforms.
3. Replace the contents of Program.cs with this code:

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

4. [Download](https://github.com/avblocks/avblocks-net-core/releases/) the Windows version of AVBlocks for .NET (net60). The file you need will have a name similar to `avblocks-net60-v3.0.0-demo.1-windows.zip` - the version number may differ. 
5. Unzip in a location of your choice, then copy the file `AVBlocks.clrcore.x64.dll` and `AVBlocks64.dll` to the project's directory. 
6. Switch to the project in Visual Studio and add a reference to `AVBlocks.clrcore.x64.dll`. 
7. Add the `AVBlocks64.dll` to your project simply as a file, open file properties, and set the 'Copy to Output Directory' to 'Copy if newer', so it is copied to the output directory automatically. The `AVBlocks.clrcore.x64.dll` needs `AVBlocks64.dll` and will not work without it.
8. Build the project (Ctrl + Shift + B)

## Run the application

1. Download the `Wildlife.wmv` HD movie from the [Internet Archive](https://archive.org/download/WildlifeHd/Wildlife.wmv) and save it in `bin/x64/Debug/net6.0` under the project's directory.
2. Run the application in Visual Studio. Wait a few seconds for the Transcoder to finish. The converted file `Wildlife.mp4` will be in the `bin/x64/Debug/net6.0` directory.   

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'AVBlocks64.dll': The specified module could not be found.` To fix that, copy the file `AVBlocks64.dll` to `bin/x64/Debug/net6.0` under the project's directory.

* Transcoder.Open may fail if there is already a file `Wildlife.mp4` in the `bin/x64/Debug/net6.0` directory. Delete `Wildlife.mp4` to solve that.         
