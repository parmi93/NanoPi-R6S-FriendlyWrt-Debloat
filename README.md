# NanoPi R6S FriendlyWrt Debloat

**NanoPi R6S FriendlyWrt Debloat Guide.** A script and guide to identify and safely remove all non-official/non-stock OpenWrt opkg packages installed by FriendlyWrt on the NanoPi R6S. Restore a cleaner, leaner OpenWrt environment while keeping core system functionality.

> [!NOTE]
> This guide is primarily written for my own future reference, in case I need to perform this procedure again.

## Comparing OpenWrt and FriendlyWrt after a clean installation
|                                      | No. OPKGs | RAM Usage |
|--------------------------------------|-----------|-----------|
| OpenWrt v24.10.2                     | 128       | ~93 MiB   |
| Friendly v24.10.2                    | 1080      | ~405 MiB  |
| Friendly v24.10.2 <br> after debloat | 134       | ~275 MiB  |


## Switch back to the OpenWrt shell

> [!CAUTION]
> After the cleanup, the LuCI interface will break (which we will fix later) and the `bash` shell (SSH) will be uninstalled, meaning you will lose the ability to interact with FriendlyWrt.  
> This step ensures that your shell remains functional after the cleanup.

FriendlyWrt uses the `bash` package shell by default (which will be uninstalled during cleanup). To ensure your shell (SSH) continues to function, you must switch to the official OpenWrt shell (`ash`) provided by the official `busybox` package.  
Edit the `/etc/passwd` file:
1. `vim /etc/passwd`
2. Find a line that contain `...bin/bash` and change it to `...bin/ash`
3. Save and close with `:wq`

## Uninstall unnecessary `opkg` packages

After listing all pre-installed packages in OpenWrt v24.10.2 and FriendlyWrt v24.10.2, I compared the lists to identify all `opkg` packages unique to FriendlyWrt v24.10.2, and subsequently uninstalled them.  
Follow these steps to uninstall unnecessary packages:
1. `vim opkg_uninstall.sh`
2. Paste this massive command:
    ```bash
    opkg --force-removal-of-dependent-packages --autoremove remove bash batctl-default bind-dig blkdiscard blkid blockd bsdiff bspatch btrfs-progs ca-certificates cfdisk collectd-mod-cpufreq collectd-mod-df collectd-mod-thermal collectd-mod-uptime collectd-mod-wireless coremark coreutils-base64 coreutils-basename coreutils-cat coreutils-chgrp coreutils-chmod coreutils-chown coreutils-chroot coreutils-cp coreutils-cut coreutils-date coreutils-dd coreutils-df coreutils-dirname coreutils-du coreutils-echo coreutils-env coreutils-expr coreutils-false coreutils-head coreutils-id coreutils-ln coreutils-ls coreutils-md5sum coreutils-mkdir coreutils-mkfifo coreutils-mknod coreutils-mktemp coreutils-mv coreutils-nice coreutils-nohup coreutils-nproc coreutils-od coreutils-printf coreutils-pwd coreutils-readlink coreutils-realpath coreutils-rm coreutils-rmdir coreutils-seq coreutils-sha1sum coreutils-sha256sum coreutils-sleep coreutils-stat coreutils-sync coreutils-tail coreutils-tee coreutils-test coreutils-timeout coreutils-touch coreutils-tr coreutils-true coreutils-truncate coreutils-tty coreutils-uname coreutils-uniq coreutils-unlink coreutils-wc coreutils-yes curl diffutils dnsmasq-full ethtool extract fdisk file findutils fstrim getopt git-http grep gzip hostapd-openssl hwclock ip-full iperf iperf3 ipset iptables-mod-checksum iptables-mod-cluster iptables-mod-conntrack-extra iptables-mod-conntrack-label iptables-mod-extra iptables-mod-filter iptables-mod-hashlimit iptables-mod-iprange iptables-mod-ipsec iptables-mod-led iptables-mod-nat-extra iptables-mod-nflog iptables-mod-nfqueue iptables-mod-physdev iptables-mod-rpfilter iptables-mod-tee iptables-mod-tproxy iptables-mod-trace iptables-mod-u32 iwlwifi-firmware-ax200 iwlwifi-firmware-ax210 kmod kmod-brcmfmac kmod-crypto-gcm kmod-dummy kmod-ikconfig kmod-ipsec4 kmod-macvlan kmod-nf-ipvs kmod-nf-nathelper kmod-nf-nathelper-extra kmod-nft-socket kmod-nft-tproxy kmod-sched-connmark kmod-tun kmod-usb-net-rtl8152 kmod-veth kmod-wireguard libfido2-1 libiwinfo-lua libopenssl-devcrypto libseccomp libuci-lua libxml2 luci-app-netdata luci-i18n-adblock-bg luci-i18n-adblock-ca luci-i18n-adblock-cs luci-i18n-adblock-de luci-i18n-adblock-el luci-i18n-adblock-es luci-i18n-adblock-fr luci-i18n-adblock-he luci-i18n-adblock-hi luci-i18n-adblock-hu luci-i18n-adblock-it luci-i18n-adblock-ja luci-i18n-adblock-ko luci-i18n-adblock-mr luci-i18n-adblock-ms luci-i18n-adblock-pl luci-i18n-adblock-pt luci-i18n-adblock-pt-br luci-i18n-adblock-ro luci-i18n-adblock-ru luci-i18n-adblock-sk luci-i18n-adblock-sv luci-i18n-adblock-tr luci-i18n-adblock-uk luci-i18n-adblock-vi luci-i18n-adblock-zh-cn luci-i18n-adblock-zh-tw luci-i18n-aria2-bg luci-i18n-aria2-ca luci-i18n-aria2-cs luci-i18n-aria2-de luci-i18n-aria2-el luci-i18n-aria2-es luci-i18n-aria2-fr luci-i18n-aria2-he luci-i18n-aria2-hi luci-i18n-aria2-hu luci-i18n-aria2-it luci-i18n-aria2-ja luci-i18n-aria2-ko luci-i18n-aria2-mr luci-i18n-aria2-ms luci-i18n-aria2-pl luci-i18n-aria2-pt luci-i18n-aria2-pt-br luci-i18n-aria2-ro luci-i18n-aria2-ru luci-i18n-aria2-sk luci-i18n-aria2-sv luci-i18n-aria2-tr luci-i18n-aria2-uk luci-i18n-aria2-vi luci-i18n-aria2-zh-cn luci-i18n-aria2-zh-tw luci-i18n-base-bg luci-i18n-base-ca luci-i18n-base-cs luci-i18n-base-de luci-i18n-base-el luci-i18n-base-es luci-i18n-base-fr luci-i18n-base-he luci-i18n-base-hi luci-i18n-base-hu luci-i18n-base-it luci-i18n-base-ja luci-i18n-base-ko luci-i18n-base-mr luci-i18n-base-ms luci-i18n-base-pl luci-i18n-base-pt luci-i18n-base-pt-br luci-i18n-base-ro luci-i18n-base-ru luci-i18n-base-sk luci-i18n-base-sv luci-i18n-base-tr luci-i18n-base-uk luci-i18n-base-vi luci-i18n-base-zh-cn luci-i18n-base-zh-tw luci-i18n-commands-bg luci-i18n-commands-ca luci-i18n-commands-cs luci-i18n-commands-de luci-i18n-commands-el luci-i18n-commands-es luci-i18n-commands-fr luci-i18n-commands-he luci-i18n-commands-hi luci-i18n-commands-hu luci-i18n-commands-it luci-i18n-commands-ja luci-i18n-commands-ko luci-i18n-commands-mr luci-i18n-commands-ms luci-i18n-commands-pl luci-i18n-commands-pt luci-i18n-commands-pt-br luci-i18n-commands-ro luci-i18n-commands-ru luci-i18n-commands-sk luci-i18n-commands-sv luci-i18n-commands-tr luci-i18n-commands-uk luci-i18n-commands-vi luci-i18n-commands-zh-cn luci-i18n-commands-zh-tw luci-i18n-cpufreq-zh-cn luci-i18n-ddns-bg luci-i18n-ddns-ca luci-i18n-ddns-cs luci-i18n-ddns-de luci-i18n-ddns-el luci-i18n-ddns-es luci-i18n-ddns-fr luci-i18n-ddns-he luci-i18n-ddns-hi luci-i18n-ddns-hu luci-i18n-ddns-it luci-i18n-ddns-ja luci-i18n-ddns-ko luci-i18n-ddns-mr luci-i18n-ddns-ms luci-i18n-ddns-pl luci-i18n-ddns-pt luci-i18n-ddns-pt-br luci-i18n-ddns-ro luci-i18n-ddns-ru luci-i18n-ddns-sk luci-i18n-ddns-sv luci-i18n-ddns-tr luci-i18n-ddns-uk luci-i18n-ddns-vi luci-i18n-ddns-zh-cn luci-i18n-ddns-zh-tw luci-i18n-firewall-bg luci-i18n-firewall-ca luci-i18n-firewall-cs luci-i18n-firewall-de luci-i18n-firewall-el luci-i18n-firewall-es luci-i18n-firewall-fr luci-i18n-firewall-he luci-i18n-firewall-hi luci-i18n-firewall-hu luci-i18n-firewall-it luci-i18n-firewall-ja luci-i18n-firewall-ko luci-i18n-firewall-mr luci-i18n-firewall-ms luci-i18n-firewall-pl luci-i18n-firewall-pt luci-i18n-firewall-pt-br luci-i18n-firewall-ro luci-i18n-firewall-ru luci-i18n-firewall-sk luci-i18n-firewall-sv luci-i18n-firewall-tr luci-i18n-firewall-uk luci-i18n-firewall-vi luci-i18n-firewall-zh-cn luci-i18n-firewall-zh-tw luci-i18n-hd-idle-bg luci-i18n-hd-idle-ca luci-i18n-hd-idle-cs luci-i18n-hd-idle-de luci-i18n-hd-idle-el luci-i18n-hd-idle-es luci-i18n-hd-idle-fr luci-i18n-hd-idle-he luci-i18n-hd-idle-hi luci-i18n-hd-idle-hu luci-i18n-hd-idle-it luci-i18n-hd-idle-ja luci-i18n-hd-idle-ko luci-i18n-hd-idle-mr luci-i18n-hd-idle-ms luci-i18n-hd-idle-pl luci-i18n-hd-idle-pt luci-i18n-hd-idle-pt-br luci-i18n-hd-idle-ro luci-i18n-hd-idle-ru luci-i18n-hd-idle-sk luci-i18n-hd-idle-sv luci-i18n-hd-idle-tr luci-i18n-hd-idle-uk luci-i18n-hd-idle-vi luci-i18n-hd-idle-zh-cn luci-i18n-hd-idle-zh-tw luci-i18n-minidlna-bg luci-i18n-minidlna-ca luci-i18n-minidlna-cs luci-i18n-minidlna-de luci-i18n-minidlna-el luci-i18n-minidlna-es luci-i18n-minidlna-fr luci-i18n-minidlna-he luci-i18n-minidlna-hi luci-i18n-minidlna-hu luci-i18n-minidlna-it luci-i18n-minidlna-ja luci-i18n-minidlna-ko luci-i18n-minidlna-mr luci-i18n-minidlna-ms luci-i18n-minidlna-pl luci-i18n-minidlna-pt luci-i18n-minidlna-pt-br luci-i18n-minidlna-ro luci-i18n-minidlna-ru luci-i18n-minidlna-sk luci-i18n-minidlna-sv luci-i18n-minidlna-tr luci-i18n-minidlna-uk luci-i18n-minidlna-vi luci-i18n-minidlna-zh-cn luci-i18n-minidlna-zh-tw luci-i18n-nft-qos-bg luci-i18n-nft-qos-ca luci-i18n-nft-qos-cs luci-i18n-nft-qos-de luci-i18n-nft-qos-el luci-i18n-nft-qos-es luci-i18n-nft-qos-fr luci-i18n-nft-qos-he luci-i18n-nft-qos-hi luci-i18n-nft-qos-hu luci-i18n-nft-qos-it luci-i18n-nft-qos-ja luci-i18n-nft-qos-ko luci-i18n-nft-qos-mr luci-i18n-nft-qos-ms luci-i18n-nft-qos-pl luci-i18n-nft-qos-pt luci-i18n-nft-qos-pt-br luci-i18n-nft-qos-ro luci-i18n-nft-qos-ru luci-i18n-nft-qos-sk luci-i18n-nft-qos-sv luci-i18n-nft-qos-tr luci-i18n-nft-qos-uk luci-i18n-nft-qos-vi luci-i18n-nft-qos-zh-cn luci-i18n-nft-qos-zh-tw luci-i18n-nlbwmon-bg luci-i18n-nlbwmon-ca luci-i18n-nlbwmon-cs luci-i18n-nlbwmon-de luci-i18n-nlbwmon-el luci-i18n-nlbwmon-es luci-i18n-nlbwmon-fr luci-i18n-nlbwmon-he luci-i18n-nlbwmon-hi luci-i18n-nlbwmon-hu luci-i18n-nlbwmon-it luci-i18n-nlbwmon-ja luci-i18n-nlbwmon-ko luci-i18n-nlbwmon-mr luci-i18n-nlbwmon-ms luci-i18n-nlbwmon-pl luci-i18n-nlbwmon-pt luci-i18n-nlbwmon-pt-br luci-i18n-nlbwmon-ro luci-i18n-nlbwmon-ru luci-i18n-nlbwmon-sk luci-i18n-nlbwmon-sv luci-i18n-nlbwmon-tr luci-i18n-nlbwmon-uk luci-i18n-nlbwmon-vi luci-i18n-nlbwmon-zh-cn luci-i18n-nlbwmon-zh-tw luci-i18n-package-manager-bg luci-i18n-package-manager-ca luci-i18n-package-manager-cs luci-i18n-package-manager-de luci-i18n-package-manager-el luci-i18n-package-manager-es luci-i18n-package-manager-fr luci-i18n-package-manager-he luci-i18n-package-manager-hi luci-i18n-package-manager-hu luci-i18n-package-manager-it luci-i18n-package-manager-ja luci-i18n-package-manager-ko luci-i18n-package-manager-mr luci-i18n-package-manager-ms luci-i18n-package-manager-pl luci-i18n-package-manager-pt luci-i18n-package-manager-pt-br luci-i18n-package-manager-ro luci-i18n-package-manager-ru luci-i18n-package-manager-sk luci-i18n-package-manager-sv luci-i18n-package-manager-tr luci-i18n-package-manager-uk luci-i18n-package-manager-vi luci-i18n-package-manager-zh-cn luci-i18n-package-manager-zh-tw luci-i18n-samba4-bg luci-i18n-samba4-ca luci-i18n-samba4-cs luci-i18n-samba4-de luci-i18n-samba4-el luci-i18n-samba4-es luci-i18n-samba4-fr luci-i18n-samba4-he luci-i18n-samba4-hi luci-i18n-samba4-hu luci-i18n-samba4-it luci-i18n-samba4-ja luci-i18n-samba4-ko luci-i18n-samba4-mr luci-i18n-samba4-ms luci-i18n-samba4-pl luci-i18n-samba4-pt luci-i18n-samba4-pt-br luci-i18n-samba4-ro luci-i18n-samba4-ru luci-i18n-samba4-sk luci-i18n-samba4-sv luci-i18n-samba4-tr luci-i18n-samba4-uk luci-i18n-samba4-vi luci-i18n-samba4-zh-cn luci-i18n-samba4-zh-tw luci-i18n-smartdns-bg luci-i18n-smartdns-ca luci-i18n-smartdns-cs luci-i18n-smartdns-de luci-i18n-smartdns-el luci-i18n-smartdns-es luci-i18n-smartdns-fr luci-i18n-smartdns-he luci-i18n-smartdns-hi luci-i18n-smartdns-hu luci-i18n-smartdns-it luci-i18n-smartdns-ja luci-i18n-smartdns-ko luci-i18n-smartdns-mr luci-i18n-smartdns-ms luci-i18n-smartdns-pl luci-i18n-smartdns-pt luci-i18n-smartdns-pt-br luci-i18n-smartdns-ro luci-i18n-smartdns-ru luci-i18n-smartdns-sk luci-i18n-smartdns-sv luci-i18n-smartdns-tr luci-i18n-smartdns-uk luci-i18n-smartdns-vi luci-i18n-smartdns-zh-cn luci-i18n-smartdns-zh-tw luci-i18n-sqm-bg luci-i18n-sqm-ca luci-i18n-sqm-cs luci-i18n-sqm-de luci-i18n-sqm-el luci-i18n-sqm-es luci-i18n-sqm-fr luci-i18n-sqm-he luci-i18n-sqm-hi luci-i18n-sqm-hu luci-i18n-sqm-it luci-i18n-sqm-ja luci-i18n-sqm-ko luci-i18n-sqm-mr luci-i18n-sqm-ms luci-i18n-sqm-pl luci-i18n-sqm-pt luci-i18n-sqm-pt-br luci-i18n-sqm-ro luci-i18n-sqm-ru luci-i18n-sqm-sk luci-i18n-sqm-sv luci-i18n-sqm-tr luci-i18n-sqm-uk luci-i18n-sqm-vi luci-i18n-sqm-zh-cn luci-i18n-sqm-zh-tw luci-i18n-statistics-bg luci-i18n-statistics-ca luci-i18n-statistics-cs luci-i18n-statistics-de luci-i18n-statistics-el luci-i18n-statistics-es luci-i18n-statistics-fr luci-i18n-statistics-he luci-i18n-statistics-hi luci-i18n-statistics-hu luci-i18n-statistics-it luci-i18n-statistics-ja luci-i18n-statistics-ko luci-i18n-statistics-mr luci-i18n-statistics-ms luci-i18n-statistics-pl luci-i18n-statistics-pt luci-i18n-statistics-pt-br luci-i18n-statistics-ro luci-i18n-statistics-ru luci-i18n-statistics-sk luci-i18n-statistics-sv luci-i18n-statistics-tr luci-i18n-statistics-uk luci-i18n-statistics-vi luci-i18n-statistics-zh-cn luci-i18n-statistics-zh-tw luci-i18n-ttyd-bg luci-i18n-ttyd-ca luci-i18n-ttyd-cs luci-i18n-ttyd-de luci-i18n-ttyd-el luci-i18n-ttyd-es luci-i18n-ttyd-fr luci-i18n-ttyd-he luci-i18n-ttyd-hi luci-i18n-ttyd-hu luci-i18n-ttyd-it luci-i18n-ttyd-ja luci-i18n-ttyd-ko luci-i18n-ttyd-mr luci-i18n-ttyd-ms luci-i18n-ttyd-pl luci-i18n-ttyd-pt luci-i18n-ttyd-pt-br luci-i18n-ttyd-ro luci-i18n-ttyd-ru luci-i18n-ttyd-sk luci-i18n-ttyd-sv luci-i18n-ttyd-tr luci-i18n-ttyd-uk luci-i18n-ttyd-vi luci-i18n-ttyd-zh-cn luci-i18n-ttyd-zh-tw luci-i18n-upnp-bg luci-i18n-upnp-ca luci-i18n-upnp-cs luci-i18n-upnp-de luci-i18n-upnp-el luci-i18n-upnp-es luci-i18n-upnp-fr luci-i18n-upnp-he luci-i18n-upnp-hi luci-i18n-upnp-hu luci-i18n-upnp-it luci-i18n-upnp-ja luci-i18n-upnp-ko luci-i18n-upnp-mr luci-i18n-upnp-ms luci-i18n-upnp-pl luci-i18n-upnp-pt luci-i18n-upnp-pt-br luci-i18n-upnp-ro luci-i18n-upnp-ru luci-i18n-upnp-sk luci-i18n-upnp-sv luci-i18n-upnp-tr luci-i18n-upnp-uk luci-i18n-upnp-vi luci-i18n-upnp-zh-cn luci-i18n-upnp-zh-tw luci-i18n-watchcat-bg luci-i18n-watchcat-ca luci-i18n-watchcat-cs luci-i18n-watchcat-de luci-i18n-watchcat-el luci-i18n-watchcat-es luci-i18n-watchcat-fr luci-i18n-watchcat-he luci-i18n-watchcat-hi luci-i18n-watchcat-hu luci-i18n-watchcat-it luci-i18n-watchcat-ja luci-i18n-watchcat-ko luci-i18n-watchcat-mr luci-i18n-watchcat-ms luci-i18n-watchcat-pl luci-i18n-watchcat-pt luci-i18n-watchcat-pt-br luci-i18n-watchcat-ro luci-i18n-watchcat-ru luci-i18n-watchcat-sk luci-i18n-watchcat-sv luci-i18n-watchcat-tr luci-i18n-watchcat-uk luci-i18n-watchcat-vi luci-i18n-watchcat-zh-cn luci-i18n-watchcat-zh-tw luci-lib-ipkg luci-lib-iptparser luci-proto-3g luci-proto-qmi luci-ssl-openssl luci-theme-material luci-theme-openwrt-2020 mount-utils mt76x2-firmware mt792x-firmware openssh-client-utils openssh-sftp-server parted pciutils ppp-mod-passwordfd ppp-mod-pppoa ppp-mod-pppol2tp ppp-mod-pptp ppp-mod-radius psmisc px5g-wolfssl qmi-utils qrencode resize2fs rtl8822be-firmware rtl8822ce-firmware sed sfdisk strace tar tini triggerhappy tune2fs umbim unrar unzip usb-modeswitch-official usbutils uuidgen vim-full vsftpd wget-ssl wpa-supplicant-openssl wsdd2 bind-libs block-mount bzip2 comgt git hostapd-common kmod-crypto-ctr kmod-crypto-lib-chacha20poly1305 kmod-crypto-lib-curve25519 kmod-cryptodev kmod-fs-autofs4 kmod-fs-btrfs kmod-ipsec kmod-ipt-checksum kmod-ipt-cluster kmod-ipt-conntrack-extra kmod-ipt-conntrack-label kmod-ipt-debug kmod-ipt-extra kmod-ipt-filter kmod-ipt-hashlimit kmod-ipt-iprange kmod-ipt-ipsec kmod-ipt-led kmod-ipt-nat-extra kmod-ipt-nflog kmod-ipt-nfqueue kmod-ipt-physdev kmod-ipt-rpfilter kmod-ipt-tee kmod-ipt-tproxy kmod-ipt-u32 kmod-mmc kmod-mppe kmod-nf-socket kmod-pppoa kmod-pppol2tp kmod-pptp kmod-usb-net-cdc-mbim libacl libbpf1 libcbor0 libcurl4 libevdev libextractor libfdisk1 libipset13 libkmod libparted libpci libqmi libqrencode libusb-1.0-0 libwolfsslcpu-crypto5.7.6.e624513f linux-atm luci-app-adblock luci-app-aria2 luci-app-commands luci-app-cpufreq luci-app-ddns luci-app-hd-idle luci-app-minidlna luci-app-nft-qos luci-app-nlbwmon luci-app-samba4 luci-app-smartdns luci-app-sqm luci-app-statistics luci-app-ttyd luci-app-upnp luci-app-watchcat openssh-client openssl-util pciids uqmi usbids iptables-mod-ipopt iptables-nft kmod-atm kmod-crypto-arc4 kmod-crypto-blake2b kmod-crypto-deflate kmod-crypto-des kmod-crypto-ecb kmod-crypto-echainiv kmod-crypto-kpp kmod-crypto-lib-chacha20 kmod-crypto-md5 kmod-crypto-xxhash kmod-gre kmod-ifb kmod-ipt-conntrack kmod-ipt-ipset kmod-ipt-nat kmod-ipt-raw kmod-ipt-raw6 kmod-lib-textsearch kmod-nf-conncount kmod-nf-dup-inet kmod-nf-tproxy kmod-nft-bridge kmod-nft-netdev kmod-sched-cake kmod-udptunnel4 kmod-udptunnel6 kmod-usb-net-cdc-ether kmod-usb-net-qmi-wwan libcap libdaemon libdevmapper libelf1 libltdl7 libmbim libnettle8 libopenssl-legacy libpopt0 libreadline8 libubus-lua libunistring lsblk rpcd-mod-rpcsys tc-tiny ucode-mod-lua glib2 kmod-dm kmod-ipt-ipopt kmod-iptunnel kmod-nf-ipt6 kmod-sched-core kmod-usb-net kmod-usb-wdm libgmp10 libmount1 libncurses6 libopenssl-conf xtables-nft kmod-dax kmod-ipt-core kmod-nft-compat kmod-usb-core libiptext-nft0 libiptext0 libiptext6-0 libpcre2 terminfo kmod-nf-ipt kmod-nls-base libxtables12 kmod-nf-conntrack-netlink
    ```
3. Save and close with `:wq`
4. Set execution permissions: `chmod +x opkg_uninstall.sh`
5. Excecute to unistall: `./opkg_uninstall.sh`

By reviewing the output, you'll notice that some configuration files were not removed, so delete them manually using the command:
```bash
rm /etc/config/dhcp /etc/config/adblock /etc/config/nft-qos /etc/config/samba4 /etc/config/smartdns /etc/config/luci_statistics /etc/config/ttyd /etc/config/watchcat
```

### Some packages are required
Two packages must not be uninstalled:

- `luci-lib-fs`: This package is not included in standard OpenWrt, but it seems vital for the FriendlyWrt LuCI interface to work correctly. Crucially, if you uninstall it, you won't be able to get it back, as it is missing from the package repository.
- `libustream-openssl20201210`: FriendlyWrt replaced Mbed TLS with OpenSSL, which makes sense because Mbed TLS is designed for truly low-end platforms with limited memory at the expense of speed performance. OpenSSL, while using more memory (which is not an issue with the NanoPi R6S, as we have 8GiB of RAM), offers higher speed performance. Therefore, I decided to keep `libustream-openssl20201210` instead of reverting to the official package, `libustream-mbedtls20201210`.

## Install some essentials `opkg`s

Install two packages:
1. `luci-compat`: Not being a standard OpenWrt package, it was automatically removed as an orphan during a system cleanup. However, it seems this package is actually needed to resolve the broken LuCI interface. The FriendlyWrt LuCI setup, unlike the standard version, appears to be dependent on `luci-compat` for correct operation.
2. `dnsmasq`: This is the official package found in OpenWrt. FriendlyWrt replaced it with the full version, `dnsmasq-full` (which you just uninstalled), so by installing `dnsmasq` we revert back to the original one.

```bash
opkg update
opkg install luci-compat dnsmasq
```

## [OPTIONAL] Install some usefull package

I found a few particularly useful packages:
- `htop`: Allows real-time monitoring of system resource usage (CPU, memory, temperature, network, etc.)
- `libsensors5`: Allows `htop` to read the CPU cores temperatures.
- `block-mount`: Allows you to configure and mount hard drives using the LuCI graphical interface (found under System -> Mount Points).

```bash
opkg install htop libsensors5 block-mount
```

## How did I identify the unnecessary packages?

This section explains the method used to determine which packages added by FriendlyWrt are not part of the standard OpenWrt installation.

> [!NOTE]
> This paragraph is here only in case I need to generate a new list of packages to remove with a new version of FriendlyWrt.

1. First, I found the list of pre-installed packages in **OpenWrt 24.10.2** by installing it on the NanoPi R6S and executing the following command:
   ```bash
   opkg list-installed | awk '{print $1}'
   ```
   <details>
     <summary><font size="4" color="darkred"><b>List of official opkg extracted from OpenWrt 24.10.2</b></font></summary>
     
     ```txt
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
     kmod-crypto-acompress
     kmod-crypto-crc32c
     kmod-crypto-hash
     kmod-gpio-button-hotplug
     kmod-hwmon-core
     kmod-lib-crc-ccitt
     kmod-lib-crc32c
     kmod-lib-lzo
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
     libblobmsg-json20240329
     libc
     libcomerr0
     libe2p2
     libext2fs2
     libf2fs6
     libgcc1
     libiwinfo-data
     libiwinfo20230701
     libjson-c5
     libjson-script20240329
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
     libubox20240329
     libubus20250102
     libuci20250120
     libuclient20201210
     libucode20230711
     libudebug
     libustream-mbedtls20201210
     libuuid1
     logd
     luci
     luci-app-firewall
     luci-app-package-manager
     luci-base
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
     opkg
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
     ucode-mod-math
     ucode-mod-ubus
     ucode-mod-uci
     uhttpd
     uhttpd-mod-ubus
     urandom-seed
     urngd
     usign
     ```
   </details>

2. I installed **FriendlyWrt 24.10.2** on the NanoPi R6S and created the file `openwrt_official_opkg.txt`, into which I copied the list of official packages obtained from Step 1.
    1. `vim openwrt_official_opkg.txt`
    2. Paste the list obtained from Step 1.
    3. Save and close with `:wq`
3. I created the file `friendlywrt_required_opkg.txt` containing the list of essential packages needed for FriendlyWrt's core functionality.
    1. `vim friendlywrt_required_opkg.txt`
    2. Paste:
        ```txt
        luci-lib-fs
        libustream-openssl20201210
        ```
    3. Save and close with `:wq`
4. Create the script that generates the list of all **top-level packages** (packages that are not used as dependencies by any other package), excluding the packages listed in `openwrt_official_opkg.txt`, `friendlywrt_required_opkg.txt` and package that are marked as essential (`Essential: yes`) by FriendlyWrt.
    1. `vim get_top_level_opkg.sh`
    2. Paste the following script (it uses `awk` to parse package status and dependencies):
       ```bash
       opkg list-installed | awk '{print $1}' | sort > /tmp/all_pkgs.txt

       opkg status | awk '
         function read_file_or_error(filename) {
           ret = getline line < filename
           if (ret == -1) {
             print "ERROR: " filename " is missing!" > "/dev/stderr"
             exit 1
           } else {
             print line
             while ((getline line < filename) > 0) {
               print line
             }
             close(filename)
             return 1
           }
         }
         
         /^Package:/ { 
           pkg = $2
         }
         /^Essential: yes/ { essential[pkg] = 1 }
         /^Provides:/ {
           gsub(/^Provides: /, "")
           split($0, aliases, ", ")
           for (i in aliases) {
             sub(/ .*/, "", aliases[i])
             if (aliases[i] in provides) {
               print "ERROR: " pkg " has " aliases[i] " as alias which has been already found for " provides[aliases[i]] " package"
             } else {
               provides[aliases[i]] = pkg
             }
           }
         }
         /^Depends:/ { 
           gsub(/^Depends: /, "")
           split($0, deps, ", ")
           for (j in deps) {
             sub(/ .*/, "", deps[j])
             dep[deps[j]] = 1
           }
         }
         END {
           # Print essential packages
           for (e in essential) print e
           
           # Read official and required packages
           read_file_or_error("openwrt_official_opkg.txt")
           read_file_or_error("friendlywrt_required_opkg.txt")
           
           # Print direct dependencies
           for (d in dep) print d
           
           # Print packages that provide the dependencies
           for (d in dep) {
             if (d in provides) {
               print provides[d]
             }
           }
         }
       ' | sort -u > /tmp/exclude.txt

       grep -vxFf /tmp/exclude.txt /tmp/all_pkgs.txt | tr '\n' ' ' > opkgs_to_remove.txt
       grep "ERROR" /tmp/exclude.txt
       rm /tmp/all_pkgs.txt /tmp/exclude.txt
       ```
    3. Save and close with `:wq`
    4. Set execution permissions: `chmod +x get_top_level_opkg.sh`
5. Execute: `./get_top_level_opkg.sh`.  
   This creates the file `opkgs_to_remove.txt` with the list of packages to be removed.  
   **Note**: The script returns two errors that, in theory, should not occur, as they essentially indicate that there are two packages with the same name. Specifically, there are two packages that share the same alias, which doesn't make sense. In any case, I decided to ignore these errors and proceed with the subsequent steps.
   ```txt
   ERROR: ca-bundle has ca-certs as alias which has been already found for ca-certificates package
   ERROR: wget-ssl has wget as alias which has been already found for uclient-fetch package
   ```
6. Uninstall the packages obtained from Step 5:
   ```bash
   xargs opkg --force-removal-of-dependent-packages --autoremove remove < opkgs_to_remove.txt
   ```

> [!NOTE]
> Steps 5 and 6 must be repeated multiple times. Even after the initial removal, many remaining orphaned packages might not be automatically uninstalled (likely because they were installed prior to the top-level packages and not pulled in automatically as dependencies of the top-level packages). These packages then become new top-level candidates, so you must repeat Steps 5 and 6 until `opkgs_to_remove.txt` is empty. For FriendlyWrt 24.10.2, you'll likely need to repeat steps 5 and 6 about seven times to ensure all unnecessary packages are removed.
