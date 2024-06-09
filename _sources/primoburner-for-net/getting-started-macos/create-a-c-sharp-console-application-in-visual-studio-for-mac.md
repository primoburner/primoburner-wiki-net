---
title: Create a C# Console App in Visual Studio
html_meta:
    description: This article describes the steps needed to configure a C# console application in Visual Studio for Mac.
taxonomy:
    category: docs
---

# Create a C# Console App in Visual Studio for Mac

This topic describes the steps needed to configure a C# console application in Visual Studio for Mac. These steps have been verified to work with Visual Studio for Mac 2022, on macOS Ventura 13.5.2.

## Create the Visual Studio project

1. Create a C# console application in Visual Studio.
    * Choose `.NET 6.0` for Target Framework
    * Name the project `enum-devices`. 
    * Leave `Put project file in a subfolder` unchecked
    
2. Replace the contents of Program.cs with this code:

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

4. [Download](https://github.com/primoburner/primoburner-net-core/releases/) the Darwin version of PrimoBurner for .NET (net60). The file you need will have a name similar to `primoburner-net60-v5.0.1-demo.1-darwin.zip` - the version number may differ. 
5. Unzip in a location of your choice, then copy the file `PrimoBurner.clrcore.x64.dll` and `libPrimoBurner.dylib` to the project's directory. 
6. Switch to the project in Visual Studio and add a reference to `PrimoBurner.clrcore.x64.dll`. To do this right click the `enum-devices` project, select `Add | Reference | .NET Assembly | Browse` then select `PrimoBurner.clrcore.x64.dll`. 
7. Add the `libPrimoBurner.dylib` to your project simply as a file, open file properties, and set the 'Copy to Output Directory' to 'Copy if newer', so it is copied to the output directory automatically. The `PrimoBurner.clrcore.x64.dll` needs `libPrimoBurner.dylib` and will not work without it.
8. Build the project (Cmd + K)

## Run the application

```bash
./bin/Debug/net6.0/enum-devices
```

## Troubleshooting

* You may get a `'DllNotFoundException' exception: Unable to load DLL 'libPrimoBurner.dylib': The specified module could not be found.` To fix that, copy the file `libPrimoBurner.dylib` to `bin/Debug/net6.0` under the project's directory.
