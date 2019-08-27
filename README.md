## Introduction
The NFD and it's related tools and software packages can be installed on platforms mentioned in the official **[NFD](http://named-data.net/doc/NFD/current/)** documentation. However, a successful installation on constrained hardware like the **[Raspberry Pi](https://www.raspberrypi.org/)** with the official [Raspbian OS](https://www.raspberrypi.org/downloads/raspbian/) is tricky and sometimes suggested to perform a cross-compilation and install the generated binaries. Although it's possible to install [Ubuntu MATE](https://ubuntu-mate.org/download/) and install the relevant pre-compiled binaries directly, one might need to go through the painstaking process of the MATE environment setup, and not be able to modify code to test/experiment self-written features.

In this documentation, I will talk about how to perform a step-by-step (as noob as possible) installation of NFD and related software from source on the Raspberry Pi running official Raspbian OS. The trick is two-folds-

* Install **[Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)** instead of full Raspbian Stretch with Pixel desktop to save a lot of memory.
* Use `clang++` instead of GCC as default compiler to speedup compilation.

The process discussed here has been successfully tested with both Raspberry Pi model 2 and 3b with wireless interface networking.

## Prepare the Pi
* Follow official [Raspbian installation](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) guide to install Raspbian Lite on the SD card.
* Connect Raspberry Pi with HDMI to an external display, attach power cord and a keyboard.
* Boot into your Pi device.
* Login:
	* default user : **pi**
	* default password : **raspberry**
* **Prerequisites for Model 2 only**
	* Plug in USB Wifi dongle.
	* Follow this tutorial on [How to setup usb wifi on Raspberry Pi 2](https://www.electronicshub.org/setup-wifi-raspberry-pi-2-using-usb-dongle/).
	* Reboot Pi using 
```bash
sudo reboot
```
* Connect to wifi by using (do a little google for details or follow [this](https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md)) 
```bash
sudo raspi-config
```
## Installing `clang++` compiler
Using `clang++` helps speedup the compilation and also helps to avoid memory hogging by `gcc` and therefore does not require increasing swap memory.

* Follow [this article to install `clang 8`](https://solarianprogrammer.com/2018/04/22/raspberry-pi-raspbian-install-clang-compile-cpp-17-programs/) compiler (not tested with newer version).

## Installing NFD and related NDN software

```bash
sudo apt-get update
sudo apt-get upgrade
```

#### Prerequisites
Note: minimum boost library requirement is **1.58**.
```bash
sudo apt-get install git build-essential pkg-config libboost1.62-all-dev \
                     libsqlite3-dev libssl-dev libpcap-dev
```

#### Manpage and API docs
Note: I skipped them. Details on how to install are available on [NFD](http://named-data.net/doc/NFD/current/INSTALL.html) installation guide.
```bash
sudo apt-get install doxygen graphviz python-sphinx
```

## Download and install ndn-cxx
```bash
git clone --recursive https://github.com/named-data/ndn-cxx.git
cd ndn-cxx
CXX=clang++ ./waf configure
./waf
sudo ./waf install
sudo ldconfig
```

## Download and install NFD
```bash
git clone --recursive https://github.com/named-data/NFD.git
cd NFD
CXX=clang++ ./waf configure
./waf
sudo ./waf install
```
After installing NFD from source, you need to create a proper config file. If default location for ./waf configure was used, this can be accomplished by simply copying the sample configuration file:
```bash
sudo cp /usr/local/etc/ndn/nfd.conf.sample /usr/local/etc/ndn/nfd.conf
```
To make NFD properly work, you also need to either disable IPv6 on NFD config file or enable IPv6 probing on Raspberry Pi. I disabled IPv6 in NFD config for now by setting **`enable_v6 no`** to all relevant entries.

## Install ndn-tools (optional)
```bash
git clone --recursive https://github.com/named-data/ndn-tools.git
cd ndn-tools
CXX=clang++ ./waf configure
./waf
sudo ./waf install
```
## Install NDN-routing infoedit tool (optional)
```bash
git clone --recursive https://github.com/NDN-Routing/infoedit.git
cd infoedit
make
sudo make install
```

Note: Edit the `Makefile` as following:

```bash
all: infoedit

infoedit: infoedit.hpp infoedit.cpp
	clang++ -o infoedit -std=c++14 infoedit.cpp -lboost_program_options

install: infoedit
	cp infoedit /usr/local/bin/infoedit

uninstall:
	rm /usr/local/bin/infoedit
```

### Full set of documentation (tutorials + API) in build/docs
`./waf docs`

### Only tutorials in `build/docs`
`./waf sphinx`

### Only API docs in `build/docs/doxygen`
`./waf doxgyen`

## General
All other documentations can be found at the [official NFD install page](http://named-data.net/doc/NFD/current/).