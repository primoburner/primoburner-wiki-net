---
title: Pad Video
metadata:
    description: This article explains how you can fit a 16:9 video into a 4:3 frame by sizing and padding the original picture.
taxonomy:
    category: docs
---

# Pad Video

This article explains how you can fit a 16:9 video into a 4:3 frame by sizing and padding the original picture.

## Source Video

For a source video we use the MP4 file from the TED talk video [What's the next window into our universe?](https://archive.org/details/AndrewConnolly_2014) by Andrew Connolly. The original video format is Wide 480p or 16:9, 854 x 480.

## Code

This code takes a 16:9 480p (854x480) video, and squeezes it into a 4:3 640x480 frame, while maintaining the original 16:9 picture aspect ratio. The audio stream is copied from the source as is.    

### Windows
 
``` cpp
#include <primo/avblocks/avb.h>
#include <primo/platform/reference++.h>

// link with AVBlocks64.lib
#pragma comment(lib, "./avblocks/lib/x64/AVBlocks64.lib")

using namespace primo::codecs;
using namespace primo::avblocks;

int main(int argc, const char * argv[]) {
    Library::initialize();

    // start with two identical input and output configurations
    ref<MediaInfo> inputInfo (Library::createMediaInfo());
    inputInfo->setInputFile(L"AndrewConnolly_2014.mp4");

    ref<MediaInfo> outputInfo (Library::createMediaInfo());
    outputInfo->setInputFile(L"AndrewConnolly_2014.mp4");

    if (inputInfo->load() && outputInfo->load()) {
        using namespace Param::Video;

        // create input socket
        ref<MediaSocket> inputSocket (Library::createMediaSocket(inputInfo.get()));

        // create output socket
        ref<MediaSocket> outputSocket (Library::createMediaSocket(outputInfo.get()));
        outputSocket->setFile(L"AndrewConnolly_2014_640x480_Padded.mp4");

        // get output video pin
        auto outVideoPin = outputSocket->pins()->at(0);

        auto outVideoStream = static_cast<VideoStreamInfo*>(outVideoPin->streamInfo()); 
        
        // set the new frame width and height to 640 x 480
        outVideoStream->setFrameWidth(640);
        outVideoStream->setFrameHeight(480);

        // set the display ratio to 4:3
        outVideoStream->setDisplayRatioWidth(4);
        outVideoStream->setDisplayRatioHeight(3);

        // set Pad.Top and Pad.Bottom
        ref<ParameterList> outVidePinsParams (Library::createParameterList()); 

        // The input video is 16:9, 854 x 480,  
        // to squeeze it in 640 x 480 and keep the 16:9 display ratio, 
        // the height has to be: 640 * 9 / 16

        // Pad by (480 - 640 * 9 / 16) / 2 pixels on each side.
        ref<IntParameter> padTop (Library::createIntParameter());
        padTop->setName(Pad::Top);
        padTop->setValue((480 - 640 * 9 / 16) / 2);

        outVidePinsParams->add(padTop.get());

        ref<IntParameter> padBotttom (Library::createIntParameter());
        padBotttom->setName(Pad::Bottom);
        padBotttom->setValue((480 - 640 * 9 / 16) / 2);

        outVidePinsParams->add(padBotttom.get());

        // The padding will trigger downscaling, 
        // so we also set the interpolation method,
        // for best result when downscaling, use InterpolationMethod::Super
        ref<IntParameter> interpolationMethod (Library::createIntParameter()); 
        interpolationMethod->setName(Resize::InterpolationMethod);
        interpolationMethod->setValue(InterpolationMethod::Super);

        outVidePinsParams->add(interpolationMethod.get());

        outVideoPin->setParams(outVidePinsParams.get());

        // create a Transcoder
        ref<Transcoder> transcoder (Library::createTranscoder());

        // add input and output sockets
        transcoder->inputs()->add(inputSocket.get());
        transcoder->outputs()->add(outputSocket.get());

        // process
        if (transcoder->open()) {
            transcoder->run();
            transcoder->close();
        }
    }

    Library::shutdown();

    return 0;
}
```

#### How to run

Follow the [steps](../getting-started-windows/create-a-c-plus-console-app-in-visual-studio) to create a C++ console application in Visual Studio, but use the code from this article. 

Download the `AndrewConnolly_2014.mp4` MPEG4 file from the [Internet Archive](https://archive.org/details/AndrewConnolly_2014) and save it in the project directory.

Run the application in Visual Studio. Wait a few seconds for the Transcoder to finish. The converted file `AndrewConnolly_2014_640x480_Padded.mp4` will be in the project directory.

