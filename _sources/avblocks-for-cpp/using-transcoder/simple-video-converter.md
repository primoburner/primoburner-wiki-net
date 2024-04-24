---
title: Simple Video Converter
metadata:
    description: This topic is about building a video converter using C++ and Visual Studio.
taxonomy:
    category: docs
---

# Simple Video Converter

Building a video converter using C++ and Visual Studio.

## Source Video

For a video source we use the `Wildlife.wmv` HD movie from the [Internet Archive](https://archive.org/download/WildlifeHd/Wildlife.wmv). The original video format is WMV 720p, 16:9, 1280 x 720.

## Code

Just under 50 lines of code, this snippet is a fully functional video converter. It will take any input supported by AVBlocks and will convert it to an iPad HD 720p video.

### Windows

``` cpp
#include <primo/avblocks/avb.h>
#include <primo/platform/reference++.h>

// link with AVBlocks64.lib
#pragma comment(lib, "./avblocks/lib/x64/AVBlocks64.lib")

using namespace primo;
using namespace primo::codecs;
using namespace primo::avblocks;

int main(int argc, const char * argv[]) {
    // needed for Windows Media Codecs
    CoInitializeEx(nullptr, COINITBASE_MULTITHREADED);

    Library::initialize();

    ref<MediaInfo> inputInfo(Library::createMediaInfo());
    inputInfo->inputs()->at(0)->setFile(L"Wildlife.wmv");

    if (inputInfo->open()) {
        ref<MediaSocket> inputSocket(Library::createMediaSocket(inputInfo.get()));
        ref<MediaSocket> outputSocket(Library::createMediaSocket(Preset::Video::Generic::MP4::Base_H264_AAC));
        outputSocket->setFile(L"Wildlife.mp4");

        ref<Transcoder> transcoder(Library::createTranscoder());
        transcoder->inputs()->add(inputSocket.get());
        transcoder->outputs()->add(outputSocket.get());

        if (transcoder->open()) {
            transcoder->run();
            transcoder->close();
        }

        inputInfo->close();
    }

    Library::shutdown();

    CoUninitialize();

    return 0;
}
```

#### How to run   

Follow the [steps](../getting-started-windows/create-a-c-plus-console-app-in-visual-studio) to create a C++ console application in Visual Studio, but use the code from this article. 

Download the `Wildlife.wmv` HD movie from the [Internet Archive](https://archive.org/download/WildlifeHd/Wildlife.wmv) and save it in the project directory.

Run the application in Visual Studio. Wait a few seconds for the Transcoder to finish. The converted file `Wildlife.mp4` will be in the project directory.

         