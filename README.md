# GIGABYTE-B360M_AORUS_PRO-Hackintosh-OpenCore-EFI
技嘉 B360M AORUS PRO / i7-8700 UHD630 / BigSur / 黑苹果 OpenCore EFI / GIGABYTE B360M AORUS PRO Hackintosh OpenCore EFI

## 硬件配置


## BIOS 设置

参考：[intel-bios-settings](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#intel-bios-settings)

### Disable
- Fast Boot
- Secure Boot
- Serial/COM Port
- Parallel Port
- VT-d (can be enabled if you set DisableIoMapper to YES)
- CSM
- Thunderbolt(For initial install, as Thunderbolt can cause issues if not setup correctly)
- Intel SGX
- Intel Platform Trust
- CFG Lock (MSR 0xE2 write protection)(This must be off, if you can't find the option then enable both AppleCpuPmCfgLock and AppleXcpmCfgLock under Kernel -> Quirks. Your hack will not boot with CFG-Lock enabled)

### Enable
- VT-x
- Above 4G decoding
- Hyper-Threading
- Execute Disable Bit
- EHCI/XHCI Hand-off
- OS type: Windows 8.1/10 UEFI Mode
- DVMT Pre-Allocated(iGPU Memory): 64MB
- SATA Mode: AHCI

## 截图

![关于本机](images/../readme_images/关于本机.png)

![图形卡显示器](images/../readme_images/图形卡显示器.png)

![以太网卡](images/../readme_images/以太网卡.png)

![音频](images/../readme_images/音频.png)

![Intel Power Gadget](images/../readme_images/Intel%20Power%20Gadget.png)

![USB](images/../readme_images/USB.png)