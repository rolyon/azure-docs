---
title: About the Cognitive Services Speech SDK
description: An overview of the SDKs available for the Speech service.
titleSuffix: "Microsoft Cognitive Services"
services: cognitive-services
author: v-jerkin

ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/24/2018
ms.author: v-jerkin
---

# About the Cognitive Services Speech SDK

The Cognitive Services Speech Software Development Kit (SDK) provides your applications native access to the functions of the Speech service, making it easier to develop software. Currently, the SDK provides access to **Speech to Text**, **Speech Translation**, and **Intent Recognition**.

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

[!INCLUDE [License Notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

## Get the SDK

### Windows

For Windows we support the following languages:

* C# (UWP and .NET), C++:
  You can reference and use the latest version of our Speech SDK NuGet package.
  The package includes 32-bit and 64-bit client libraries as well as managed (.NET) libraries.
  The SDK can be installed in Visual Studio using NuGet; simply search for `Microsoft.CognitiveServices.Speech`.

* Java:
  You can reference and use the latest version of our Speech SDK Maven package, which supports Windows x64 only.
  In your Maven project, add `https://csspeechstorage.blob.core.windows.net/maven/` as an additional repository, and reference `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0` as a dependency. 

### Linux

> [!NOTE]
> Currently, we only support Ubuntu 16.04 on a PC (x86 or x64 for C++ development, x64 for .NET Core and Java).

Make sure you have the required compiler and libraries installed by running the following shell commands:

```sh
sudo apt-get update
sudo apt-get install build-essential libssl1.0.0 libcurl3 libasound2
```

* C#:
  You can reference and use the latest version of our Speech SDK NuGet package.
  To reference the SDK, add the following package reference into your project:

  ```xml
  <PackageReference Include="Microsoft.CognitiveServices.Speech" Version="1.0.0" />
  ```

* Java:
  You can reference and use the latest version of our Speech SDK Maven package.
  In your Maven project, add `https://csspeechstorage.blob.core.windows.net/maven/` as an additional repository, and reference `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0` as a dependency. 

* C++: download the SDK as a [.tar package](https://aka.ms/csspeech/linuxbinary) and unpack the files in into a directory of your choice. The following table shows the SDK folder structure.

  |Path|Description|
  |-|-|
  |`license.md`|License|
  |`ThirdPartyNotices.md`|Third-party notices|
  |`include`|Header files for C and C++|
  |`lib/x64`|Native x64 library for linking with your application|
  |`lib/x86`|Native x86 library for linking with your application|

  To create an application, copy or move the required binaries (and libraries) into your development environment, and include them as required into your build process.

### Android

The Java SDK for Android is packaged as an [AAR (Android Library)](https://developer.android.com/studio/projects/android-library), which includes the necessary libraries as well as required Android permissions for using it.
It is hosted in a Maven repository at `https://csspeechstorage.blob.core.windows.net/maven/` as package `com.microsoft.cognitiveservices.speech:client-sdk:1.0.0`.
To consume the package from your Android Studio project make the following changes:

* In the project-level `build.gradle` file, add the following into the `repository` section:

  ```text
  maven { url 'https://csspeechstorage.blob.core.windows.net/maven/' }
  ```

* In the module-level `build.gradle` file, add the following into the `dependencies` section:

  ```text
  implementation 'com.microsoft.cognitiveservices.speech:client-sdk:1.0.0'
  ```

The Java SDK is also part of the [Speech Devices SDK](speech-devices-sdk.md).

[!INCLUDE [Get the samples](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## Next steps

* [Get your Speech trial subscription](https://azure.microsoft.com/try/cognitive-services/)
* [See how to recognize speech in C#](quickstart-csharp-dotnet-windows.md)
