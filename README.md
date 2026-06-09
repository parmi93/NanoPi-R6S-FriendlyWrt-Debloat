# NanoPi R6S FriendlyWrt 25.12.2 Debloat

**NanoPi R6S FriendlyWrt 25.12.2 Debloat Guide.** A script and guide to identify and safely remove all non-official/non-stock OpenWrt apk packages installed by FriendlyWrt on the NanoPi R6S. Restore a cleaner, leaner OpenWrt environment while keeping core system functionality.

> [!NOTE]
> This guide is primarily written for my own future reference, in case I need to perform this procedure again.

## Comparing OpenWrt and FriendlyWrt after a clean installation

### Comparison of Memory Usage
Comparison of the number of pre-installed packages and RAM usage after a clean installation of FriendlyWrt and OpenWrt
|                                      | No. APKs | RAM Usage |
|--------------------------------------|----------|-----------|
| OpenWrt 25.12.2                      | 136      | ~101 MiB  |
| Friendly 25.12.2 <br> before debloat | 1059     | ~364 MiB  |
| Friendly 25.12.2 <br> after debloat  | 141      | ~278 MiB  |

While the debloating process successfully reduces RAM usage by about **86 MiB** by **uninstalling 919 apks**, the consumption is still roughly **177 MiB higher** than OpenWrt's baseline. I believe this difference is due to the extra features baked into FriendlyWrt BSP Kernel. Specifically, FriendlyWrt supports hardware-accelerated video transcoding via the **Rockchip VPU (RKMPP)**, includes **native NTFS filesystem support**, and provides shell support via HDMI display. These, along with other added functionalities, likely account for the increased RAM footprint.

### Comparison of CPU clocks
Comparison of the maximum clock speeds achievable by the different CPU cores.
<table>
    <thead>
        <tr>
            <th></th>
            <th colspan="4">Quad-core ARM Cortex-A55</th>
            <th colspan="4">Quad-core ARM Cortex-A76</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td>Core0</td>
            <td>Core1</td>
            <td>Core2</td>
            <td>Core3</td>
            <td>Core4</td>
            <td>Core5</td>
            <td>Core6</td>
            <td>Core7</td>
        </tr>
        <tr>
            <td>OpenWrt 25.12.2</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>2400 MHz</td>
            <td>2400 MHz</td>
            <td>2400 MHz</td>
            <td>2400 MHz</td>
        </tr>
        <tr>
            <td>FriendlyWrt 25.12.2</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>1800 MHz</td>
            <td>2304 MHz</td>
            <td>2304 MHz</td>
            <td>2256 MHz</td>
            <td>2256 MHz</td>
        </tr>
    </tbody>
</table>

The Cortex-A76 cores running FriendlyWrt reach a maximum clock frequency that is approximately 100 to 150 MHz lower than with OpenWrt. This behavior is intentional in FriendlyWrt, which uses an advanced frequency scaling method that takes into account factors such as silicon quality, allowing it to achieve maximum CPU performance without sacrificing system stability.

OpenWrt, or more precisely the upstream Linux kernel, does not take these parameters into account, allowing the CPU to reach its theoretical maximum clock frequencies. Apparently, there are plans to implement the same mechanism in the official Linux kernel as well.

See this article for more details: https://www.collabora.com/news-and-blog/news-and-events/rockchip-rk3588-upstream-support-progress-future-plans.html

That said, personally I have never encountered any issues with OpenWrt while running the `scaling_governor` set to `performance` on all cores for an entire year.

### OpenSSL performance comparison
The Rockchip RK3588S comes equipped with 2 dedicated cipher engines on SoC and the ARMv8 Cryptography Extensions on all eight cores. I'm not clear on which of these hardware accelerators FriendlyWrt actually uses, or if they even rely on specific apk packages to run. So, to check for any performance impact, I ran OpenSSL benchmarks both before and after the debloat.  
Specifically, I executed the benchmarks suggested on the OpenWrt Wiki page: https://openwrt.org/docs/guide-user/perf_and_log/benchmark.openssl  

<details>
  <summary><font size="4" color="darkred"><b><code>openssl_multi_core_benchmark.sh</code></b></font></summary>
    
  ```bash
  CPUCoreCount=$(grep -c ^processor /proc/cpuinfo); W='\033[0;37m' ; G='\033[0;32m' ; 
  #CPUCoreCount=2
  openssl speed -provider default -provider legacy -multi $CPUCoreCount md5 sha1 sha256 sha512 des des-ede3 aes-128-cbc aes-192-cbc aes-256-cbc rsa2048 dsa2048 | tee /tmp/sslspeedm
  . /etc/os-release; echo -e "\n\n${G}"\
  $(awk -v "rev=${BUILD_ID%%-*}" -v "FS=: " -v "ORS=" -v "CC=$CPUCoreCount" -e 'BEGIN \
      {print "| " rev " -multi " CC " "} !a[$0]++ && /(Processor|Hardware|machine|cpu model|system type|BogoMIPS)/ \
      {print "| " $2 " "}' /proc/cpuinfo) \
  $(awk -v "ORS=" -e '$1 ~ /OpenSSL/ {print "| " $2 " |"} $1 ~ /version/ {print "| " $2 " |"} $1 ~ /(md5|sha)/ \
      {print "  " $5 " | "} $1 ~ /(des|aes)/ {b = b "  " $6 " |"} $1 ~ /(rsa|dsa)/ \
      {print b " " $6 " | " $7 " | ";b=""}' /tmp/sslspeedm \
  | sed -e 's/\.\(..\)k/\10/g') ;  echo -e "\n${W}"
  ```
</details>

Benchmark with openssl v3.5.6:
|                                         | Performance <br> (compared to OpenWrt) | MD5            | SHA-1          | SHA-256        | SHA-512        | DES          | 3DES         | AES-128         | AES-192         | AES-256        | RSA Sign | RSA Verify | DSA Sign | DSA Verify |
|-----------------------------------------|----------------------------------------|----------------|----------------|----------------|----------------|--------------|--------------|-----------------|-----------------|----------------|----------|------------|----------|------------|
| OpenWrt 25.12.2                         |                                        | 2,505,255,590  | 5,756,167,510  | 5,709,989,550  | 1,802,351,620  | 328,081,410  | 117,661,700  | 12,867,564,890  | 10,269,518,510  | 8,666,895,700  | 1,637.4  | 64,514.2   | 4,664.4  | 5,057.4    |
| FriendlyWrt 25.12.2 <br> before debloat | -2.83%                                 | 2,341,796,520  | 5,294,303,570  | 5,290,237,270  | 1,710,886,910  | 326,063,450  | 116,979,030  | 12,710,502,400  | 10,148,883,110  | 8,563,605,500  | 1,612.8  | 63,713.9   | 4,610.7  | 4,997.0    |
| FriendlyWrt 25.12.2 <br> after debloat  | -0.39%                                 | 2,514,027,180  | 5,773,312,340  | 5,751,678,980  | 1,800,385,540  | 327,980,370  | 117,606,250  | 12,777,660,420  | 10,201,470,290  | 8,612,173,140  | 1,620.8  | 64,095.4   | 4,591.3  | 5,006.5    |

<details>
  <summary><font size="4" color="darkred"><b><code>openssl_single_core_benchmark.sh</code></b></font></summary>
  
  ```bash
  W='\033[0;37m' ; G='\033[0;32m' 
  taskset -c 5 openssl speed -provider default -provider legacy md5 sha1 sha256 sha512 des des-ede3 aes-128-cbc aes-192-cbc aes-256-cbc rsa2048 dsa2048 | tee /tmp/sslspeed
  . /etc/os-release; echo -e "\n\n${G}"\
  $(awk -v "rev=${BUILD_ID%%-*}" -v "FS=: " -v "ORS=" -e 'BEGIN \
      {print "| " rev " single-thread "} !a[$0]++ && /(Processor|Hardware|machine|cpu model|system type|BogoMIPS)/ \
	  {print "| " $2 " "}' /proc/cpuinfo) \
  $(awk -v "ORS=" -e '$1 ~ /OpenSSL/ {print "| " $2 " |"} $1 ~ /version/ {print "| " $2 " |"} $1 ~ /(md5|sha)/ \
      {print "  " $5 " | "} $1 ~ /(des|aes)/ {b = b "  " $6 " |"} $1 ~ /(rsa|dsa)/ \
      {print b " " $6 " | " $7 " | ";b=""}' /tmp/sslspeed \
  | sed -e 's/\.\(..\)k/\10/g') ;  echo -e "\n${W}"
  ```
</details>

Benchmark with openssl v3.5.6:
|                                         | Performance <br> (compared to OpenWrt) | MD5         | SHA-1       | SHA-256       | SHA-512     | DES        | 3DES       | AES-128       | AES-192       | AES-256       | RSA Sign | RSA Verify | DSA Sign | DSA Verify |
|-----------------------------------------|----------------------------------------|-------------|-------------|---------------|-------------|------------|------------|---------------|---------------|---------------|----------|------------|----------|------------|
| OpenWrt 25.12.2                         |                                        | 422,395,900 | 988,246,990 |   998,605,140 | 305,033,220 | 59,561,300 | 21,678,760 | 1,822,285,820 | 1,520,061,100 | 1,303,369,050 | 247.7    | 9,764.1    | 706.9    | 742.6      |
| FriendlyWrt 25.12.2 <br> before debloat | -1.43%                                 | 402,104,320 | 923,163,990 |   935,078,570 | 294,182,910 | 59,678,720 | 21,719,720 | 1,825,472,510 | 1,522,832,730 | 1,305,578,150 | 247.8    | 9,766.1    | 709.0    | 753.6      |
| FriendlyWrt 25.12.2 <br> after debloat  | +0.88%                                 | 426,299,730 | 991,142,910 | 1,005,902,510 | 306,829,310 | 59,886,250 | 21,790,720 | 1,831,652,010 | 1,527,747,930 | 1,309,862,570 | 248.8    | 9,800.9    | 710.7    | 779.7      |

The benchmark results are almost identical both before and after the debloat process, with a slight improvement observed after debloating. It's worth noting that the `openssl_multi_core_benchmark.sh` benchmark recorded a performance decrease of about -1% when run on FriendlyWrt compared to OpenWrt. This is likely attributable to a lower clock speed on the four Cortex-A76 cores under FriendlyWrt.

In any case, I can confirm that the debloat procedure did not have a negative impact on OpenSSL performance.

## [BEFORE DEBLOAT] Switch back to the OpenWrt shell

> [!CAUTION]
> After the cleanup, the `bash` shell (SSH) will be uninstalled, meaning you will lose the ability to interact with FriendlyWrt.  
> This step ensures that your shell remains functional after the cleanup.

FriendlyWrt uses the `bash` package shell by default (which will be uninstalled during cleanup). To ensure your shell (SSH) continues to function, you must switch to the official OpenWrt shell (`ash`) provided by the official `busybox` package.  
Edit the `/etc/passwd` file:
1. Open th `passwd` file:
    ```bash
    vim /etc/passwd
    ```
2. Find a line that contain `...bin/bash` and change it to `...bin/ash`
3. Save and close with `:wq`

## [DEBLOAT] Uninstall unnecessary `apk` packages
Here are the steps to follow to uninstall all unnecessary packages from FriendlyWrt 25.12.2.
1. Create the uninstall script:
    ```bash
    vim apk_uninstall.sh
    ```
2. Paste this massive command:
    ```bash
    apk del coreutils-nohup coreutils-stat coreutils-du btrfs-progs file coreutils-env coreutils-cp cfdisk coreutils-ln bind-dig iwlwifi-firmware-ax210 coreutils-base64 coreutils-mknod iptables-mod-extra bspatch coreutils-tail coreutils-chown iptables-mod-cluster iptables-mod-hashlimit coreutils-sha256sum iptables-mod-conntrack-extra coreutils-tr coreutils-chgrp findutils coreutils-pwd coreutils-cat iptables-mod-checksum blockd coreutils-df coreutils-readlink collectd-mod-df coreutils-echo coreutils-true coreutils-od coreutils-cut coreutils-sync dnsmasq-full coreutils-seq coreutils-chmod coreutils-chroot coreutils-sha1sum coreutils-dd blkdiscard coreutils-expr coreutils-rmdir blkid ca-certificates collectd-mod-thermal coreutils-md5sum coreutils-test coreutils-date coreutils-dirname curl coreutils-false kmod-ipsec4 iwlwifi-firmware-ax200 coreutils-sleep coreutils-id diffutils batctl-default collectd-mod-uptime bsdiff ipset coreutils-nice coreutils-unlink coreutils-truncate coreutils-mv collectd-mod-wireless coreutils-ls hwclock collectd-mod-cpufreq kmod bash iptables-mod-filter coreutils-nproc hostapd-openssl gzip coreutils-basename coremark ethtool coreutils-head getopt coreutils-mktemp iptables-mod-iprange extract iptables-mod-tee iptables-mod-rpfilter coreutils-rm kmod-brcmfmac coreutils-mkfifo coreutils-mkdir coreutils-printf iptables-mod-nflog kmod-nft-socket luci-i18n-commands-bg kmod-tun kmod-macvlan grep iperf3 iptables-mod-tproxy coreutils-uname coreutils-yes iptables-mod-conntrack-label git-http coreutils-uniq coreutils-realpath iperf coreutils-tee iptables-mod-trace kmod-dummy iptables-mod-u32 iwinfo fdisk luci-i18n-adblock-pl coreutils-wc luci-i18n-commands-ca kmod-usb-net-rtl8152 coreutils-timeout fstrim luci-i18n-adblock-it libopenssl-devcrypto iptables-mod-ipsec kmod-nft-tproxy kmod-nft-bridge coreutils-touch iptables-mod-nfqueue iptables-mod-led kmod-ikconfig iptables-mod-nat-extra coreutils-tty iptables-mod-physdev kmod-nf-nathelper-extra luci-i18n-firewall-pt-br libuci-lua kmod-nf-ipvs luci-i18n-commands-it kmod-crypto-gcm luci-i18n-commands-de luci-i18n-adblock-de libiwinfo-lua luci-i18n-commands-zh-tw luci-i18n-adblock-el luci-i18n-adblock-hu luci-i18n-package-manager-ca luci-i18n-firewall-hu kmod-sched-connmark luci-i18n-package-manager-de luci-i18n-base-pt-br luci-i18n-package-manager-pl luci-i18n-commands-cs kmod-nft-netdev luci-i18n-samba4-ms kmod-nf-nathelper luci-i18n-adblock-cs luci-i18n-adblock-ms luci-i18n-ddns-hu luci-i18n-samba4-fr luci-i18n-package-manager-zh-cn luci-i18n-adblock-ro luci-i18n-ddns-ro luci-i18n-firewall-fr luci-i18n-samba4-sk luci-i18n-minidlna-ru luci-i18n-ddns-ko luci-i18n-nlbwmon-vi luci-i18n-firewall-pt luci-i18n-statistics-mr luci-i18n-package-manager-el luci-i18n-package-manager-cs luci-i18n-adblock-sv luci-i18n-ddns-bg luci-i18n-statistics-el luci-i18n-commands-hi luci-i18n-adblock-fr luci-i18n-ddns-he luci-i18n-ddns-hi luci-i18n-minidlna-zh-tw luci-i18n-hd-idle-bg luci-i18n-adblock-hi luci-i18n-ddns-fr luci-i18n-firewall-ja luci-i18n-commands-ko luci-i18n-package-manager-pt-br luci-i18n-aria2-vi luci-i18n-minidlna-bg luci-i18n-samba4-es luci-i18n-firewall-ca luci-i18n-adblock-ru luci-i18n-firewall-bg luci-i18n-firewall-pl luci-i18n-nlbwmon-bg libfido2-1 luci-i18n-minidlna-pl luci-i18n-aria2-zh-cn luci-i18n-base-pt luci-i18n-aria2-ja luci-i18n-nlbwmon-ms luci-i18n-nlbwmon-zh-cn luci-i18n-package-manager-uk luci-i18n-aria2-mr luci-i18n-nlbwmon-pt-br luci-i18n-ddns-vi luci-i18n-firewall-sk luci-i18n-firewall-uk luci-i18n-ddns-uk luci-i18n-firewall-vi luci-i18n-aria2-sv luci-i18n-statistics-cs luci-i18n-ddns-ca luci-i18n-ddns-zh-cn luci-i18n-package-manager-ko luci-i18n-hd-idle-tr luci-i18n-samba4-pt-br luci-i18n-ddns-tr luci-i18n-hd-idle-ro luci-i18n-nlbwmon-hu luci-i18n-base-ko luci-i18n-base-hi libseccomp luci-i18n-aria2-ru luci-i18n-samba4-mr luci-i18n-samba4-ro luci-i18n-aria2-ms luci-i18n-aria2-ko luci-i18n-commands-pt-br luci-i18n-package-manager-es luci-i18n-minidlna-mr luci-i18n-commands-hu luci-i18n-adblock-mr luci-i18n-minidlna-ro luci-i18n-aria2-hi luci-i18n-nlbwmon-ru luci-i18n-commands-ru luci-i18n-ddns-it luci-i18n-aria2-pl luci-i18n-aria2-he luci-i18n-base-he luci-i18n-commands-mr luci-i18n-commands-fr luci-i18n-firewall-he luci-i18n-nlbwmon-zh-tw luci-i18n-nlbwmon-ja luci-i18n-firewall-tr luci-i18n-firewall-hi luci-i18n-firewall-ms luci-i18n-adblock-bg luci-i18n-samba4-ru luci-i18n-package-manager-ru luci-i18n-base-ca luci-i18n-commands-ro luci-i18n-aria2-fr luci-i18n-hd-idle-zh-tw luci-i18n-package-manager-pt luci-i18n-package-manager-hi luci-i18n-ddns-zh-tw luci-i18n-commands-pl luci-i18n-package-manager-vi luci-i18n-aria2-sk luci-i18n-package-manager-tr luci-i18n-base-hu luci-i18n-adblock-es luci-i18n-commands-he luci-i18n-base-mr luci-i18n-statistics-es luci-i18n-samba4-pl luci-i18n-adblock-pt luci-i18n-commands-ms luci-i18n-upnp-sv luci-i18n-upnp-el luci-i18n-aria2-es luci-i18n-firewall-zh-tw luci-i18n-statistics-zh-tw luci-i18n-commands-es luci-i18n-package-manager-sv luci-i18n-commands-pt luci-i18n-statistics-de luci-i18n-base-fr luci-i18n-samba4-cs luci-i18n-package-manager-he luci-i18n-package-manager-ja luci-i18n-ddns-sk luci-i18n-ttyd-ca luci-i18n-samba4-ca luci-i18n-statistics-tr luci-i18n-commands-el luci-i18n-minidlna-pt luci-i18n-hd-idle-vi luci-i18n-firewall-ko luci-i18n-base-it luci-i18n-adblock-vi luci-i18n-package-manager-zh-tw luci-i18n-package-manager-sk luci-i18n-sqm-hu luci-i18n-aria2-hu luci-i18n-adblock-ja luci-i18n-commands-zh-cn luci-i18n-firewall-ru luci-i18n-base-zh-tw luci-i18n-statistics-fr luci-i18n-adblock-zh-cn luci-i18n-minidlna-pt-br luci-i18n-statistics-ms luci-i18n-base-tr luci-i18n-nlbwmon-sk luci-i18n-base-cs luci-i18n-package-manager-hu luci-i18n-aria2-cs luci-i18n-nlbwmon-fr luci-i18n-commands-sv luci-i18n-minidlna-ko luci-i18n-minidlna-ja luci-i18n-firewall-sv luci-i18n-aria2-it luci-i18n-base-bg luci-i18n-commands-uk luci-i18n-nlbwmon-pl luci-i18n-package-manager-bg luci-i18n-statistics-sk luci-i18n-nlbwmon-ca luci-i18n-aria2-uk luci-i18n-aria2-pt-br luci-i18n-adblock-pt-br kmod-wireguard luci-i18n-adblock-uk luci-i18n-firewall-it kmod-veth luci-i18n-statistics-ro luci-i18n-ddns-sv luci-i18n-statistics-pl luci-i18n-nlbwmon-ko luci-i18n-sqm-es luci-i18n-ddns-el luci-i18n-aria2-tr luci-i18n-nlbwmon-it luci-i18n-aria2-pt luci-i18n-package-manager-fr luci-i18n-sqm-sv luci-i18n-firewall-ro luci-i18n-base-zh-cn luci-i18n-adblock-ko luci-i18n-adblock-sk luci-i18n-adblock-ca luci-i18n-statistics-vi luci-i18n-ddns-cs luci-i18n-ttyd-el luci-i18n-samba4-bg luci-i18n-adblock-he luci-i18n-hd-idle-uk luci-i18n-aria2-ca luci-i18n-upnp-ms luci-i18n-minidlna-cs luci-i18n-aria2-de libxml2-16 luci-i18n-adblock-tr luci-i18n-statistics-uk luci-i18n-hd-idle-ru luci-i18n-base-ro luci-i18n-ddns-es luci-i18n-upnp-ca luci-i18n-sqm-ca luci-i18n-commands-sk luci-i18n-ddns-de luci-i18n-aria2-zh-tw luci-i18n-commands-vi luci-i18n-adblock-zh-tw luci-i18n-aria2-el luci-i18n-firewall-zh-cn luci-i18n-smartdns-sv luci-i18n-statistics-ja luci-i18n-base-el luci-i18n-sqm-sk luci-i18n-samba4-ko luci-i18n-statistics-sv luci-i18n-hd-idle-pt-br luci-i18n-minidlna-el luci-i18n-nlbwmon-de luci-i18n-nlbwmon-hi luci-i18n-base-ru luci-i18n-aria2-ro luci-i18n-hd-idle-es luci-i18n-package-manager-ms luci-i18n-commands-tr luci-i18n-ttyd-pl luci-i18n-ddns-ms luci-i18n-hd-idle-zh-cn luci-i18n-statistics-ru luci-i18n-nlbwmon-sv luci-i18n-minidlna-hu luci-i18n-commands-ja luci-i18n-samba4-ja luci-i18n-hd-idle-ja luci-i18n-ddns-pt-br luci-i18n-firewall-mr luci-i18n-ttyd-mr luci-i18n-base-sv luci-i18n-samba4-it luci-i18n-ttyd-hi luci-i18n-ttyd-ko luci-i18n-minidlna-it luci-i18n-statistics-ko luci-i18n-aria2-bg luci-i18n-samba4-pt luci-i18n-base-ja luci-i18n-ttyd-pt-br luci-i18n-sqm-it luci-i18n-package-manager-mr luci-i18n-nlbwmon-uk luci-i18n-ddns-pt luci-i18n-package-manager-it luci-i18n-base-es luci-i18n-minidlna-zh-cn luci-i18n-minidlna-ca luci-i18n-samba4-he luci-i18n-base-pl luci-i18n-nlbwmon-he luci-i18n-minidlna-ms luci-i18n-minidlna-fr luci-i18n-sqm-zh-cn luci-i18n-ddns-ru luci-i18n-ddns-pl luci-i18n-samba4-el luci-i18n-base-vi luci-i18n-smartdns-hi luci-i18n-upnp-hi luci-i18n-minidlna-vi luci-i18n-upnp-ko luci-i18n-hd-idle-ca luci-i18n-ttyd-pt luci-i18n-ttyd-ms luci-i18n-upnp-cs luci-i18n-nlbwmon-ro luci-i18n-ttyd-he luci-i18n-system-zh-cn luci-i18n-upnp-sk luci-i18n-hd-idle-ko luci-i18n-statistics-he luci-i18n-statistics-it luci-i18n-statistics-hi luci-i18n-base-de luci-i18n-smartdns-cs luci-i18n-smartdns-uk luci-i18n-upnp-de luci-i18n-minidlna-hi luci-i18n-base-sk luci-i18n-emmc-tools-zh-cn luci-i18n-samba4-hu luci-i18n-base-uk luci-i18n-nlbwmon-mr ppp-mod-pptp luci-i18n-hd-idle-mr luci-i18n-nlbwmon-es luci-i18n-upnp-ja luci-i18n-hd-idle-de luci-i18n-ddns-mr luci-i18n-hd-idle-hi luci-i18n-sqm-fr luci-i18n-sqm-el luci-i18n-hd-idle-it luci-i18n-upnp-ru luci-i18n-hd-idle-ms luci-i18n-statistics-zh-cn luci-i18n-upnp-hu luci-i18n-sqm-zh-tw luci-i18n-upnp-es luci-i18n-samba4-sv luci-i18n-smartdns-ms luci-i18n-smartdns-tr luci-i18n-ttyd-de luci-i18n-base-ms luci-i18n-sqm-tr luci-i18n-sqm-bg luci-i18n-ddns-ja luci-i18n-package-manager-ro luci-i18n-smartdns-mr luci-i18n-nlbwmon-tr luci-i18n-samba4-tr luci-i18n-sqm-mr luci-i18n-samba4-hi luci-i18n-upnp-fr luci-i18n-hd-idle-pt luci-i18n-watchcat-es luci-i18n-hd-idle-sv luci-i18n-samba4-de luci-i18n-hd-idle-pl luci-i18n-hd-idle-sk tini luci-i18n-statistics-pt-br luci-i18n-smartdns-zh-cn luci-proto-qmi luci-i18n-ttyd-zh-tw luci-i18n-nlbwmon-el luci-i18n-smartdns-ro luci-i18n-ttyd-fr luci-i18n-firewall-de luci-i18n-ttyd-cs luci-i18n-statistics-ca luci-i18n-upnp-he luci-i18n-hd-idle-cs luci-i18n-nlbwmon-cs luci-i18n-hd-idle-el luci-i18n-nlbwmon-pt luci-i18n-ttyd-es luci-i18n-samba4-zh-cn luci-i18n-samba4-vi luci-i18n-firewall-el luci-i18n-minidlna-sk luci-i18n-ttyd-vi luci-i18n-smartdns-hu luci-i18n-ttyd-zh-cn luci-i18n-sqm-vi luci-i18n-smartdns-de ppp-mod-pppoa luci-i18n-sqm-hi luci-i18n-statistics-hu luci-i18n-sqm-uk luci-i18n-sqm-ru luci-i18n-smartdns-pl luci-i18n-hd-idle-he luci-i18n-hd-idle-fr luci-i18n-samba4-uk luci-i18n-upnp-zh-cn luci-i18n-smartdns-he luci-i18n-smartdns-sk luci-i18n-sqm-ja luci-i18n-minidlna-es luci-i18n-minidlna-sv luci-i18n-sqm-he luci-i18n-smartdns-es luci-i18n-sqm-ko mt792x-firmware luci-i18n-statistics-bg luci-i18n-upnp-mr luci-i18n-smartdns-pt luci-i18n-upnp-tr luci-i18n-smartdns-bg luci-i18n-hd-idle-hu luci-i18n-upnp-bg luci-i18n-upnp-uk luci-i18n-firewall-es luci-i18n-sqm-de luci-i18n-smartdns-ko luci-i18n-watchcat-sv luci-i18n-smartdns-vi luci-i18n-watchcat-el luci-i18n-firewall-cs luci-i18n-upnp-vi luci-i18n-samba4-zh-tw luci-i18n-smartdns-ca luci-i18n-sqm-pt luci-i18n-smartdns-ja luci-i18n-smartdns-fr luci-i18n-sqm-ro luci-theme-openwrt-2020 luci-i18n-sqm-cs sfdisk px5g-wolfssl luci-i18n-smartdns-zh-tw luci-i18n-minidlna-uk luci-i18n-watchcat-he luci-i18n-watchcat-de luci-i18n-upnp-it luci-i18n-ttyd-sv ppp-mod-radius luci-i18n-watchcat-cs luci-i18n-smartdns-ru luci-i18n-ttyd-uk luci-i18n-ttyd-bg luci-i18n-watchcat-ca luci-i18n-smartdns-el luci-i18n-upnp-ro luci-i18n-ttyd-ja luci-i18n-upnp-pt-br wsdd2 luci-i18n-statistics-pt ppp-mod-pppol2tp luci-i18n-sqm-ms pciutils luci-i18n-sqm-pl luci-i18n-watchcat-tr luci-i18n-sqm-pt-br luci-i18n-watchcat-ko luci-i18n-minidlna-tr luci-i18n-smartdns-it luci-i18n-minidlna-he luci-i18n-watchcat-fr wpa-supplicant-openssl luci-i18n-smartdns-pt-br luci-i18n-watchcat-hi luci-i18n-watchcat-vi luci-i18n-watchcat-ru luci-i18n-watchcat-zh-tw luci-i18n-minidlna-de luci-i18n-ttyd-sk luci-ssl-openssl luci-i18n-ttyd-hu luci-i18n-upnp-pl luci-i18n-watchcat-zh-cn umbim luci-i18n-upnp-zh-tw rtl8822be-firmware parted resize2fs luci-i18n-watchcat-ja luci-lib-iptparser luci-i18n-ttyd-ro sed luci-i18n-upnp-pt luci-i18n-ttyd-tr luci-i18n-watchcat-pl luci-i18n-watchcat-hu luci-i18n-ttyd-it ucode-mod-socket luci-i18n-watchcat-bg luci-i18n-ttyd-ru mount-utils unrar tune2fs luci-i18n-watchcat-ro luci-i18n-watchcat-pt luci-i18n-watchcat-uk mt76x2-firmware luci-i18n-watchcat-ms tar luci-i18n-watchcat-it usbutils luci-i18n-watchcat-mr luci-i18n-watchcat-pt-br rtl8822ce-firmware luci-lib-ipkg strace luci-i18n-watchcat-sk luci-theme-material usb-modeswitch-official triggerhappy luci-proto-3g ppp-mod-passwordfd openssh-sftp-server openssh-client-utils qrencode vsftpd psmisc qmi-utils vim-full unzip uuidgen kmod-crypto-lib-chacha20poly1305 kmod-ipt-conntrack-extra hostapd-common kmod-asn1-decoder kmod-ipt-ipsec kmod-crypto-ghash kmod-ipt-led kmod-ipt-debug kmod-ipt-hashlimit findutils-find kmod-ipt-filter kmod-crypto-ctr bind-libs findutils-xargs comgt kmod-brcmutil bzip2 kmod-cfg80211 kmod-ipt-extra kmod-ipt-iprange kmod-ipsec kmod-cryptodev brcmfmac-firmware-usb git kmod-ipt-conntrack-label kmod-fs-btrfs libpci findutils-locate kmod-mppe libwolfsslcpu-crypto5.8.4.e624513f kmod-ipt-cluster kmod-pppoa kmod-ipt-nat-extra kmod-ipt-tproxy kmod-crypto-lib-curve25519 kmod-ipt-checksum block-mount libqmi kmod-ipt-nfqueue kmod-ipt-physdev kmod-iptunnel4 kmod-fs-autofs4 kmod-ipt-rpfilter kmod-pppol2tp libfdisk1 kmod-pptp kmod-nf-socket libipset13 libmagic kmod-ipt-nflog libkmod liblzo2 openssh-keygen kmod-ipt-tee libparted luci-app-hd-idle kmod-ipt-u32 kmod-mmc libcurl4 luci-app-nlbwmon kmod-usb-net-cdc-mbim libcbor0 usbids libiperf3 luci-app-samba4 libacl openssl-util luci-app-watchcat luci-app-smartdns libextractor vim-runtime libqrencode r8152-firmware openssh-client linux-atm luci-app-upnp libudev-zero luci-app-ddns luci-app-adblock libusb-1.0-0 luci-app-statistics luci-app-aria2 luci-app-ttyd luci-app-minidlna luci-app-sqm pciids luci-app-commands xz uqmi kmod-l2tp kmod-atm ddns-scripts kmod-crypto-deflate kmod-crypto-kpp aria2 collectd-mod-iwinfo collectd-mod-cpu collectd-mod-load kmod-br-netfilter kmod-nf-tproxy kmod-crypto-des collectd-mod-network libopenldap collectd-mod-interface kmod-nfnetlink-queue kmod-lib-zstd kmod-lib-raid6 libnghttp3 kmod-nfnetlink-log kmod-crypto-echainiv iw kmod-crypto-authenc libevdev libdevmapper collectd-mod-rrdtool kmod-ipt-conntrack kmod-crypto-gf128 kmod-usb-net-qmi-wwan hd-idle kmod-ipt-ipset kmod-crypto-ecb kmod-nf-dup-inet kmod-crypto-arc4 kmod-crypto-lib-chacha20 kmod-lib-lzo kmod-crypto-md5 kmod-crypto-blake2b kmod-lib-textsearch kmod-crypto-seqiv kmod-crypto-lib-poly1305 kmod-ipt-raw kmod-nf-conncount adblock kmod-lib-xor kmod-crypto-xxhash chat libngtcp2 collectd-mod-memory kmod-ipt-raw6 smartdns libqrtr-glib kmod-usb-net-cdc-ncm kmod-ipt-nat liblzma kmod-gre libidn2 luci-lib-chartjs libmbim miniupnpd-nftables liburcu libzstd libssh2-1 wifi-scripts wwan sqm-scripts lsblk nlbwmon xz-utils minidlna rrdtool1 wireless-regdb libnghttp2-14 ttyd samba4-server watchcat collectd kmod-udptunnel4 kmod-crypto-user glib2 kmod-crypto-geniv aria2-openssl iptables-mod-ipopt iptables-nft kmod-lib-zlib-deflate kmod-dm kmod-iptunnel kmod-udptunnel6 ddns-scripts-services coreutils-sort kmod-usb-net-cdc-ether kmod-usb-wdm kmod-ip6tables kmod-crypto-acompress kmod-lib-zlib-inflate gawk kmod-sched-cake ip-full kmod-lib-xxhash kmod-ifb libffmpeg-audio-dec libexif libcap-ng libid3tag libflac libmount1 libsasl2 tc-tiny libvorbis librrd1 ucode-mod-digest libjpeg-turbo libwebsockets-full libunistring libstdcpp6 samba4-libs ucode-mod-nl80211 libsqlite3-0 resolveip ucode-mod-rtnl attr libgnutls kmod-keys-encrypted coreutils kmod-ipt-ipopt libbpf1 libopenssl-legacy libavahi-client libuv1 kmod-dax kmod-sched-core libreadline8 kmod-nf-ipt6 liburing kmod-usb-net libpopt0 libogg0 libffi libltdl7 libcap libtirpc libbz2-1.0 libtasn1 xtables-nft kmod-ipt-core libelf1 avahi-dbus-daemon libatomic1 libiptext6-0 kmod-crypto-rng kmod-keys-trusted libiptext-nft0 libattr kmod-crypto-cbc kmod-nft-compat libopenssl-conf libiptext0 kmod-usb-core libncurses6 kmod-crypto-sha256 libnettle8 kmod-tpm kmod-crypto-sha3 kmod-crypto-sha1 kmod-usb-common kmod-nf-ipt libavahi-dbus-support kmod-crypto-sha512 kmod-nls-base kmod-crypto-hmac libgmp10 libxtables12 libdaemon terminfo dbus kmod-random-core libnetfilter-conntrack3 kmod-crypto-manager libexpat libnfnetlink0 kmod-crypto-aead libdbus kmod-nf-conntrack-netlink kmod-crypto-null wget-ssl
    ```
3. Save and close with `:wq`
4. Set execution permissions: 
    ```bash
    chmod +x apk_uninstall.sh
    ```
5. Excecute to uninstall:
    ```bash
    ./apk_uninstall.sh
    ```

### Some packages are required
Three packages must not be uninstalled:

- `luci-app-emmc-tools` and `luci-compat`: These packages are not included in standard OpenWrt, but they are required for the `System` -> `eMMC Tools` to work correctly, also, if you uninstall `luci-app-emmc-tools`, you won't be able to get it back, as it is missing from the package repository.
- `libustream-openssl20201210`: FriendlyWrt replaced Mbed TLS with OpenSSL, which makes sense because Mbed TLS is designed for truly low-end platforms with limited memory at the expense of speed performance. OpenSSL, while using more memory (which is not an issue with the NanoPi R6S, as we have 8GiB of RAM), offers higher speed performance. Therefore, I decided to keep `libustream-openssl20201210` instead of reverting to the official `libustream-mbedtls20201210` package.

### Install some essentials `apk`s
Install `dnsmasq`: This is the official package found in OpenWrt. FriendlyWrt replaced it with the full version, `dnsmasq-full` (which you just uninstalled), so by installing `dnsmasq` we revert back to the original one.

```bash
apk add dnsmasq
```

> [!NOTE]
> If this command fails, make sure that the NanoPi's date/time are correct.

## [OPTIONAL] Install some usefull package

I found a few particularly useful packages:
- `htop`: Allows real-time monitoring of system resource usage (CPU, memory, temperature, network, etc.)
- `libsensors5`: Allows `htop` to read the CPU cores temperatures.
- `block-mount`: Allows you to configure and mount hard drives using the LuCI graphical interface (found under System -> Mount Points).

```bash
apk add htop libsensors5 block-mount
```

## How did I identify the unnecessary packages?

After listing all pre-installed packages in OpenWrt 25.12.2 and FriendlyWrt 25.12.2, I compared the lists to identify all `apk` packages unique to FriendlyWrt 25.12.2, and subsequently uninstalled them.  

> [!NOTE]
> This paragraph is here only in case I need to generate a new list of packages to remove with a new version of FriendlyWrt.

1. First, I found the list of pre-installed packages in **OpenWrt 25.12.2** by installing it on the NanoPi R6S and executing the following command:
   ```bash
   apk info
   ```
   <details>
     <summary><font size="4" color="darkred"><b>List of official apk installed in OpenWrt 25.12.2</b></font></summary>
     
     ```txt
     apk-mbedtls
     attendedsysupgrade-common
     base-files
     busybox
     ca-bundle
     cgi-io
     dnsmasq
     dropbear
     e2fsprogs
     firewall4
     fstools
     fwtool
     getrandom
     jansson4
     jshn
     jsonfilter
     kernel
     kmod-crypto-crc32c
     kmod-crypto-hash
     kmod-gpio-button-hotplug
     kmod-hwmon-core
     kmod-i2c-core
     kmod-lib-crc-ccitt
     kmod-lib-crc32c
     kmod-libphy
     kmod-mdio-devres
     kmod-mii
     kmod-nf-conntrack
     kmod-nf-conntrack6
     kmod-nf-flow
     kmod-nf-log
     kmod-nf-log6
     kmod-nf-nat
     kmod-nf-reject
     kmod-nf-reject6
     kmod-nfnetlink
     kmod-nft-core
     kmod-nft-fib
     kmod-nft-nat
     kmod-nft-offload
     kmod-phy-realtek
     kmod-ppp
     kmod-pppoe
     kmod-pppox
     kmod-r8169
     kmod-slhc
     libblkid1
     libblobmsg-json20260213
     libc
     libcomerr0
     libe2p2
     libext2fs2
     libf2fs6
     libgcc1
     libiwinfo-data
     libiwinfo20230701
     libjson-c5
     libjson-script20260213
     liblucihttp-ucode
     liblucihttp0
     libmbedtls21
     libmnl0
     libnftnl11
     libnl-tiny1
     libpthread
     librt
     libsmartcols1
     libss2
     libubox20260213
     libubus20251202
     libuci20250120
     libuclient20201210
     libucode20230711
     libudebug
     libustream-mbedtls20201210
     libuuid1
     logd
     luci
     luci-app-attendedsysupgrade
     luci-app-firewall
     luci-app-package-manager
     luci-base
     luci-lib-uqr
     luci-light
     luci-mod-admin-full
     luci-mod-network
     luci-mod-status
     luci-mod-system
     luci-proto-ipv6
     luci-proto-ppp
     luci-ssl
     luci-theme-bootstrap
     mkf2fs
     mtd
     netifd
     nftables-json
     odhcp6c
     odhcpd-ipv6only
     openwrt-keyring
     owut
     partx-utils
     ppp
     ppp-mod-pppoe
     procd
     procd-seccomp
     procd-ujail
     px5g-mbedtls
     r8169-firmware
     rpcd
     rpcd-mod-file
     rpcd-mod-iwinfo
     rpcd-mod-luci
     rpcd-mod-rpcsys
     rpcd-mod-rrdns
     rpcd-mod-ucode
     uboot-envtools
     ubox
     ubus
     ubusd
     uci
     uclient-fetch
     ucode
     ucode-mod-fs
     ucode-mod-html
     ucode-mod-log
     ucode-mod-math
     ucode-mod-ubus
     ucode-mod-uci
     ucode-mod-uclient
     ucode-mod-uloop
     uhttpd
     uhttpd-mod-ubus
     urandom-seed
     urngd
     usign
     zlib
     ```
   </details>

2. I installed **FriendlyWrt 25.12.2** on the NanoPi R6S and created the file `openwrt_official_apk.txt`, into which I copied the list of official packages obtained from Step 1.
    1. Create the file listing the official APKs:
        ```bash
        vim openwrt_official_apk.txt
        ```
    2. Paste the list obtained from Step 1.
    3. Save and close with `:wq`
3. I created the file `friendlywrt_required_apk.txt` containing the list of essential packages needed for FriendlyWrt's core functionality.
    1. Create the file listing the required APKs:
        ```bash
        vim friendlywrt_required_apk.txt
        ```
    2. Paste:
        ```txt
        libustream-openssl20201210
        luci-compat
        luci-app-emmc-tools
        ```
    3. Save and close with `:wq`
4. Create the script that generates the list of all **top-level packages** (packages that are not used as dependencies by any other package), excluding the packages listed in `openwrt_official_apk.txt` and `friendlywrt_required_apk.txt`.
    1. Create the script that gets the top level packages:
        ```bash
        vim get_top_level_apk.sh
        ```
    2. Paste the following script:
        ```bash
        #!/bin/sh

        # Define the file where the list of removable packages will be saved
        output_file="apks_to_remove.txt"

        # Initialize or clear the output file
        : > "$output_file"

        # Load the protection lists into variables at the beginning to avoid repeated disk I/O
        # These files contain packages that should NEVER be removed
        official=$(cat openwrt_official_apk.txt 2>/dev/null)
        required=$(cat friendlywrt_required_apk.txt 2>/dev/null)

        # Function to check if a package is an orphan (has no other packages depending on it)
        # Returns 0 (true) if NO other package requires this one
        has_reverse_deps() {
            # 'apk info -r' lists reverse dependencies. 
            # We filter out warnings and the header text to get a clean list.
            deps=$(apk info --installed -r "$1" 2>/dev/null | grep -v "^WARNING" | grep -v "is required by:" | grep -v "^$")

            # If the string is empty (-z), it means no other package depends on it
            [ -z "$deps" ]
        }

        # Function to check if a package name exists within a provided list (exact match)
        is_protected_pkg() {
            printf '%s\n' "$2" | grep -Fxq -- "$1"
        }

        # Retrieve the list of all currently installed packages
        pkg_list="$(apk info -q 2>/dev/null)"

        # Iterate through every installed package
        for pkg in $pkg_list; do
            # Execute each check in a subshell background process (&) to significantly 
            # speed up the execution by allowing multiple checks to run in parallel.
            (
                # Step 1: Check if the package is an orphan (has no reverse dependencies)
                if has_reverse_deps "$pkg"; then
                    # Step 2: Ensure the package is NOT in the official or required protection lists
                    if ! is_protected_pkg "$pkg" "$official" && ! is_protected_pkg "$pkg" "$required"; then
                        # If both conditions are met, it's safe to suggest removal
                        printf '%s\n' "$pkg" >> "$output_file"
                    fi
                fi
            ) &
        done

        # Wait for all background subshells to finish before exiting the script
        wait
        ```
    3. Save and close with `:wq`
    4. Set execution permissions: 
        ```bash
        chmod +x get_top_level_apk.sh
        ```
5. Execute:
    ```bash
    ./get_top_level_apk.sh
    ```  
   This creates the file `apks_to_remove.txt` with the list of packages to be removed.  
6. Uninstall the packages obtained from Step 5:
    ```bash
    xargs apk del < apks_to_remove.txt
    ```

> [!NOTE]
> Steps 5 and 6 must be repeated multiple times. Even after the initial removal, many remaining orphaned packages might not be automatically uninstalled (likely because they were installed prior to the top-level packages and not pulled in automatically as dependencies of the top-level packages). These packages then become new top-level candidates, so you must repeat Steps 5 and 6 until `apks_to_remove.txt` is empty. For FriendlyWrt 25.12.2, you'll likely need to repeat steps 5 and 6 about ten times to ensure all unnecessary packages are removed.
