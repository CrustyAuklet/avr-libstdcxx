FROM crustyauklet/avr-libstdcxx:latest

ENV PREFIX=/opt/avr-gcc
WORKDIR /opt
ENV PATH=$PREFIX/bin:$PATH

RUN cd gcc && \
    git pull && \
    cd obj-avr && \
    ../configure --prefix=$PREFIX --target=avr --enable-languages=c,c++ --disable-nls --disable-libssp --with-dwarf2 --with-newlib --disable-__cxa_atexit --disable-threads --disable-shared --enable-static --disable-sjlj-exceptions --enable-libstdcxx --enable-lto --disable-hosted-libstdcxx && \
    make -s -j`nproc` && \
    make -s install
