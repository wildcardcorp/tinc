# FIPS-compliant tinc
Born out of a need for FIPS-140-2 compliance, this fork simply calls `FIPS_mode_set()` in the `main` function and checks it again when it daemonizes. tinc's sole usage of OpenSSL for cryptographic functions and decision to use AES256 and SHA256 allows for a drop-in replacement of FIPS-capable OpenSSL. This version of tinc (1.0.32) has also been checked for insufficient entropy seeding and non-CSPRNG usage in cryptographic functions per FIPS Security Policy.

## Installation
#### Test Environment
Date: 2017-10-12

Operating System: CentOS Linux release 7.4.1708 (Core) 3.10.0-693.2.2.el7.x86_64 #1 SMP

Tinc version: 1.0.32

OpenSSL version: 1.0.2l

OpenSSL FIPS Object Module version: 2.0.14

#### Build/install Automake >= 1.14 (CentOS 7 doesn't have this version as of 2017-10-11)
------------------------
```
yum install -y perl autoconf automake libtool
curl -O http://ftp.gnu.org/gnu/automake/automake-1.14.tar.gz
tar xzf automake-1.14.tar.gz
cd automake-1.14
./configure
make && make install
```


#### Build OpenSSL FIPS Object Module
------------------------
This section is extremely important and follows the OpenSSL FIPS Object Module v2.0 [User Guide](https://www.openssl.org/docs/fips/UserGuide-2.0.pdf)! In order to build this module, you MUST obtain the FIPS Object Module source code from physical media (yes, read the User Guide). The `config`/`config no-asm` and `make` commands CANNOT BE CHANGED IN ANY WAY.
```
cp openssl-fips/openssl-fips-2.0.14.tar.gz ./
gunzip -c openssl-fips-2.0.14.tar.gz | tar xf -
cd openssl-fips-2.0.14
./config
make
make install
```


#### Build FIPS-capable OpenSSL
------------------------
You can specify other configuration arguments here. The above warning was only for the FIPS Object Module. According to OpenSSL documentation, specifying `--prefix=/usr` and `--openssldir=/usr` for `config` will overwrite the default installation of OpenSSL.
```
curl -XGET https://www.openssl.org/source/openssl-1.0.2l.tar.gz -O
tar xf openssl-1.0.2l.tar.gz
cd openssl-1.0.2l
./config fips shared no-ssl2 no-ssl3
make depend && make && make install
```


#### Build FIPS-approved mode Tinc
------------------------
`tincd` will be installed to `/usr/local/sbin`.
```
yum install -y git zlib-devel lzo-devel texinfo
git clone https://github.com/wildcardcorp/tinc
cd tinc
autoconf --include m4 --force
autoreconf --install
./configure --with-openssl-lib=/usr/local/ssl/lib --with-openssl-include=/usr/local/ssl/include
make
make install
```


#### Optional: Create symlinks for libcrypto and libssl
------------------------
In case you didn't overwrite the default OpenSSL libraries, you'll probably want to create symlinks to the generated libraries, so you don't have to specify LD_LIBRARY_PATH every time you run `tincd`
```
ln -s /usr/local/ssl/lib/libcrypto.so.1.0.0 /usr/lib64/libcrypto.so.1.0.0
ln -s /usr/local/ssl/lib/libssl.so.1.0.0 /usr/lib64/libssl.so.1.0.0
```

Example Output
------------------------

```
[root@localhost ~]# systemctl status tinc@testnet
● tinc@testnet.service - Tinc net testnet
   Loaded: loaded (/usr/lib/systemd/system/tinc@.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2017-10-12 14:28:47 EDT; 1h 30min ago
 Main PID: 9398 (tincd)
   CGroup: /system.slice/system-tinc.slice/tinc@testnet.service
           └─9398 /usr/sbin/tincd -n testnet -D

Oct 12 14:28:47 localhost.localdomain systemd[1]: Started Tinc net testnet.
Oct 12 14:28:47 localhost.localdomain systemd[1]: Starting Tinc net testnet...
Oct 12 14:28:47 localhost.localdomain tincd[9398]: Current FIPS mode: 0
Oct 12 14:28:47 localhost.localdomain tincd[9398]: Setting FIPS mode...
Oct 12 14:28:47 localhost.localdomain tincd[9398]: FIPS mode successfully set!
Oct 12 14:28:47 localhost.localdomain tincd[9398]: tincd 1.0.32-fips starting, debug level 0
Oct 12 14:28:47 localhost.localdomain tincd[9398]: FIPS mode verified for child process
Oct 12 14:28:47 localhost.localdomain tincd[9398]: /dev/net/tun is a Linux tun/tap device (tun mode)
Oct 12 14:28:47 localhost.localdomain tincd[9398]: Ready
```


## License
tinc is Copyright (C) 1998-2017 by:

Ivo Timmermans,
Guus Sliepen <guus@tinc-vpn.org>,
and others.

For a complete list of authors see the AUTHORS file.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or (at
your option) any later version. See the file COPYING for more details.
