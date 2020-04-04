# dotnet CLI example with JSON

## Overview

This repo is a simple demo application built with dotnet that processes JSON
using Newtonsoft.JSON obtained from Nuget. It was made to help me learn,
understand, and teach using dotnet build tools on Linux without using any
visual studio tools.

## Building and Running In Various Ways

### Local Builds

The following commands can build/run the application:

```
dotnet build
dotnet run
dotnet build -c Release
dotnet run -c Release
```

### ARM Builds

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

## How this repo was made

```
sudo yum install dotnet-sdk-3.1 unix2dos
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

- Installing the dotnet tool on CentOS 7: [CentOS 7 Package Manager - Install .NET Core](https://docs.microsoft.com/en-us/dotnet/core/install/linux-package-manager-centos7)
- Deserialize an Object with Newtonsoft.JSON: [Deserialize an
  Object](https://www.newtonsoft.com/json/help/html/DeserializeObject.htm)
