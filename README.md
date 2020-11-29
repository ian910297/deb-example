# deb-example

This is a simple example to illustrate how to make a patch for debian package. I would divid the entire process into differe stages. Our goal is to add a shell script into the existing package and echo "Hello" during the process of `apt install`.  Then, publish the package on personal launchpad ppa.

* [Download](#Download)  
  get source file and know the basic information about debian package
* [Edit](#Edit)  
  make a patch
* [Test](#Test)  
  create a private debian repository and test by yourself at localhost
* [Publish](#Publish)  
  submit to launchpad ppa  

## Download

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

Let's start from the debian directory. Except the debian directory, the rest of files belong to source of hello package. The debian directory is used to control deb-series command.

* debian/rules  
  Set variables for makefile files, like CFLAGS, etc.
* debian/control  
  Package description/configure file, which is similar to package.json when you develope JS applications.
* debian/watch  
  The upstream location  

If you need more information, please read docs([Debian Developers' Manuals](https://www.debian.org/doc/devel-manuals#debmake-doc)) by yourself, I found that it's not really important to this exercise.

## Edit
You can use package maintainer scripts to interact with system during running `apt install`. The rule is a little like git hook script and you should name the file under a specific rule, and they should be `postrm`, `prerm`, `preinst` or`postinst`.

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
dpkg-source --commit
```

The following command is used to build a temporary version, if you want to send the debian patch to community. You must need to make a signature with your personal gpg key. In addition to this, remember to edit `debian/changelog`
* command
```
debuild -uc -us
```

## Test
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

* result
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

## Publish

Before you publish your change to your own ppa, you need to have your own gpg key in advance. This is a very good tutorial ([2. Getting Set Up](https://packaging.ubuntu.com/html/getting-set-up.html)), just follow it. If the command execute incorrectly, maybe you can follow this response([Decryption failed: No secret key using GPG](https://stackoverflow.com/a/50557399/10769057))  

After get your own pgp key and upload it to keyserver, the following are the flow of "create a ppa at launchpad"
* [Create a Launchpad Account.](https://login.launchpad.net/)  
* Activate a PPA.  
  Upload your fingerprint(gpg public key) to launchpad, and you would receive a mail with encrypted message. Please copy the meesage to your PC and decrypt with your pricate key. Then, you would get an activate url to confirm the public key at launchpad.
* Uploading your source packages.
  Use debuild to generate the release version and dput to upload files according to source.changes

* debian/changelog  
  Edit changelog before build the release package. Signature would use the last edit person's key and don't use security type.
```
hello (2.10-1build3) bionic; urgency=medium                        
                                                                        
  * Add testing.sh                                                          
                                                                           
 -- Chungyi Chi <demonic@csie.io>  Thu, 13 Feb 2020 10:53:55 -0500        
                                                                         
hello (2.10-1build3) bionic-security; urgency=medium                       
                                                                       
  * No-change rebuild for testing purposes.                                                  
                                                                                                   
 -- Marc Deslauriers <marc.deslauriers@ubuntu.com>  Thu, 13 Feb 2020 10:53:55 -0500
```

* result
```
demonic@demonic-System-Product-Name[10:18 暮]~/gitPro/deb-example/hello-2.10 (main)> automake-1.14
demonic@demonic-System-Product-Name[10:19 暮]~/gitPro/deb-example/hello-2.10 (main)> debuild -S -sd
.....
.....
.....
.....
Finished running lintian.
Now signing changes and any dsc files...
 signfile dsc hello_2.10-1build3.dsc Chungyi Chi <demonic@csie.io>

 fixup_buildinfo hello_2.10-1build3.dsc hello_2.10-1build3_source.buildinfo
 signfile buildinfo hello_2.10-1build3_source.buildinfo Chungyi Chi <demonic@csie.io>

 fixup_changes dsc hello_2.10-1build3.dsc hello_2.10-1build3_source.changes
 fixup_changes buildinfo hello_2.10-1build3_source.buildinfo hello_2.10-1build3_source.changes
 signfile changes hello_2.10-1build3_source.changes Chungyi Chi <demonic@csie.io>

Successfully signed dsc, buildinfo, changes files
demonic@demonic-System-Product-Name[10:19 暮]~/gitPro/deb-example/hello-2.10 (main)> cd ..
demonic@demonic-System-Product-Name[10:19 暮]~/gitPro/deb-example (main)> ls
chungyi-hello_2.10-1build3_amd64.deb          hello_2.10-1build3_amd64.buildinfo  hello_2.10-1build3_source.build      README.md
chungyi-hello-dbgsym_2.10-1build3_amd64.ddeb  hello_2.10-1build3_amd64.changes    hello_2.10-1build3_source.buildinfo
hello-2.10                                    hello_2.10-1build3.debian.tar.xz    hello_2.10-1build3_source.changes
hello_2.10-1build3_amd64.build                hello_2.10-1build3.dsc              hello_2.10.orig.tar.gz
```

Finally, we finish all the instructions and it's very close to the end. Use dput to upload `hello_2.10-1build3_source.changes`. Note that: you must have activiated your ppa.

* Add ppa to install chungyi-hello  
  I don't know what's wrong with https but it works correctly now~ I think that the http/https use different database to look up the key, so it should take some time to synchornize!?
```
demonic@demonic-System-Product-Name[10:25 暮]~/gitPro/deb-example (main)> sudo add-apt-repository ppa:pcsquare/ppa -k http://keyserver.ubuntu.com
```

* result ([ppa:pcsquare/ppa](https://launchpad.net/~pcsquare/+archive/ubuntu/ppa))
```
demonic@demonic-System-Product-Name[10:29 暮]~/gitPro/deb-example (main)> sudo apt install chungyi-hello 
正在讀取套件清單... 完成
正在重建相依關係          
正在讀取狀態資料... 完成
下列【新】套件將會被安裝：
  chungyi-hello
升級 0 個，新安裝 1 個，移除 0 個，有 13 個未被升級。
需要下載 51.3 kB 的套件檔。
此操作完成之後，會多佔用 275 kB 的磁碟空間。
下載:1 http://ppa.launchpad.net/pcsquare/ppa/ubuntu bionic/main amd64 chungyi-hello amd64 2.10-1build3 [51.3 kB]
取得 51.3 kB 用了 2秒 (22.1 kB/s)        
選取了原先未選的套件 chungyi-hello。
（讀取資料庫 ... 目前共安裝了 211714 個檔案和目錄。）
準備解開 .../chungyi-hello_2.10-1build3_amd64.deb ...
解開 chungyi-hello (2.10-1build3) 中...
設定 chungyi-hello (2.10-1build3) ...
this is a test from Chungyi Chi
Processing triggers for install-info (6.5.0.dfsg.1-2) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
demonic@demonic-System-Product-Name[10:29 暮]~/gitPro/deb-example (main)> dpkg -S testing.sh; testing.sh
chungyi-hello: /usr/bin/testing.sh
this is a test from Chungyi Chi
demonic@demonic-System-Product-Name[10:30 暮]~/gitPro/deb-example (main)> cat /etc/apt/sources.list.d/pcsquare-ubuntu-ppa-bionic.list
deb http://ppa.launchpad.net/pcsquare/ppa/ubuntu bionic main
# deb-src http://ppa.launchpad.net/pcsquare/ppa/ubuntu bionic main
```

It is expected that you could execute each step correctly, if there is any question, please feel free to contact me.

## Reference
* [3.4. Creating a local Debian repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)
* [Create your own custom and authenticated APT repository](https://medium.com/sqooba/create-your-own-custom-and-authenticated-apt-repository-1e4a4cf0b864)
* [Creating-a-local-Debian-repository](https://blog.heckel.io/2015/10/18/how-to-create-debian-package-and-debian-repository/#Creating-a-local-Debian-repository)
* [9.1 Executable Scripts](https://www.gnu.org/software/automake/manual/automake.html#Scripts)
* [2. Getting Set Up](https://packaging.ubuntu.com/html/getting-set-up.html)
* [How do I create a PPA?](https://askubuntu.com/a/71516)

