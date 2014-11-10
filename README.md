# Vanilla OpenWrt for Arduino Yún

## Description

This is my port of a vanilla [OpenWrt](https://openwrt.org) for [Arduino
Yún](http://arduino.cc/en/Main/ArduinoBoardYun?from=Products.ArduinoYUN). It's unofficial and meant for developers,
testers and advanced users.

Current version is [Barrier Breaker (14.07)](https://forum.openwrt.org/viewtopic.php?pid=242292#p242292).

## Disclaimer

**Flash this at your own risk, I take no responsibility for your device! Flashing will VOID your warranty!**

## What's included

- The default OpenWrt packages plus `block-mount` and `e2fsprogs` for easy extrooting.
- Yún specific patches from [arduino/openwrt-yun](https://github.com/arduino/openwrt-yun).

Note: this build does not include [`LuCI`](http://wiki.openwrt.org/doc/howto/luci.essentials), but it can be installed later, [see below](#installing-luci).

## License

See:
- [LICENSE](blob/master/LICENSE)
- [OpenWrt license](http://wiki.openwrt.org/about/license)
- [OpenWrt Yún license](https://github.com/arduino/openwrt-yun/blob/master/LICENSE)

## Downloads

See [the releases page](https://github.com/ljani/openwrt-yun/releases).

## Flashing with TFTP

I'd suggest you to do the initial flashing using TFTP, because it teaches you how to restore the original if anything goes wrong.  To do this, familiarize yourself with [the original instructions](http://arduino.cc/en/Tutorial/YunUBootReflash), but use the binaries from the [releases](https://github.com/ljani/testing/releases) page.

Short summary, run these commands over the serial:

```
setenv serverip 192.168.3.1;
setenv ipaddr 192.168.3.2;

tftp 0x80060000 openwrt-ar71xx-generic-yun-kernel.bin;
erase 0x9fea0000 +0x140000;
cp.b $fileaddr 0x9fea0000 $filesize;

tftp 0x80060000 openwrt-ar71xx-generic-yun-rootfs-squashfs.bin;
erase 0x9f050000 +0xe50000;
cp.b $fileaddr 0x9f050000 $filesize;

bootm 0x9fea0000;
```

## Flashing with sysupgrade

If you're coming from the original the magic signature won't probably match, so please use the `--force` option. Again
refer to [the original guide](http://arduino.cc/en/Tutorial/YunSysupgrade) (section `Upgrading Using the Terminal`).

Summary:

```
sysupgrade -v -n --force openwrt-ar71xx-generic-yun-squashfs-sysupgrade.bin
```

## Restoring the original image

Follow [the reflashing guide](http://arduino.cc/en/Tutorial/YunUBootReflash), but skip flashing `u-boot`.

## Tips

### Extroot

[Original guide](http://wiki.openwrt.org/doc/howto/extroot).

Short summary:
```
# Format the SD card
mkfs.ext4 /dev/sda1

# Optionally copy the current configuration:
mount /dev/sda1 /mnt
tar -C /overlay -cvf - . | tar -C /mnt -xf -
sync
umount /mnt

# Check the UUID by running
block info

# Edit fstab to something like this, change the UUID though:
cat /etc/config/fstab
config 'global'
        option  anon_swap       '0'
        option  anon_mount      '0'
        option  auto_swap       '1'
        option  auto_mount      '1'
        option  delay_root      '5'
        option  check_fs        '0'

config 'mount'
        option target           '/overlay'
        option uuid             '4626e87b-20dd-4e94-b8ae-1752419e902b'
        option fstype           'ext4'
        option options          'noatime,dev,exec,suid,rw'
        option enabled          '1'
        option enabled_fsck     '1'
```

### WiFi

[Original info](http://wiki.openwrt.org/doc/uci/wireless)

Short summary:
```
# Here's my config:
cat /etc/config/network
config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd7e:56fd:e086::/48'

config interface 'lan'
        option proto 'dhcp'
        option hostname 'yun'

config interface 'wan'
        option ifname 'eth1'
        option proto 'dhcp'

config interface 'wan6'
        option ifname '@wan'
        option proto 'dhcpv6'

root@yun:/# cat /etc/config/wireless
config wifi-device  radio0
        option type     mac80211
        option channel  auto
        option hwmode   11g
        option path     'platform/ar933x_wmac'
        option htmode   HT40+
        option country  '00'

config wifi-iface
        option device           'radio0'
        option network          'lan'
        option mode             'sta'
        option ssid             'MySSID'
        option encryption       'psk2'
        option key              'MyKey123'
```

### Installing LuCI

[Original guide](http://wiki.openwrt.org/doc/howto/luci.essentials).

Short summary:
```
opkg update
opkg install luci
/etc/init.d/uhttpd enable
/etc/init.d/uhttpd start
ifconfig
```

Go to http://ip.address.assigned.by.dhcp/ and follow the instructions.

## Compiling

[Original guide](http://wiki.openwrt.org/doc/howto/build).

```
# Install the dependencies mentioned in the original guide and then continue from here:
git clone https://github.com/ljani/openwrt-yun.git
cd openwrt-yun
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
# And select Arduino Yún
make defconfig
make
ls bin/ar71xx
```

## TODO

- [ ] Cleanup the Yún patches
- [ ] Port/rewrite the Yún specific packages such as flashing sketches from the web UI

## Links

Discussion via [the Yún forum](http://forum.arduino.cc/index.php?topic=284023.0).

## Thanks

- [OpenWrt](https://openwrt.org)
- [OpenWrt Yún maintainers](https://github.com/arduino/openwrt-yun)
