# C++ Standard Library (P0829 Freestanding) for AVR microcontrollers

This is a collection of docker containers to build an example implementation of 
P0829 - freestanding for AVR microcontrollers. The origional work is located at 
[https://gitlab.com/avr-libstdcxx](https://gitlab.com/avr-libstdcxx). 
Please open any issues with the compiler or C++ libraries there. 
This repo is only responsible for building ubuntu and win32 executables.

The reason for this repo is two-fold:

- Provide pre-compiled distrobution for windows
- provide pre-compiled distrobution for non-musl linux systems

The source repository has a docker container that build the compiler using alpine.
This makes the final binary non-portable to ubuntu systems. This could be solved by 
installing musl on ubuntu, and statically linking to other libraries in the alpine container. 
But to build for windows we need MinGW, which is not available on mainline Alpine. 
For the least amount of effort, and the smallest change to the Dockerfiles from the 
source repo, I instead just changed the base image to ubuntu.
