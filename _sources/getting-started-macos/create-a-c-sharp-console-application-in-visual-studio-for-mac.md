---
title: Create a C# Console App in Visual Studio
metadata:
    description: This article describes the steps needed to configure a C# console application in Visual Studio for Mac.
taxonomy:
    category: docs
---

# Create a C# Console App in Visual Studio for Mac

This topic describes the steps needed to configure a C# console application in Visual Studio for Mac. These steps have been verified to work with Visual Studio for Mac 2022, on macOS Ventura 13.5.2.

## Create the Visual Studio project

1. Create a C# console application in Visual Studio.
    * Choose `.NET 6.0` for Target Framework
    * Name the project `simple-converter`. 
    * Leave `Put project file in a subfolder` unchecked
    
2. Replace the contents of Program.cs with this code:

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

                inputInfo.Inputs[0].File = "AAP.m4v";
                if (inputInfo.Open())
                {
                    var inputSocket = MediaSocket.FromMediaInfo(inputInfo);
                    var outputSocket = MediaSocket.FromPreset(Preset.Video.Generic.MP4.Base_H264_AAC);
                    outputSocket.File = "AAP.mp4";

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

4. [Download](https://github.com/avblocks/avblocks-net-core/releases/) the Darwin version of AVBlocks for .NET (net60). The file you need will have a name similar to `avblocks-net60-v3.0.0-demo.1-darwin.zip` - the version number may differ. 
5. Unzip in a location of your choice, then copy the file `AVBlocks.clrcore.x64.dll` and `libAVBlocks.dylib` to the project's directory. 
6. Switch to the project in Visual Studio and add a reference to `AVBlocks.clrcore.x64.dll`. To do this right click the `simple-converter` project, select `Add | Reference | .NET Assembly | Browse` then select `AVBlocks.clrcore.x64.dll`. 
7. Add the `libAVBlocks.dylib` to your project simply as a file, open file properties, and set the 'Copy to Output Directory' to 'Copy if newer', so it is copied to the output directory automatically. The `AVBlocks.clrcore.x64.dll` needs `libAVBlocks.dylib` and will not work without it.
8. Build the project (Cmd + K)

## Run the application

1. Download the `AAP.m4v` HD movie from the [Internet Archive](https://archive.org/details/Wildlife-filming) and and save it in `bin/Debug/net6.0` under the project's directory.

2. Run the application in Visual Studio. Wait a few seconds for the Transcoder to finish. The converted file `AAP.mp4` will be in the `bin/Debug/net6.0` directory.   

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'libAVBlocks.dylib': The specified module could not be found.` To fix that, copy the file `libAVBlocks.dylib` to `bin/Debug/net6.0` under the project's directory.

* Transcoder.Open may fail if there is already a file `AAP.mp4` in the `bin/Debug/net6.0` directory. Delete `AAP.mp4` to solve that.         
