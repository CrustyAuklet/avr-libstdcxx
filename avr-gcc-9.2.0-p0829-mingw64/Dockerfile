FROM docker.pkg.github.com/crustyauklet/avr-libstdcxx/libstdcxx-base:latest

RUN apt-get -y update \
	&& apt-get install -y curl mingw-w64 mingw-w64-common mingw-w64-i686-dev mingw-w64-x86-64-dev mingw-w64-tools\
	&& apt-get autoremove --purge -y \
	&& apt-get autoclean -y \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PREFIX_WIN=/opt/avr-gcc-mingw32
ENV HOST_WIN="x86_64-w64-mingw32"

WORKDIR /opt

#First download and install avr-gcc binutils
# Since we don't need it after installation, just remove it completly afterwards
RUN wget https://ftpmirror.gnu.org/binutils/binutils-2.31.tar.xz \
	&& tar -xf binutils-2.31.tar.xz  \
	&& rm binutils-2.31.tar.xz \
	&& cd binutils-2.31  \
	&& mkdir obj-avr  \
	&& cd obj-avr  \
	&& ../configure --prefix=$PREFIX_WIN --target=avr --disable-nls --host=$HOST_WIN --build=`../config.guess` \
	&& make -s -j`nproc` \
	&& make -s install \
	&& cd ../.. \
	&& rm -rf binutils-2.31

# Now get gcc and compile and install it for the first layer.
# This will make avr-gcc available and allow us to build the C-library
RUN cd gcc \
	&& git pull \
	&& chmod +x ./contrib/download_prerequisites \
	&& ./contrib/download_prerequisites \
	&& mkdir obj-avr-win32  \
	&& cd obj-avr-win32 \
	&& ../configure --prefix=$PREFIX_WIN --target=avr --enable-languages=c,c++ --disable-nls --disable-libssp --with-dwarf2 --host=$HOST_WIN --build=`../config.guess` \
	&& make -s -j`nproc` \
	&& make -s install

# Get avr-libc, this will allow us to use avr-libc as the c library for libstdc++ needed for running on the avr instead of newlib
# Just like with binutils, we're not going to use this afterwards, so just remove everything to reduce the size of the image
RUN wget http://download.savannah.gnu.org/releases/avr-libc/avr-libc-2.0.0.tar.bz2 \
	&& tar -xf avr-libc-2.0.0.tar.bz2 \
	&& rm avr-libc-2.0.0.tar.bz2 \
	&& cd avr-libc-2.0.0 \
	&& mkdir obj-avr \
	&& cd obj-avr \
	&& ../configure --prefix=$PREFIX_WIN --build=`../config.guess` --host=avr \
	&& make -s -j`nproc` \
	&& make -s install \
	&& cd ../.. \
	&& rm -rf avr-libc-2.0.0

# We are now ready to run the compile of the freestanding libstdc++.
# Since we want to modify the contents of libstdc++ we will leave the files in there
RUN cd gcc/obj-avr-win32 \
	&& ../configure --prefix=$PREFIX_WIN --target=avr --host=$HOST_WIN --build=`../config.guess` --enable-languages=c,c++ --disable-nls --disable-libssp --with-dwarf2 --with-newlib --disable-__cxa_atexit --disable-threads --disable-shared --enable-static --disable-sjlj-exceptions --enable-libstdcxx --enable-lto --disable-hosted-libstdcxx && \
	make -s -j`nproc` && \
	make -s install

# archive the ubuntu and win32 version of the compiler and libcxx
# links used to have a nicely named root directory on the eventual host system
RUN echo "*** Creating archives for avr-gcc compilers ***" \
	&& ln -s /opt/avr-gcc /opt/avr-gcc-9.2.0-P0829 \
	&& tar chzvf avr-gcc-9.2.0-P0829-x86_64-linux.tar.gz avr-gcc-9.2.0-P0829 \
	&& rm /opt/avr-gcc-9.2.0-P0829 \
	&& ln -s $PREFIX_WIN /opt/avr-gcc-9.2.0-P0829 \
	&& tar chzvf avr-gcc-9.2.0-P0829-$HOST_WIN.tar.gz avr-gcc-9.2.0-P0829 \
	&& rm /opt/avr-gcc-9.2.0-P0829
