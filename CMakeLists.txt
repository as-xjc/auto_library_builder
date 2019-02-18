cmake_minimum_required(VERSION 3.9)
project(auto_library_builder)

set(INSTALL_PATH ${CMAKE_INSTALL_PREFIX})

message(STATUS "install path: ${INSTALL_PATH}")

include(ExternalProject)

ExternalProject_Add(libhiredis
        EXCLUDE_FROM_ALL 1
        URL https://github.com/redis/hiredis/archive/v0.13.3.tar.gz
        CONFIGURE_COMMAND ""
        BUILD_IN_SOURCE 1
        BUILD_COMMAND make
        INSTALL_COMMAND make PREFIX=${INSTALL_PATH} install
        )

ExternalProject_Add(libz
        EXCLUDE_FROM_ALL 1
        URL http://prdownloads.sourceforge.net/libpng/zlib-1.2.11.tar.gz?download
        CMAKE_CACHE_ARGS
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DCMAKE_BUILD_TYPE:STRING=Release
            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
        )

ExternalProject_Add(libboost
        EXCLUDE_FROM_ALL 1
        DEPENDS libz
        URL https://dl.bintray.com/boostorg/release/1.69.0/source/boost_1_69_0.tar.gz
        CONFIGURE_COMMAND ./bootstrap.sh
        BUILD_IN_SOURCE 1
        BUILD_COMMAND ""
        INSTALL_COMMAND ./b2 cxxflags=--std=c++14 link=static install --prefix=${INSTALL_PATH} -sZLIB_INCLUDE=${INSTALL_PATH}/include -sZLIB_LIBPATH=${INSTALL_PATH}/lib
        )

ExternalProject_Add(libopenssl
        EXCLUDE_FROM_ALL 1
        URL https://www.openssl.org/source/openssl-1.0.2q.tar.gz
        CONFIGURE_COMMAND ./config --prefix=${INSTALL_PATH}
        BUILD_IN_SOURCE 1
        BUILD_COMMAND make
        INSTALL_COMMAND make install
        )

ExternalProject_Add(libevent
        EXCLUDE_FROM_ALL 1
        URL https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
        CONFIGURE_COMMAND ./configure --prefix=${INSTALL_PATH}
        BUILD_IN_SOURCE 1
        BUILD_COMMAND make
        INSTALL_COMMAND make install
        )

ExternalProject_Add(libthrift
        EXCLUDE_FROM_ALL 1
        DEPENDS libboost libopenssl libevent
        URL http://apache.website-solution.net/thrift/0.11.0/thrift-0.11.0.tar.gz
        CONFIGURE_COMMAND ./configure --prefix=${INSTALL_PATH}
            --enable-tutorial=no
            --enable-tests=no
            --enable-shared=no
            --with-boost=${INSTALL_PATH}
            --with-openssl=${INSTALL_PATH}
            --with-libevent=${INSTALL_PATH}
            --with-cpp
            --without-qt4
            --without-qt5
            --without-c_glib
            --without-csharp
            --without-java
            --without-erlang
            --without-nodejs
            --without-lua
            --without-python
            --without-perl
            --without-php
            --without-php_extension
            --without-dart
            --without-ruby
            --without-haskell
            --without-go
            --without-rs
            --without-haxe
            --without-dotnetcore
            --without-d

        BUILD_IN_SOURCE 1
        BUILD_COMMAND make
        INSTALL_COMMAND make install
        )

ExternalProject_Add(libprotobuf
        EXCLUDE_FROM_ALL 1
        URL https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-cpp-3.6.1.tar.gz
        SOURCE_SUBDIR cmake
        CMAKE_CACHE_ARGS
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DCMAKE_BUILD_TYPE:STRING=Release
            -Dprotobuf_BUILD_TESTS:BOOL=OFF
            -Dprotobuf_WITH_ZLIB:BOOL=OFF
            -Dprotobuf_MSVC_STATIC_RUNTIME:BOOL=OFF
            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
        )

ExternalProject_Add(libc-ares
        EXCLUDE_FROM_ALL 1
        URL https://c-ares.haxx.se/download/c-ares-1.15.0.tar.gz
        CMAKE_CACHE_ARGS
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DCARES_SHARED:BOOL=OFF
            -DCARES_STATIC:BOOL=ON
            -DCARES_STATIC_PIC:BOOL=ON
            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
        )

ExternalProject_Add(libgrpc
        DEPENDS libc-ares libprotobuf libz libopenssl
        EXCLUDE_FROM_ALL 1
        URL https://github.com/grpc/grpc/archive/v1.18.0.tar.gz
        CMAKE_CACHE_ARGS
            -DCMAKE_BUILD_TYPE:STRING=Release
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DgRPC_INSTALL:BOOL=ON
            -DgRPC_BUILD_TESTS:BOOL=OFF
            -DCMAKE_PREFIX_PATH:PATH=${INSTALL_PATH}

            -DgRPC_PROTOBUF_PROVIDER:STRING=package
            -DProtobuf_DIR:PATH=${INSTALL_PATH}

            -DgRPC_ZLIB_PROVIDER:STRING=package
            -DZLIB_ROOT:STRING=${INSTALL_PATH}

            -DgRPC_CARES_PROVIDER:STRING=package
            -Dc-ares_DIR:PATH=${INSTALL_PATH}

            -DgRPC_SSL_PROVIDER:STRING=package
            -DOPENSSL_ROOT_DIR:PATH=${INSTALL_PATH}

            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
        )

ExternalProject_Add(libmongoc
        DEPENDS libopenssl
        EXCLUDE_FROM_ALL 1
        URL https://github.com/mongodb/mongo-c-driver/releases/download/1.13.0/mongo-c-driver-1.13.0.tar.gz
        CMAKE_CACHE_ARGS
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DCMAKE_BUILD_TYPE:STRING=Release
            -DENABLE_AUTOMATIC_INIT_AND_CLEANUP:BOOL=OFF
            -DCMAKE_PREFIX_PATH:PATH=${INSTALL_PATH}
            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
        )

ExternalProject_Add(libcpprest
        DEPENDS libboost libopenssl
        EXCLUDE_FROM_ALL 1
        URL https://github.com/Microsoft/cpprestsdk/archive/v2.10.5.tar.gz
        SOURCE_SUBDIR Release
        CMAKE_CACHE_ARGS
            -DCMAKE_POSITION_INDEPENDENT_CODE:BOOL=TRUE
            -DCMAKE_BUILD_TYPE:STRING=Release
            -DBUILD_SHARED_LIBS:BOOL=OFF
            -DCMAKE_PREFIX_PATH:PATH=${INSTALL_PATH}
            -DCMAKE_INSTALL_PREFIX:PATH=${INSTALL_PATH}
            -DBUILD_TESTS:BOOL=OFF
            -DBUILD_SAMPLES:BOOL=OFF
        )