# The Third-party Software List

[TOC]

## mDNSResponder

- Source
   + <https://github.com/HybridOS2/mDNSResponderHBD>
- Version
   + 2200.0.8+
- Building script:

```bash
git clone git@github.com:HybridOS2/mDNSResponderHBD.git -b mDNSResponder-2200.0.8-hbd
cd mDNSResponderHBD/mDNSPosix
make os=linux
sudo make install
```

### mbedtls-2.28

- Ubuntu 22.04 LTS
   + `libmbedtls-dev` - lightweight crypto and SSL/TLS library
- Source
   + <https://github.com//mDNSResponderHBD>
- Version
   + 2.28.0+
- Building script:

```bash
git clone git@github.com:Mbed-TLS/mbedtls.git -b mbedtls-2.28
mkdir build
cd build/
cmake ..
make
sudo make install
```

## HBDInetd

### libnl-3

- Ubuntu 22.04 LTS
   + `libnl-3-200` - library for dealing with netlink sockets
   + `libnl-3-dev` -  development library and headers for libnl-3
   + `libnl-genl-3-200` - library for dealing with netlink sockets - generic netlink
   + `libnl-genl-3-dev` - development library and headers for libnl-genl-3
   + `libnl-route-3-200` - library for dealing with netlink sockets - route interface
   + `libnl-route-3-dev` - development library and headers for libnl-route-3
   + `libnl-cli-3-200` - library for dealing with netlink sockets - cli helpers
   + `libnl-cli-3-dev` - development library and headers for libnl-cli-3
   + `libnl-idiag-3-200` - library for dealing with netlink sockets - inetdiag interface
   + `libnl-idiag-3-dev` - development library and headers for libnl-genl-3
   + `libnl-nf-3-200` - library for dealing with netlink sockets - netfilter interface
   + `libnl-nf-3-dev` - development library and headers for libnl-nf-3
   + `libnl-xfrm-3-200` - library for dealing with netlink sockets - package transformations
   + `libnl-xfrm-3-dev` - development library and headers for libnl-xfrm-3
- Source
   + <http://www.infradead.org/~tgr/libnl>
- Version
   + 3.2.25+
- Building script:

```bash
wget http://www.infradead.org/~tgr/libnl/files/libnl-3.2.25.tar.gz
tar zxf libnl-3.2.25.tar.gz
cd libnl-3.2.25
./configure --prefix={$PREFIX}
make
sudo make install
```

### OpenSSL

- Ubuntu 22.04 LTS
   + `libssl-dev` - Secure Sockets Layer toolkit - development files
   + `libssl3` - Secure Sockets Layer toolkit - shared libraries
- Source
   + <https://github.com/openssl/openssl>
- Version
   + 3.0.2+
- Building script:

```bash
wget https://github.com/openssl/openssl/releases/download/openssl-3.0.8/openssl-3.0.8.tar.gz
tar zxf openssl-3.0.8.tar.gz
cd openssl-3.0.8
./Configure --prefix={$PREFIX}/ssl --openssldir={$PREFIX}/ssl '-Wl,--enable-new-dtags,-rpath,$(LIBRPATH)'
make
sudo make install
```

### Linux WPA/WPA2/IEEE 802.1X Supplicant

- Ubuntu 22.04 LTS
   + `wpasupplicant`
   + `libwpa-client-dev`
- Source
   + <http://w1.fi/wpa_supplicant/>
- Version
   + 2.10+
- Building script:

```bash
wget https://w1.fi/releases/wpa_supplicant-2.10.tar.gz
tar zxf wpa_supplicant-2.10.tar.gz
cd wpa_supplicant-2.10/wpa_supplicant
cp defconfig .config
make CONFIG_BUILD_WPA_CLIENT_SO=1 DSTDIR={$PREFIX}/usr/local
sudo make CONFIG_BUILD_WPA_CLIENT_SO=1 DSTDIR={$PREFIX} install
```

