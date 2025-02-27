FROM ubuntu:16.04
LABEL maintainer="hcorion"


ENV QTVER=5.15.2
ENV QTVERMIN=515
ENV LLVMVER=11
ENV GCCVER=9

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

RUN \
	apt-get update -y && \
	apt-get install -y curl libcurl4-openssl-dev software-properties-common apt-transport-https && \
	curl -Lvs https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
	add-apt-repository -y ppa:beineri/opt-qt-${QTVER}-xenial && \
	add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
	add-apt-repository -y "deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-${LLVMVER} main" && \
	apt-get update -y --fix-missing && \
	\
	apt-get -qq install -y \
	build-essential binutils $GCC_BINARY $GXX_BINARY git ccache bc $CLANG_BINARY $CLANGXX_BINARY $LLD_BINARY \
	libasound2-dev libopenal-dev freeglut3-dev libudev-dev libevdev-dev libpulse-dev libxrandr-dev \
	libwayland-dev libwayland-cursor0 libwayland-client0 libwayland-server0 \
	desktop-file-utils dput gnupg pbuilder ubuntu-dev-tools apt-file \
	qt${QTVERMIN}base qt${QTVERMIN}declarative qt${QTVERMIN}xmlpatterns qt${QTVERMIN}svg && \
	apt-get install --only-upgrade dpkg
	
RUN \
	curl -sLO http://http.us.debian.org/debian/pool/main/g/glew/libglew2.2_2.2.0-1_amd64.deb && \
	curl -sLO http://http.us.debian.org/debian/pool/main/g/glew/libglew-dev_2.2.0-1_amd64.deb && \
	dpkg -i ./*.deb && \
	rm ./*.deb

# Setup newer CMake
ENV CMAKEVER=3.16.9
RUN \
	curl -sLO https://cmake.org/files/v${CMAKEVER%.*}/cmake-${CMAKEVER}-Linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-Linux-x86_64.sh --skip-license && \
	rm ./cmake*.sh && \
	cmake --version

# We gotta build our own version of Ninja as well
ENV NINJAVER=1.10.1
RUN \
	curl -sL -o ninja.tar.gz https://github.com/ninja-build/ninja/archive/v${NINJAVER}.tar.gz && \
	tar -xf ninja.tar.gz && \
	cd ninja-* && \
	cmake -Bbuild-cmake -H. && \
	cmake --build build-cmake && \
	cp ./build-cmake/ninja /usr/bin/ninja && \
	cd ../ && \
	rm -r ./ninja-* && \
	rm ./*.tar.gz

# Need to manually build vulkan headers and libvulkan
ENV VULKAN_HEADER_VER=1.2.154.0
ENV VULKAN_LOADER_VER=1.2.154.1
RUN \
	curl -sL -o Vulkan-Headers.tar.gz https://github.com/KhronosGroup/Vulkan-Headers/archive/sdk-${VULKAN_HEADER_VER}.tar.gz && \
	tar -xf Vulkan-Headers*.tar.gz && \
	cd Vulkan-Headers* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Headers*
RUN \
	curl -sL -o Vulkan-Loader.tar.gz https://github.com/KhronosGroup/Vulkan-Loader/archive/sdk-${VULKAN_LOADER_VER}.tar.gz && \
	tar -xf Vulkan-Loader.tar.gz && \
	cd Vulkan-Loader* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib -DCMAKE_INSTALL_SYSCONFDIR=/etc -DCMAKE_INSTALL_DATADIR=/share && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Loader*

# We need to build a custom version of SDL2 as well
ENV SDL2VER=2.0.12
RUN \
	curl -sL -o SDL2.tar.gz https://www.libsdl.org/release/SDL2-${SDL2VER}.tar.gz && \
	tar -xf SDL2.tar.gz && \
	cd SDL2* && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$CLANG_BINARY -DCMAKE_CXX_COMPILER=$CLANGXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DSDL_DLOPEN=ON -DARTS=OFF -DESD=OFF -DNAS=OFF -DALSA=ON -DPULSEAUDIO_SHARED=ON -DRPATH=OFF -DCLOCK_GETTIME=ON && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./SDL2*

ENV SHELLCHECKVER=0.7.1
RUN \
	curl -sL -o shellcheck.tar.gz https://github.com/koalaman/shellcheck/releases/download/v0.7.1/shellcheck-v${SHELLCHECKVER}.linux.x86_64.tar.xz && \
	tar -xf shellcheck.tar.gz && \
	cd shellcheck* && \
	cp ./shellcheck /usr/bin/ && \
	cd .. && \
	rm -r ./shellcheck* && \
	shellcheck --version