---
layout: post
title:  Build Aseprite from source on Linux
categories: Aseprite, Linux
excerpt: Aseprite is an animated sprite editor and pixel art tool. This tutorial shows how to install it on Linux OS.
---

![]({{site.baseurl}}/images/2021-09-23-build-aseprite-from-source-on-linux.png)

## Build Aseprite from source on Linux 

[Aseprite](https://www.aseprite.org/) lets you create 2D animations for video games. From sprites, to pixel-art, retro style graphics, and whatever you like about the 8-bit and 16-bit era.

*It is a paid software well worth the price and purchasable for any platform. If you like it you should definitely buy it. This method should only be used to try it out or to contribute to its development. There is also a trial version on their website.*

In this tutorial I will go over how to get the Aseprite source code, install required dependencies and compile the code to get an executable.

Official installation instructions can be found [here](https://github.com/aseprite/aseprite/blob/main/INSTALL.md).

**Versions used:** Ubuntu 20.04 LTS

### Repository

The following commands will clone Aseprite source code inside the `~/Aseprite/aseprite/` directory (`~` being your home directory, you can replace this with a path of your choice).

```
mkdir ~/Aseprite
cd ~/Aseprite
git clone --recursive https://github.com/aseprite/aseprite.git
cd aseprite
git pull
git submodule update --init --recursive
```

### Dependencies

To compile Aseprite you will need:

- The latest version of CMake (3.14 or greater)
- Ninja build system
- Pre-built package of the `aseprite-m81` branch of the Skia library

#### CMake and Ninja

Run the following commands to install required dependencies on Ubuntu:

```
sudo apt-get install -y g++ cmake ninja-build libx11-dev libxcursor-dev libxi-dev libgl1-mesa-dev libfontconfig1-dev libharfbuzz-dev
```

Alternatives for Fedora or Arch can be found [here](https://github.com/aseprite/aseprite/blob/main/INSTALL.md#linux-dependencies).

#### Skia

Use the following commands to create a directory for Skia:

```
cd ~/Aseprite
mkdir deps
cd deps
mkdir skia
```

Download required Skia release from [here](https://github.com/aseprite/skia/releases/tag/m81-b607b32047) (`Skia-Linux-Release-x64.zip` for 64 bit OS).

Unzip content and place it inside the previously created `skia` directory.

### Build

Now that we got all the dependencies it's finally time to build Aseprite, for this you will first need to create the `build` directory:

```
cd ~/Aseprite/aseprite
mkdir build
cd build
```

Run the following command:

```
cmake \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DLAF_BACKEND=skia \
  -DSKIA_DIR=~/Aseprite/deps/skia \
  -DSKIA_LIBRARY_DIR=~/Aseprite/deps/skia/out/Release-x64 \
  -DSKIA_LIBRARY=~/Aseprite/deps/skia/out/Release-x64/libskia.a \
  -G Ninja \
  ..
```

And finally run:

```
ninja aseprite
```

### Run

Now you can run aseprite with the following command:

```
~/Aseprite/aseprite/build/bin/aseprite
```

### Application

If you want to make an application for Aseprite, instead of running it with a command, you will need to create a `.desktop` file for it:

```
sudo nano /usr/share/applications/aseprite.desktop
```

Enter the following content:

```
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Type=Application
Name=Aseprite
Icon=~/Aseprite/aseprite/data/icons/ase256.png
Path=~/Aseprite/aseprite/
Exec=~/Aseprite/aseprite/build/bin/aseprite
StartupNotify=false
OnlyShowIn=GNOME;Unity;
```

Save, exit and enjoy drawing.
