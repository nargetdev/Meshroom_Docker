# Dockerfile for Building and Running Meshroom
This repository contains a Dockerfile for building and running [Meshroom](https://github.com/alicevision/meshroom/) and its dependencies in a Docker container. Meshroom is a program that can be used for 3D reconstructions. The program itself is really just the graphical interface for the reconstruction pipeline which is implemented by the program(s) called [AliceVision](https://github.com/alicevision/AliceVision). The version built here currently uses AliceVision's git commit 62ab2b5 from 29. August 2024. It supports pretty much all features that Meshroom can currently offer, GPU, CCTags, Lidar, etc ... It also includes some adjustments I've made, which are partly described in this video:

[![Watch the video](https://img.youtube.com/vi/XUKu1apUuVE/hqdefault.jpg)](https://www.youtube.com/embed/XUKu1apUuVE)

## Why Docker?
I spent about a week (literally) trying to compile Meshroom/AliceVision on my Arch Linux system. My main issue was that Arch uses very up-to-date versions of most packages, some of which were too new for compiling AliceVision. For example, to build AliceVision with GPU support you need to use nvcc to compile CUDA programs. The latest NVIDIA CUDA Toolkit (as of when I created this Dockerfile) is version 12.6. This nvcc version requires the gcc version to be at most 13.2 (see [here](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html) for details). On my Arch Linux system I had gcc version 14.2.1. You can install 13.2.1 from the repositories but even that didn't work (CCTag didn't compile with it, some symbol that was there in 13.2 wasn't there anymore in 13.2.1). Even after compiling a suitable gcc version manually -- which takes forever -- other errors kept popping up. Eventually I got Meshroom/AliceVision to compile without errors but as soon as I started the program it crashed with lots of runtime errors.

Using Docker allows us to pick a base operating system that's old enough to support the requirements of the current Meshroom/AliveVision implementation. The downside is that it requires more space because quite a lot of packages that you already have on your host system will also be installed in the container. I have not noticed any performance degradation (compared to running pre-built binaries of older Meshroom versions).

## System (Requirements)
I've built this Dockerfile with my setup in mind. If your setup is different then you may have to do some adjustments:

#### Display Server
I'm on Arch Linux with Wayland. If you use X-Org instead of Wayland then you probably have to change the Dockerfile as described [here](https://wiki.archlinux.org/title/Docker#Run_graphical_programs_inside_a_container) (although I haven't tried that).

#### Rootless Docker
I'm also using *rootless* Docker (see [here](https://wiki.archlinux.org/title/Docker#Rootless_Docker_daemon)) so the root user inside the container gets mapped to my user on the host. If instead you use the "standard" version of Docker (with root rights) then all files that Meshroom creates in the container will be owned by root, even if saved on the host system.

#### CUDA Version
The Docker image uses CUDA Toolkit version 12.6. According to [NVIDIA](https://docs.nvidia.com/deploy/cuda-compatibility/index.html) this version supports all CUDA drivers newer than (and including) 525.60.13. If you've got an older version of the CUDA driver then consider upgrading or change the Dockerfile to use an older version of the Toolkit (although I haven't tested that).

## Limitations
Since Meshroom runs in a container it can't access programs on your host system. So if, for example, you press the button to open the MeshroomCache folder (on the bottom left of the screen) with your filemanager then nothing will happen. You can still open the folder on your host system though.

## Which Adjustments Have I Made?
I've applied a few patches to Meshroom/AliceVision:
- CCTags doesn't seem to build correctly on newer OSs (at least not on my two computers). I've added a patch to implement the workaround I described [here](https://github.com/alicevision/CCTag/issues/219). The patch causes AliceVision to clone my fork of CCTags, which includes the workaround.
- I've changed one of AliceVision's CMake files to use parallel building for both, building dependencies and building AliceVision (see the comments in the Dockerfile for more details).
- I've added a patch for what I think is an error in the file *main_importKnownPoses.cpp*, see the github issue I raised about it [here](https://github.com/alicevision/AliceVision/issues/1748).
- I've done a slight change to Meshroom's GUI, disabling the Gizmo/Trackball (which I never use) and instead enabling the Origin.

## Steps To Install/Use
You can find all commands and a little description in the comments at the top of the Dockerfile. Just be aware that, depending on your system, the build process can take quite some time (~30 min on my rather powerful computer, a fair bit longer on my laptop). Make sure that you have a steady internet connection. A few times, when I tried it on my (patchy) mobile internet, the build process stopped because some github repo couldn't be reached. It worked fine with my main internet connection. Also, you'll need some space on our hard drive. The final image requires about 4.15GB. During the build process Docker's build cache can swell up to roughly 30GB, which you can clear when you finished the build process.
