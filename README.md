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
To study deb-series command, you need to understand the directory structure first. Except the debian directory, the rest of files belong to source of hello package. The debian directory is used to control deb-series command.

* debian/rules  
  Set variables for makefile files, like CFLAGS, etc.
* debian/control  
  Package description/configure file, which is similar to package.json when you develope JS applications.
* debian/watch  
  The upstream location  

If you need more information you can read docs([Debian Developers' Manuals](https://www.debian.org/doc/devel-manuals#debmake-doc)) by yourself, I found that it's not really important to this exercise.

## [edit]
You can use package maintainer scripts to interact with system during runing `apt install`. The rule is a little like git hook script and you should name the file under a specific rule, and they should be `postrm`, `prerm`, `preinst` or`postinst`.

* debian/postinst  
  show message during runing `apt install`
```shell
#!/bin/sh
echo "this is a test from Chungyi Chi"
```

* testing.sh  
  If use maintainer scripts to copy testing.sh to `/usr/bin`, you couldn't find the relationship when running `dpkg -S testing.sh`. We maybe should edit `configure`, `Makefile.am`, etc.
```shell
#!/bin/sh
echo "this is a test from Chungyi Chi"
```

* Makefile.am  
  Use dist_bin_SCRIPTS to install executable scripts. Note that run configure again
```
dist_bin_SCRIPTS = testing.sh
```

To find the package easily, I change the package name.
* debian/changelog
```
Package: chungyi-hello
```

Before build the package, you need to commit whatever you have edited. Make sure the commit message matchs dep-3
* command
```
* dpkg-source --commit
```

The following command is used to build a temporary version, if you want to send the debian patch to community. You must need to make a signature with your personal gpg key. In addition to this, remember to edit `debian/changelog`
* command
```
debuild -uc -us
```

## [package]
create a temporary directory `mkdir /tmp/repo/` and add the directory into apt source.list

Note that: copy the deb to `/tmp/repo` in advance
* command
```
mkdir /tmp/repo
sudo vim /etc/apt/source.list.d/localhost.list
cd /tmp/repo
dpkg-scanpackages -m . | gzip --fast > Packages.gz
```

* localhost.d
```
deb [trusted=yes] file:/tmp/repo ./
```

`[trusted=yes]` is used to allow me to use unsigned debian package source, it would take a little time to build a formal debian repository. Current version is for developer, if you are really interested at that, you can follow the instruction to build from scratch ([Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864))

Finally, we finish all the instructions and it's very close to the end.

```
demonic@demonic-System-Product-Name[09:16 朝]~/gitPro/deb-example (main)> cp chungyi-hello_2.10-1build3_amd64.deb /tm$
/repo
demonic@demonic-System-Product-Name[09:16 朝]~/gitPro/deb-example (main)> cd -
/tmp/repo
demonic@demonic-System-Product-Name[09:16 朝]/tmp/repo> dpkg-scanpackages -m . | gzip --fast > Packages.gz
dpkg-scanpackages: 資訊: Wrote 1 entries to output Packages file.
demonic@demonic-System-Product-Name[09:16 朝]/tmp/repo> sudo apt update
下載:1 file:/tmp/repo ./ InRelease
略過:1 file:/tmp/repo ./ InRelease
......
demonic@demonic-System-Product-Name[09:17 朝]/tmp/repo> sudo apt install chungyi-hello
正在讀取套件清單... 完成
正在重建相依關係
正在讀取狀態資料... 完成
下列套件將會被升級：
  chungyi-hello
升級 1 個，新安裝 0 個，移除 0 個，有 13 個未被升級。
需要下載 0 B/51.3 kB 的套件檔。
此操作完成之後，會多佔用 1024 B 的磁碟空間。
下載:1 file:/tmp/repo ./ chungyi-hello 2.10-1build3 [51.3 kB]
（讀取資料庫 ... 目前共安裝了 211736 個檔案和目錄。）
準備解開 .../chungyi-hello_2.10-1build3_amd64.deb ...
Unpacking chungyi-hello (2.10-1build3) over (2.10-1build3) ...
設定 chungyi-hello (2.10-1build3) ...
this is a test from Chungyi Chi
Processing triggers for install-info (6.5.0.dfsg.1-2) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
demonic@demonic-System-Product-Name[09:17 朝]/tmp/repo> testing.sh
this is a test from Chungyi Chi
demonic@demonic-System-Product-Name[09:17 朝]/tmp/repo> dpkg -S testing.sh
chungyi-hello: /usr/bin/testing.sh
```

## Reference
* [3.4. Creating a local Debian repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)
* [Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864)
* [Creating-a-local-Debian-repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)
* [9.1 Executable Scripts](https://www.gnu.org/software/automake/manual/automake.html#Scripts)
