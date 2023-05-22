# Compiling FFmpeg with VMAF on Windows

This document records how I compiled FFmpeg with VMAF on Windows 11 using MSYS2.

## Prerequisite: MSYS2

**MSYS2** is a collection of tools and libraries providing you with an easy-to-use environment for building, installing and running native Windows software.

To install it, go to its [official website](https://www.msys2.org/) and follow the Installation section. After downloading, several environments appear in the Start menu, such as UCRT64,  MINGW64 and CLANG64. MSYS2 comes with different environments/subsystems and the first thing you have to decide is which one to use. The differences among the environments are mainly environment variables, default compilers/linkers, architecture, system libraries used etc.

**NOTE**: if you have had git-bash on windows, uninstall it (you may need to uninstall Git). You can install it in MSYS later.

Now, UCRT64 is officially recommended. But MINGW64 has been widely used, so I use it for install compile FFmpeg. There is also an application **MSYS2 SYS** in the Start menu. This is also a terminal, but cannot be used to build programs (if you do not believe, try it yourself in the compiling step). However, it can be open in Windows terminal, which you may configure as you like. So, you may do other things in MSYS2 SYS but compile programs in MINGW64.

Now, you have an Linux-like environment on Windows. It includes a package manager `pacman`. Update it by

```bash
pacman -Syu
```

To install `gcc` in it, run

```bash
pacman -S mingw-w64-x86_64-toolchain
pacman -S base-devel    
pacman -S yasm nasm
```

Now you can call `gcc` to build software for Windows

```bash
$ gcc --version
gcc.exe (Rev6, Built by MSYS2 project) 13.1.0
Copyright (C) 2023 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

You also need some packages for install VMAF.

```bash
pacman -S --noconfirm --needed mingw-w64-x86_64-nasm mingw-w64-x86_64-gcc mingw-w64-x86_64-meson mingw-w64-x86_64-ninja
```

## Downloading Files

You need to install of VMAF, FFmpeg. Many guides suggest installing `x264` as well. You are recommended to keep all files in one directory (in the following tutorial, I use `/d/ffmpeg`, which is referred as `<project root directory>`).

### FFmpeg

Download the *source code* of FFmpeg from the [official website](https://ffmpeg.org/download.html). Then, unzip it in the `<project root directory>`. An absolute path is recommended. Possibly, you will get a folder named `ffmpeg-x.x`. Now, make a directory in  `<project root directory>` for compiling result

```bash
mkdir ffmpeg-install
```

### VMAF

It is assumed the final results will be in `<project root directory>/vmaf-install`. So, in  `<project root directory>`

```bash
git clone https://github.com/Netflix/vmaf.git
mkdir vmaf-install
```

### x264

Similarly, we set the final results will be in `<project root directory>/x264-install`.

```bash
git clone https://code.videolan.org/videolan/x264.git
mkdir x264-install
```

Now, your  `<project root directory>` has 6 folders:

```bash
$ ls
ffmpeg-x.x  ffmpeg-install  vmaf  vmaf-install  x264  x264-install
```

## Compile and Build Projects

### x264

We build x264 first. The following script is used for install x264. Save it as  `<project root directory>/build-x264.sh`.

```sh
#!/bin/sh
basepath=$(cd `dirname $0`;pwd)
echo ${basepath}

cd ${basepath}/x264
pwd

./configure --prefix=${basepath}/x264-install --enable-shared
make -j8
make install
```

Run it in  `<project root directory>`:

```bash
./build-x264.sh
```

### VMAF

In  `<project root directory>`, setup the meson and build:

```bash
cd vmaf
meson setup libvmaf libvmaf/build --buildtype release --default-library static --prefix <project root directory>/vmaf-install
meson install -C libvmaf/build
```

