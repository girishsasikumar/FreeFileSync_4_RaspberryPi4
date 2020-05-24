# FreeFileSync_4_RaspberryPi4
Free file sync compiled to work on Raspberry Pi 4 running Raspberrian

Thanks to instructions from jmsxl and jper at https://freefilesync.org/forum/viewtopic.php?t=6190
Thanks to Paul at https://solarianprogrammer.com/2017/12/08/raspberry-pi-raspbian-install-gcc-compile-cpp-17-programs/
Thanks to jeffli678 at https://github.com/jeffli678/build-FreeFileSync

Instructtions to build FreeFileSync 10.14

# update system
sudo apt update && sudo apt upgrade -y

# Install gcc
sudo apt install git

git clone https://bitbucket.org/sol_prog/raspberry-pi-gcc-binary.git

cd raspberry-pi-gcc-binary
tar -xjvf gcc-10.1.0-armhf-raspbian.tar.bz2
sudo mv gcc-10.1.0 /opt
cd ..
rm -rf raspberry-pi-gcc-binary

echo 'export PATH=/opt/gcc-10.1.0/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/opt/gcc-10.1.0/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
. ~/.bashrc
sudo ln -s /usr/include/arm-linux-gnueabihf/sys /usr/include/sys
sudo ln -s /usr/include/arm-linux-gnueabihf/bits /usr/include/bits
sudo ln -s /usr/include/arm-linux-gnueabihf/gnu /usr/include/gnu
sudo ln -s /usr/include/arm-linux-gnueabihf/asm /usr/include/asm
sudo ln -s /usr/lib/arm-linux-gnueabihf/crti.o /usr/lib/crti.o
sudo ln -s /usr/lib/arm-linux-gnueabihf/crt1.o /usr/lib/crt1.o
sudo ln -s /usr/lib/arm-linux-gnueabihf/crtn.o /usr/lib/crtn.o

# check gcc
gcc-10.1 --version

# I see the below in my Pi4
gcc-10.1 (GCC) 10.1.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

# Test gcc
nano if_test.cpp

# copy the below code
int main() {
     // if block with init-statement:
     if(int a = 5; a < 8) {
         std::cout << "Local variable a is < 8\n";
     } else {
         std::cout << "Local variable a is >= 8\n";
	 }
     return 0;
	 }

# compile, test and see the result
g++-10.1 -std=c++17 -Wall -pedantic if_test.cpp -o if_test

# install required software
sudo apt install build-essential libgtk-3-dev libboost-dev libssl-dev libcurl4-openssl-dev libssh2-1-dev

# install wxWidgets
wget https://github.com/wxWidgets/wxWidgets/releases/download/v3.1.2/wxWidgets-3.1.2.tar.bz2
tar xf wxWidgets-3.1.2.tar.bz2
cd wxWidgets-3.1.2/
mkdir gtk-build # or any other name you like
cd gtk-build
../configure --disable-shared --enable-unicode
make
make install # use sudo if necessary

#install boost
sudo apt install libboost-all-dev

# Download FFS source from
https://freefilesync.org/download/FreeFileSync_10.14_Source.zip

#Unzip and do the following changed

The Makefile is at FreeFileSync_10.14_Source/FreeFileSync/Source/Makefile. We need to change all occurrances (there should be two) of "g++" to "g++-10.1" .
also per jmsxl you need to: change gtk+-2.0 to gtk+-3.0 and add "-latomic" to the end of LINKFLAGS (there are a number of loactions, so change them all)

In "afs/sftp.cpp", add at line 58:
#define MAX_SFTP_OUTGOING_SIZE 30000
#define MAX_SFTP_READ_SIZE 30000
also In "afs/sftp.cpp", add at line 1662 (just before the #if)
#define LIBSSH2_SFTP_DEFAULT_MODE -1

Run "make" in folder FreeFileSync_10.14_Source/FreeFileSync/Source. It will take roughly hour plus.

The binary should be waiting for you in FreeFileSync_10.14_Source/FreeFileSync/Build/Bin.
I opened it in a terminal and executed: ./FreeFileSync_armv7l
