# (Near) Mainline linux (and lk2nd) on XTC Z6 (imoo Z6)

msm8909w+pm8916 device, cmd mode panel

## Status

Just boots, there's ramoops dump available through lk2nd, see [Example ramoops console](example_ramoops_console.txt)

---

## Issues

- Display is not supported: msm8909w "is quite different from other Qualcomm SoCs (MDP3 instead of MDP4/MDP5) and needs a new driver in Linux. However, this only applies to devices where the display is connected via DSI" ([Postmarket Wiki](<https://wiki.postmarketos.org/wiki/Qualcomm_Snapdragon_210_(MSM8909)>))

- (More testing needed)

## Build & Run

## lk2nd

```bash
git clone https://github.com/msm8916-mainline/lk2nd --depth 1
cd lk2nd
# copy msm8909-i18.dts to lk2nd/device/dts/msm8909/
# edit lk2nd/device/dts/msm8909/rules.mk to include the dts file

# LK2ND_FORCE_FASTBOOT=1 is needed; it doesn't boot automatically correctly for some reason
# You can safely remove other arguments
make TOOLCHAIN_PREFIX=arm-none-eabi- DEBUG_FBCON=1 DEBUG=1 LK2ND_FORCE_FASTBOOT=1 lk2nd-msm8909 
# flash build-lk2nd-msm8909/lk2nd.img to boot
```

### Kernel

```bash
git clone https://github.com/msm8916-mainline/linux --depth 1 -b msm8916/6.6 # TODO: test other kernel version (up to 6.17)
cd linux
# copy qcom-msm8909-genius-i18.dts to arch/arm/boot/dts/qcom/
# modify arch/arm/boot/dts/qcom/Makefile to include the new device tree file
cat arch/arm64/configs/msm8916_defconfig arch/arm/configs/msm8916_defconfig.part > arch/arm/configs/msm8916_defconfig
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- CC=arm-linux-gnueabi-gcc HOSTCC=gcc O=.output msm8916_defconfig
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- CC=arm-linux-gnueabi-gcc HOSTCC=gcc O=.output
```

### boot.img

```bash
cat .output/arch/arm/boot/zImage .output/arch/arm/boot/dts/qcom/qcom-msm8909-genius-i18.dtb > zImage-dtb 
mkbootimg --kernel zImage-dtb --ramdisk ../initramfs.cpio.gz --cmdline "console=ttyMSM0,115200n8 console=tty0 earlycon clk_ignore_unused regulator.ignore_unused=1 pd_ignore_unused" --base 0x80008000 --pagesize 2048 -o boot.img
fastboot boot boot.img
```
