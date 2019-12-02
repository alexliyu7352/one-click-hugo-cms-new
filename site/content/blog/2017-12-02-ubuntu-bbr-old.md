---
title: Debian/Ubuntu TCP BBR 改进版/增强版
m_slug: ubuntu bbr old
author: Alex
categories:
  - Linux
type: post
date: 2017-12-01T18:08:02.457Z
description: '發現一個比較好的腳本記錄一下, 轉載自https://moeclub.org/2017/06/24/278/'
featured: /images/uploads/pic03.jpg
featuredpath: date
---
**背景:**

原版的BBR对于我们来说,速度不太稳定.

通过修改BBR源码,调整参数,使其更强劲.

- - -

更新:

\[2017.07.25]

修复一个由检测gcc版本引起的不可预料的错误.

\[2017.07.15]

自动检测gcc版本,如果gcc版本大于4.9的将不会再安装gcc.

\[2017.07.12]

支持用户自行指定内核版本(需要与 -f 命令同时使用).

- - -

* **准备:**
    直接执行此命令进行开启BBR先.
    ```
  wget --no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && chmod a+x BBR.sh && bash BBR.sh -f
    ```
   注意這個命令會導致重啓

* **安裝改動版本的BBR**
   ```
  wget --no-check-certificate -qO 'BBR_POWERED.sh' 'https://moeclub.org/attachment/LinuxShell/BBR_POWERED.sh' && chmod a+x BBR_POWERED.sh && bash BBR_POWERED.sh -f v4.11.9
   ```
* **說明**
  * \-f 參數指定內核版本
  * 模块默认为开机自动加载.
  * 模块名称:tcp_bbr_powered
  * 可用 `modprobe tcp_bbr_powered`命令进行加载模块.
  * 可执行`sysctl -w net.ipv4.tcp_congestion_control=bbr_powered`使用此模块.
* **完整代碼**


```
    #!/bin/bash
 
    [ "$EUID" -ne '0' ] && echo "Error,This script must be run as root! " && exit 1
    [ $# -gt '1' ] && [ "$1" == '-f' ] && tmpKernelVer="$2" || tmpKernelVer='';
[ -z "$(dpkg -l |grep 'grub-')" ] && echo "Not found grub." && exit 1
which make >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Install make..."
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq make >/dev/null 2>&1
which make >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Error! Install make. "
exit 1
}
}
which awk >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Install awk..."
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq gawk >/dev/null 2>&1
which awk >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Error! Install awk. "
exit 1
}
}
which gcc >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Install gcc..."
DEBIAN_FRONTEND=noninteractive apt-get install -y -qq gcc >/dev/null 2>&1
which gcc >/dev/null 2>&1
[ $? -ne '0' ] && {
echo "Error! Install gcc. "
echo "Please 'apt-get update' and try again! "
exit 1
}
}
GCCVER="$(readlink `which gcc` |grep -o '[0-9].*')"
GCCVER1="$(echo $GCCVER |awk -F. '{print $1}')"
GCCVER2="$(echo $GCCVER |awk -F. '{print $2}')"
[ -n "$GCCVER1" ] && [ "$GCCVER1" -gt '4' ] && CheckGCC='0' || CheckGCC='1'
[ "$CheckGCC" == '1' ] && [ -n "$GCCVER2" ] && [ "$GCCVER2" -ge '9' ] && CheckGCC='0'
[ "$CheckGCC" == '1' ] && {
echo "The gcc version require gcc-4.9 or higher. "
echo "You can try apt-get install -y gcc-4.9 or apt-get install -y gcc-6"
echo "Please upgrade it manually! "
exit 1
}
KernelVer='';
KernelBitVer='';
MainURL='http://kernel.ubuntu.com/~kernel-ppa/mainline'
[ -n "$tmpKernelVer" ] && {
wget -qO /dev/null "$MainURL/$tmpKernelVer"
[ $? -ne '0' ] && echo 'Please input a vaild kernel version! exp: v4.11.9.' && exit 1
KernelVer="$tmpKernelVer"
}
[ -z "$tmpKernelVer" ] && {
KernelVerBIG="$(wget -qO- "$MainURL" |awk -F '/">|href="' '{print $2}' |sed '/rc/d;/^$/d' |tail -n1)"
[ -n "$KernelVerBIG" ] && KernelVer="$(wget -qO- "$MainURL" |awk -F '/">|href="' '{print $2}' |sed '/rc/d;/^$/d' |grep ''${KernelVerBIG}'' |sort -n |tail -n1)"
}
[ -z "$KernelVer" ] && echo 'Error,Get Kernel fail! ' && exit 1
ReleaseURL="$(echo -n "$MainURL/$KernelVer")"
KernelBit="$(getconf LONG_BIT)"
[ "$KernelBit" == '32' ] && KernelBitVer='i386'
[ "$KernelBit" == '64' ] && KernelBitVer='amd64'
[ -z "$KernelBitVer" ] && echo "Error! " && exit 1
HeadersFile="$(wget -qO- "$ReleaseURL" |awk -F '">|href="' '/generic.*.deb/{print $2}' |grep 'headers' |grep "$KernelBitVer" |head -n1)"
[ -n "$HeadersFile" ] && HeadersAll="$(echo "$HeadersFile" |sed 's/-generic//g;s/_'${KernelBitVer}'.deb/_all.deb/g')"
[ -z "$HeadersAll" ] && echo "Error! Get Linux Headers for All." && exit 1
echo "$HeadersFile" | grep -q "$(uname -r)"
[ $? -ne '0' ] && echo "Error! Header not be matched by Linux Kernel." && exit 1
echo -ne "Download Kernel Headers for All\n\t$HeadersAll\n"
wget -qO "$HeadersAll" "$ReleaseURL/$HeadersAll"
echo -ne "Install Kernel Headers for All\n\t$HeadersAll\n"
dpkg -i "$HeadersAll" >/dev/null 2>&1
echo -ne "Download Kernel Headers\n\t$HeadersFile\n"
wget -qO "$HeadersFile" "$ReleaseURL/$HeadersFile"
echo -ne "Install Kernel Headers\n\t$HeadersFile\n"
dpkg -i "$HeadersFile" >/dev/null 2>&1
echo -ne "Download BBR POWERED Source code\n"
[ -e ./tmp ] && rm -rf ./tmp
mkdir -p ./tmp && cd ./tmp
[ $? -eq '0' ] && {
wget --no-check-certificate -qO- 'https://moeclub.org/attachment/LinuxSoftware/bbr/tcp_bbr_powered.c.deb' >./tcp_bbr_powered.c
echo 'obj-m:=tcp_bbr_powered.o' >./Makefile
make -s -C /lib/modules/$(uname -r)/build M=`pwd` modules CC=`which gcc`
echo "Loading TCP BBR POWERED..."
[ -f ./tcp_bbr_powered.ko ] && [ -f /lib/modules/$(uname -r)/modules.dep ] && {
cp -rf ./tcp_bbr_powered.ko /lib/modules/$(uname -r)/kernel/net/ipv4
depmod -a >/dev/null 2>&1
}
modprobe tcp_bbr_powered
[ ! -f /etc/sysctl.conf ] && touch /etc/sysctl.conf
sed -i '/net.core.default_qdisc.*/d' /etc/sysctl.conf
sed -i '/net.ipv4.tcp_congestion_control.*/d' /etc/sysctl.conf
echo "net.core.default_qdisc = fq" >>/etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr_powered" >>/etc/sysctl.conf
}
lsmod |grep -q 'bbr_powered'
[ $? -eq '0' ] && {
sysctl -p >/dev/null 2>&1
echo "Finish! "
exit 0
} || {
echo "Error, Loading BBR POWERED."
exit 1
}
    
```

- - -

* **測試效果**
    * BBR默認的效果其實已經很不錯了, 但是這是針對網絡都懂並不是那麼大的情況
    * 改進版的BBR只能在部分的地區或者針對部分線路起到作用, 並不是都能解決
    * 單邊加速如果不用銳速之類的, 只能自己修改與編譯BBR模塊, 找到最合適的設置.
    * 另外, BBR加速對於NGINX還是有幫助的, 測試網頁下載速度明顯有提高
