---
title: Hardware Acceleration
metadata:
    description: This topic explains how to enable Intel Graphics, AMD, or NVIDIA hardware acceleration.
taxonomy:
    category: docs
---

# Hardware Acceleration

How to enable Intel Graphics, AMD, or NVIDIA hardware acceleration.

For detailed information about the supported hardware platforms, see [About Hardware Acceleration](/about-avblocks/about-hardware-acceleration/).

## Example

To enable hardware encoding, on the output {mediapin-cpp}` `, set the `HardwareEncoder` parameter to `HardwareEncoder::Auto`. This must be done before calling `Transcoder::open`:

``` cpp

// create output socket
auto outputSocket = primo::make_ref(
    Library::createMediaSocket(Preset::Video::iPad::H264_720p)
);

// enable hardware acceleration
auto outVideoPin = outputSocket->pins()->at(0);
outVideoPin->params()->addInt(Param::HardwareEncoder, HardwareEncoder::Auto);

```

## Complete C++ Code

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

    auto inputInfo = primo::make_ref(
        Library::createMediaInfo()
    );
    inputInfo->setInputFile(L"Wildlife.wmv");

    if (inputInfo->load()) {
        
        // create input socket
        auto inputSocket = primo::make_ref(
            Library::createMediaSocket(inputInfo.get())
        );
            
        // create output socket
        auto outputSocket = primo::make_ref(
            Library::createMediaSocket(Preset::Video::Generic::MP4::Base_H264_AAC)
        );

        // enable hardware acceleration
        auto outVideoPin = outputSocket->pins()->at(0);
        outVideoPin->params()->addInt(Param::HardwareEncoder, HardwareEncoder::Auto);

        outputSocket->setFile(L"Wildlife.mp4");

        // configure Transcoder inputs and outputs 
        auto transcoder = primo::make_ref(
            Library::createTranscoder()
        );
        
        transcoder->inputs()->add(inputSocket.get());
        transcoder->outputs()->add(outputSocket.get());

        // run Transcoder
        if (transcoder->open()) {
            transcoder->run();
            transcoder->close();
        }
    }

    Library::shutdown();

    CoUninitialize();

    return 0;
}
```

