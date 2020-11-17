# deb-example

This is a simple example to illustrate how to make a patch for debian package. I would divid the entire process into differe stages. Our goal is to add a shell script into the existing package and execute the script during the process of `apt install`.  

* [download]  
  get source file
* [study]  
  study the standard or conventions of debian community
* [edit]  
  make a patch
* [package]  
  submit to debian or create a private debian repository

## [download]

use `apt source [package]` to download the source files, and this example is based on [hello](http://www.gnu.org/software/hello/) package.

* command
```
apt source hello
```

* result
```
demonic@demonic-System-Product-Name[07:34 暮]~/gitPro/deb-example (main)> apt source hello
正在讀取套件清單... 完成
需要下載 734 kB 的原始套件檔。
下載:1 http://tw.archive.ubuntu.com/ubuntu bionic-updates/main hello 2.10-1build3 (dsc) [1707 B]
下載:2 http://tw.archive.ubuntu.com/ubuntu bionic-updates/main hello 2.10-1build3 (tar) [726 kB]
下載:3 http://tw.archive.ubuntu.com/ubuntu bionic-updates/main hello 2.10-1build3 (diff) [6224 B]
取得 734 kB 用了 0秒 (4615 kB/s)
dpkg-source: 資訊: extracting hello in hello-2.10
dpkg-source: 資訊: unpacking hello_2.10.orig.tar.gz
dpkg-source: 資訊: unpacking hello_2.10-1build3.debian.tar.xz
demonic@demonic-System-Product-Name[07:35 暮]~/gitPro/deb-example (main)> ls
hello-2.10  hello_2.10-1build3.debian.tar.xz  hello_2.10-1build3.dsc  hello_2.10.orig.tar.gz
```

There are four different file now.
* hello-2.10  
  all files related to hello package
* hello_2.10-1build3.debian.tar.xz  
  a compressed file stores debian related files, which is used to control deb-series command
* hello_2.10-1build3.dsc  
  dsc means debian source control files
* hello_2.10.orig.tar.gz  
  a compressed file stores source files of hello package

Note that: if the command cannot execute correctly, please edit `/etc/apt/source.list` and remove the `#` from the head of `deb-src`  


## [study]


## [edit]

You can use package maintainer scripts to interact with system during runing `apt install`. The rule is a little like git hook script and create a executable file with the specific name, and they are `postrm` `prerm` `preinst` `postinst`.

* debian/postinst
```shell
#!/bin/sh
./testing.sh
cp ./testing.sh /usr/bin
```

* debian/prerm
```shell
#!/bin/sh
rm -f /usr/bin/testing.sh
```

* testing.sh
```shell
#!/bin/sh
echo "this is a test from Chungyi Chi"
```

To find the package easily, I change the package name.
* debian/changelog
```=12
Package: chungyi-hello
```


* command
```
debuild -uc -us
```

## [package]
create a temporary directory `mkdir /tmp/repo/` and add the directory into apt source.list

* command
```
mkdir /tmp/repo
sudo vim /etc/apt/source.list.d/localhost.list
```

* localhost.d
```
deb [trusted=yes] file:/tmp/repo ./
```

`[trusted=yes]` is used to allow me to use unsigned debian package source, it would take a little time to build a formal debian repository. Current version is for developer, if you are really interested at that, you can follow the instruction to build from scratch ([Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864))

Finally, we finish all the instructions and it's very close to the end.

```
```

## Reference
* [3.4. Creating a local Debian repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)
* [Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864)
* [Creating-a-local-Debian-repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)