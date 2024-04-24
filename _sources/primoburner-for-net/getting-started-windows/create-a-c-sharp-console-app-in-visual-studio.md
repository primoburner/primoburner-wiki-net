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
    * Name the project `enum-devices`. 
    * Select `Place solution and project in the same directory`
    * Choose `.NET 6.0 (Long-term support)` for Framework
2. In Visual Studio, select `Build | Configuration Manager` from the menu, then add the `x64` platform to the solution platforms.
3. Replace the contents of Program.cs with this code:

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

4. [Download](https://github.com/primoburner/primoburner-net-core/releases/) the Windows version of PrimoBurner for .NET (net60). The file you need will have a name similar to `primoburner-net60-v5.0.1-demo.1-windows.zip` - the version number may differ. 
5. Unzip in a location of your choice, then copy the files: `PrimoBurner.clrcore.x64.dll` and `PrimoBurner64.dll`, to the project's directory. 
6. Switch to the project in Visual Studio and add a reference to `PrimoBurner.clrcore.x64.dll`. 
7. Add the `PrimoBurner64.dll` to your project simply as a file, open file properties, and set the 'Copy to Output Directory' to 'Copy if newer', so it is copied to the output directory automatically. The `PrimoBurner.clrcore.x64.dll` needs `PrimoBurner64.dll` and will not work without it.
8. Build the project (Ctrl + Shift + B)

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'PrimoBurner64.dll': The specified module could not be found.` To fix that, copy the file `PrimoBurner64.dll` to `bin/x64/Debug/net6.0` under the project's directory.
