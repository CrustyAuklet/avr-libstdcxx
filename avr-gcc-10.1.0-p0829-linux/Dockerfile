FROM ubuntu:18.04

RUN apt-get -y update \
	&& apt-get install -y git wget build-essential libgmp-dev libmpfr-dev libmpc-dev flex \
	&& apt-get autoremove --purge -y \
	&& apt-get autoclean -y \
	&& rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PREFIX=/opt/avr-gcc

WORKDIR /opt

# First download and install avr-gcc binutils
# Since we don't need it after installation, just remove it completly afterwards
RUN wget https://ftpmirror.gnu.org/binutils/binutils-2.31.tar.xz \
	&& tar -xf binutils-2.31.tar.xz  \
	&& rm binutils-2.31.tar.xz \
	&& cd binutils-2.31  \
	&& mkdir obj-avr  \
	&& cd obj-avr  \
	&& ../configure --prefix=$PREFIX --target=avr --disable-nls \
	&& make -s -j`nproc` \
	&& make -s install \
	&& cd ../.. \
	&& rm -rf binutils-2.31

ENV PATH=$PREFIX/bin:$PATH

# Now get gcc and compile and install it for the first layer.
# This will make avr-gcc available and allow us to build the C-library
RUN git clone --depth 1 --branch p0829-10-release https://gitlab.com/avr-libstdcxx/gcc.git \
	&& cd gcc \
	&& mkdir obj-avr  \
	&& cd obj-avr \
	&& ../configure --prefix=$PREFIX --target=avr --enable-languages=c,c++ --disable-nls --disable-libssp --with-dwarf2 \
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
	&& ../configure --prefix=$PREFIX --build=`../config.guess` --host=avr \
	&& make -s -j`nproc` \
	&& make -s install \
	&& cd ../.. \
	&& rm -rf avr-libc-2.0.0

# We are now ready to run the compile of the freestanding libstdc++.
# Since we want to modify the contents of libstdc++ we will leave the files in there
RUN cd gcc/obj-avr && \
	../configure --prefix=$PREFIX --target=avr --enable-languages=c,c++ --disable-nls --disable-libssp --with-dwarf2 --with-newlib --disable-__cxa_atexit --disable-threads --disable-shared --enable-static --disable-sjlj-exceptions --enable-libstdcxx --enable-lto --disable-hosted-libstdcxx && \
	make -s -j`nproc` && \
	make -s install
