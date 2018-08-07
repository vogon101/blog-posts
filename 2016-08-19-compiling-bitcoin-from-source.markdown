---
layout: post
title: "Compiling Bitcoin from Source"
date: 2016-08-19 15:19
comments: true
categories: 
 - bitcoin
 - linux
 - compiling
 - open-source
---
I recently tried to compile bitcoin from source on a Ubuntu laptop for fun. I have wanted to do this for a while and I finally got around to doing it. However, all of the guides I found were either outdated, wrong or both. So this is my "guide" to compiling bitcoin from source. It seems to work for Bitcoin v0.12.1 on Ubuntu 16.04 LTS. Beyond that I dont know. For more information see the very useful guide on github: [here](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md).
<!-- more -->
First download the bitcoin source from github like so:
``` bash
mkdir -p src && cd src
git clone https://github.com/bitcoin/bitcoin.git
git checkout v0.12.1
```

Next we need to install all of the dependencies. There are a lot so you can either copy each command from here or download them as an sh from [gist](https://gist.github.com/vogon101/72ae37cdcfdf89ba185aaff35dc0c9a0).

``` bash
sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
sudo apt-get install libqrencode-devsudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install libdb4.8-dev libdb4.8++-dev
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
sudo apt-get install libqrencode-dev
```

With all the packages in place we can now actually build bitcoin. This is as simple as:

``` bash
./autogen.sh
./configure
make
sudo make install # optional
```

The optional make install will add the bitcoin commands to your classpath so that you can simply run bitcoin-qt to start it. The initial compile will take quite a long time but once it is done simply run bitcoin-qt to open your wallet.

Thanks for reading,

Freddie
