Build this image with the following command (memory argument is optional):

`docker build -m 4G -t <image_name>:<tag> .`

Once complete, the toolchains will be in archives in `/opt` in the container.

- `/opt/avr-gcc-9.2.0-P0829-x86_64-linux.tar.gz`
- `/opt/avr-gcc-9.2.0-P0829-x86_64-w64-mingw32.tar.gz`
