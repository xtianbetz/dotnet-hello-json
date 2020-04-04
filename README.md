# dotnet CLI example with JSON

## Overview

This repo is a simple demo application built with dotnet that processes JSON. It
was made to help me learn, understand, and teach using dotnet build tools on
Linux without using any proprietary visual studio tools.

The instructions below were tested on a CentOS 7.7 build host and a Raspberry
Pi 4 running Yocto version 3.0.

## Building and Running In Various Ways

### Local Builds using the dotnet CLI

The following commands can build/run the application:

```
dotnet build
dotnet run
dotnet build -c Release
dotnet run -c Release
```

### ARM Builds using the dotnet CLI

Publish a build for the "linux-arm" runtime. This will produce something that
appears to be a native ARM binary.

```
dotnet publish -c Release -r linux-arm
```

Disable Globalization (optional, if needed)

```
cat bin/Release/netcoreapp3.1/linux-arm/publish/dotnet-hello-json.runtimeconfig.json | jq -r '.runtimeOptions |= . + {"configProperties":{"System.Globalization.Invariant":true}}' > dotnet-hello-json.runtimeconfig.json
mv dotnet-hello-json.runtimeconfig.json bin/Release/netcoreapp3.1/linux-arm/publish/dotnet-hello-json.runtimeconfig.json
```

Run the build on a raspberry pi:
```
tar -C bin/Release/netcoreapp3.1/linux-arm/publish/ -cvzf /tmp/dotnet-hello-json.tgz .
scp /tmp/dotnet-hello-json.tgz pi:/tmp
ssh pi "mkdir -p /usr/libexec/dotnet-hello-json &&
tar -C /usr/libexec/dotnet-hello-json -xzvf /tmp/dotnet-hello-json.tgz &&
ln -sf /usr/libexec/dotnet-hello-json/dotnet-hello-json /usr/bin/dotnet-hello-json &&
dotnet-hello-json"
```

### Cross-platform builds using msbuild

You can use msbuild and target the 'net48' framework and you will get a single
directory with a familiar .exe file. You will have to run this using "mono"
using the recommended shell-script wrapper technique.

```
msbuild -restore -p:TargetFramework=net48
msbuild -restore -p:Configuration=Release -p:TargetFramework=net48
```

You can now run the release build using mono:

```
tar -C bin/Release/net48/ -cvzf /tmp/clr-dotnet-hello-json.tgz .
scp /tmp/clr-dotnet-hello-json.tgz pi:/tmp
ssh pi mkdir -p /usr/libexec/clr-dotnet-hello-json
ssh pi tar -C /usr/libexec/clr-dotnet-hello-json -xzvf /tmp/clr-dotnet-hello-json.tgz
ssh pi 'echo -en "#!/bin/sh\n/usr/bin/mono /usr/libexec/clr-dotnet-hello-json/dotnet-hello-json.exe \"\$@\"\n" > /usr/bin/clr-dotnet-hello-json'
ssh pi chmod +x /usr/bin/clr-dotnet-hello-json
ssh pi clr-dotnet-hello-json
```

TODO: this repo has yet been tested with any native libraries which may require
multiple native .so files for different architectures. PRs are welcome.

## How this repo was made

```
mkdir dotnet-hello-json
cd dotnet-hello-json/
dotnet new console
dotnet add package Newtonsoft.Json
```

Convert to Unix line endings:

```
unix2dos dotnet-hello-json.csproj
```

Finally, I edited Program.cs to create a mashup of HelloWorld and the
Newtonsoft.JSON object deserialization example you can find
[here](https://www.newtonsoft.com/json/help/html/DeserializeObject.htm)

## Resources
* [Official .NET guide](https://docs.microsoft.com/en-us/dotnet/standard/)
  Required reading. Just do it!
* All about Target frameworks [Target
  Frameworks](https://docs.microsoft.com/en-us/nuget/reference/target-frameworks)
* SDK-style project definitions [Moving to SDK-Style projects and package
  references in Visual Studio, part
  2](http://hermit.no/moving-to-sdk-style-projects-and-package-references-in-visual-studio-part-2/)
* More about csproj changes for .NET Core [Additions to the csproj format for
  .NET Core](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj)
* [Package references (PackageReference) in project
  files](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files)
* Installing the dotnet tool on CentOS 7: [CentOS 7 Package Manager - Install
  .NET
  Core](https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-centos7)
* [Quickstart: Install and use a package using the dotnet
  CLI](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-using-the-dotnet-cli)
* [Couldn’t find a valid ICU package installed on the
  system?](https://github.com/dotnet/core/issues/2186)
* Deserialize an Object with Newtonsoft.JSON: [Deserialize an
  Object](https://www.newtonsoft.com/json/help/html/DeserializeObject.htm)
* [Running Mono
  Applications](https://www.mono-project.com/archived/guiderunning_mono_applications/)


