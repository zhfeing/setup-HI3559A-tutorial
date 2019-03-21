# Setup SSH

For compiling _ssh_, you need source code of [zlib](https://www.zlib.net/), [openssl](https://www.openssl.org/) and [openssh](https://www.openssh.com/).

After you have downloaded these source code, you should compile them with  `aarch64` toolset.

## Compile *Zlib*

```
prefix=/path/you/want/to/set CC=aarch64-himix100-linux-gcc ./configure
make
make install
```

After compilation, you will get shared libraries of _zlib_, which means that you need to point out the location of _zlib_ libraries: export `LD_LIBRARY_PATH`.

## Compile *OpenSSL*

```
./Configure --prefix=/path/you/want/to/set os/compiler:aarch64-himix100-linux-gcc
make
make install
```

After compilation, you will get static libraries of _openssl_, which means after compiled _openssh_, ssh will contain ssl inside its programme.

## Compile *OpenSSH*

```
./configure --host=aarch64-himix100-linux --with-libs --with-zlib=/path/you/set/zlib --with-ssl-dir=/path/you/set/openssl  --disable-etc-default-login CC=aarch64-himix100-linux-gcc AR=aarch64-himix100-linux-ar
make
```

While compilation, if it cannot find `<openssl/opensslv.h>`, trace the log and modify file `configure` to add _openssl_ path. After this, when it shows:

```
relocation R_AARCH64_ADR_PREL_PG_HI21 against external symbol stderr@@GLIBC_2.17' can not be used when making a shared object; recompile with -fPIC ... unresolvable R_AARCH64_ADR_PREL_PG_HI21 relocation against symbol stderr@@GLIBC_2.17'
```

replace `-fPIE` to `-fPIC`, and remove `-pie` in `configure`.

## Install *OpenSSH*

Make sure these folders exist in your Hi3559A board:

```
/usr/local/bin
/usr/local/etc
/usr/libexec
/var/run
/var/empty
```

if you do not have them all, try to creat them.

* Then copy files **scp, sftp, ssh, sshd, ssh-agent, ssh-keygen and ssh-keyscan** to `/usr/local/bin`.

* Copy files **moduli, ssh_config, sshd_config** to `/usr/local/etc`

* Copy **sftp-server, ssh-keysign** to `/usr/libexec`

After copying, you need to generate *Key* files:

```
cd /usr/local/etc/
ssh-keygen -t rsa -f ssh_host_rsa_key -N ""
ssh-keygen -t dsa -f ssh_host_dsa_key -N ""
ssh-keygen -t ecdsa -f ssh_host_ecdsa_key -N ""
ssh-keygen -t dsa -f ssh_host_ed25519_key -N ""
chmod 600 ssh_host_ed25519_key
```

Finally, add

```
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
``` 
to file `/etc/passwd`. Don't forget to set your password to Hi3559A by `passwd root`

## Test

Try run `/usr/local/bin/sshd` to start _ssh_ service, you will see the PID of `sshd` if you run command `ps`.

> *ps: do not forger to add shared libraries of zlib to your board and set environment variable by `export LD_LIBRARY_PATH=/path/to/lib:$LD_LIBRARY_PATH`.*