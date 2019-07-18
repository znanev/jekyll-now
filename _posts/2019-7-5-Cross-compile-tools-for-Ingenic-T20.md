---
layout: post
title: Cross-compile tools for Ingenic T20
---
The [Ingenic T20](../files/T20_PB.PDF) video SoC is used by quite a lot of Chinese IP cameras. One notable example is Xiaomi Dafang, which is capable of streaming 1080p video in H.264 format and includes pan/tilt motors. This camera is great for hacking, because it runs Linux and installing custom firmware is quite easy: https://github.com/EliasKotlyar/Xiaomi-Dafang-Hacks.
Once custom firmware is installed, you can gain SSH access and execute commands on the camera itself, or you can edit the configuration files directly from within the running system. The default editor shipped with the custom firmware is **vi**, which to be honest is quite a challenge to be used effectively. I personally prefer the **nano** editor, so following is a log of the steps that I did in order to cross-compile a binary, that can be executed on Ingenic T20 CPUs.

## Main Repository for Low-Level Development
Clone this repository using following command:

```s
git clone --recurse-submodules git@github.com:Dafang-Hacks/Main.git
```

## Setup build environment (Ubuntu / Debian)

### Install required dependencies


```
sudo apt install build-essential git gcc-mips-linux-gnu autoconf libtool cmake
```

### Use already prepared Docker image

```
mkdir ~/dafang
cd ~/dafang
docker run --rm -ti -v $(pwd):/root/ daviey/dafang-cross-compile:latest
git clone --recurse-submodules https://github.com/Dafang-Hacks/Main.git
cd Main
```

### Install some pre-requisites in Docker environment

```
apt update
apt install nano wget
```

### Cross-compile ncurses library
Shared library `ncurses` is required by some text-based tools and editors like `nano`, `htop` etc. In order to build these tools, first step is cross-compiling the `ncurses` library:

#### 1. Download ncurses
Download the source archive for ncurses-6.1 into the `Main` directory:

```
wget https://ftp.gnu.org/gnu/ncurses/ncurses-6.1.tar.gz
tar xvf ncurses-6.1
```

#### 2. Create a build script
Create a file with name 'compile-ncurses.sh' and the following contents:

```
#!/usr/bin/env bash
cd ncurses-6.1
TOOLCHAIN=$(pwd)/../toolchain/bin
INSTALLPATH=$(pwd)/..
CROSS_COMPILE=$TOOLCHAIN/mips-linux-gnu-
make clean
export CC=${CROSS_COMPILE}gcc
export CFLAGS="-muclibc -O3"
export CPPFLAGS="-muclibc -O3"
export LDFLAGS="-muclibc -O3"

./autogen.sh
./configure --host=mips-linux-gnu --disable-database --disable-db-install --with-fallbacks=vt100,vt102,vt300,screen,xterm,xterm-256color,tmux-256color,screen-256color --without-manpages --without-normal --without-progs --without-debug --without-test --enable-widec --prefix=${INSTALLPATH}/_install

make
make install
cd ..
```

Make the script executable with:

```
chmod +x compile-ncurses.sh
```

Note the argument values of the *--with-fallbacks* option of the configure script: these will be the terminal emulators available to **nano** or other *ncurses*-based tools (like **htop**). The default configuration is *xterm*, but if you like colours support, you can configure another one, e.g.:

```
TERM=tmux-256color
```

#### 3. Execute the build script

```
./compile-ncurses.sh
```

If the build completes successfully, `ncurses` headers and libraries files will be available in */root/Main/_install* directory. 

### Download, extract source code for required tools and cross-compile
#### 1. Nano

```
wget https://www.nano-editor.org/dist/v4/nano-4.3.tar.gz
tar xvf nano-4.3.tar.gz
```

Create a file with name 'compile-nano.sh' and the following contents:

```
#!/usr/bin/env bash
export INSTALLPATH="$(pwd)/_install"
TOOLCHAIN=$(pwd)/toolchain/bin
CROSS_COMPILE=$TOOLCHAIN/mips-linux-gnu-
export CC=${CROSS_COMPILE}gcc
export CXX=${CROSS_COMPILE}cpp
export LD=${CROSS_COMPILE}ld
export CFLAGS="-muclibc -O2 -static"
export CPPFLAGS="-muclibc -O2 -I${INSTALLPATH}/include -I${INSTALLPATH}/include/ncursesw"
export LDFLAGS="-muclibc -O2 -L${INSTALLPATH}/lib"

cd nano-4.3

./configure --prefix=${INSTALLPATH} --host=mipsel-linux --enable-utf8 --enable-nls --enable-color --enable-multibuffer --enable-nanorc

# Last known good configuration:
#./configure --prefix=/root/Main/_install --host=mipsel-linux --enable-utf8 --disable-nls --enable-color --enable-extra --enable-multibuffer --enable-nanorc

make -j4
make install
```

Make the script executable with:

```
chmod +x compile-nano.sh
```

Execute the build script:

```
./compile-nano.sh
```

If everything went fine, the cross-compiled executable will be available here:

```
/root/Main/_install/bin/nano
```

#### 1. Htop

```
wget https://hisham.hm/htop/releases/2.2.0/htop-2.2.0.tar.gz
tar xvf htop-2.2.0.tar.gz
```

Create a file with name 'compile-htop.sh' and the following contents:

```
#!/usr/bin/env bash
export INSTALLPATH="$(pwd)/_install"
TOOLCHAIN=$(pwd)/toolchain/bin
CROSS_COMPILE=$TOOLCHAIN/mips-linux-gnu-
export CC=${CROSS_COMPILE}gcc
export CXX=${CROSS_COMPILE}cpp
export LD=${CROSS_COMPILE}ld
export CFLAGS="-muclibc -O2 -static"
export CPPFLAGS="-muclibc -O2 -I${INSTALLPATH}/include -I${INSTALLPATH}/include/ncursesw"
export LDFLAGS="-muclibc -O2 -L${INSTALLPATH}/lib"

cd htop-2.2.0

./configure --prefix=${INSTALLPATH} --host=mipsel-linux

make -j4
make install
```


Make the script executable with:

```
chmod +x compile-nano.sh
```

Execute the build script:

```
./compile-nano.sh
```

If everything went fine, the cross-compiled executable will be available here:

```
/root/Main/_install/bin/htop
```

