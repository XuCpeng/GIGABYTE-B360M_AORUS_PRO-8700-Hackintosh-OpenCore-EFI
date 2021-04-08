# GIGABYTE-B360M_AORUS_PRO-8700-Hackintosh-OpenCore-EFI

技嘉 B360M AORUS PRO / i7-8700 UHD630 / macOS 11.0 Big Sur / macOS 10.15 Catalina / 黑苹果 OpenCore EFI / GIGABYTE B360M AORUS PRO Hackintosh OpenCore EFI

## 使用方法

> 此EFI已尽量保持纯净，没有添加任何怪异的补丁，只添加必要的配置。OC最近几个版本的配置项变动非常频繁，有大量的增删，非常不建议直接把旧的config.pdist复制过来用。

1. 从 `EFI/OC/` 文件夹中选择合适的配置文件并将其重命名为 `config.pdist` 即可
2. 默认开启了日志输出，系统安装、调试完成后可关闭日志（见文末）

## 文件详情

OC版本：OpenCore-0.6.7-RELEASE

- `config-NoAWAC-WiFiBT.plist`: 无AWAC补丁，有Wifi和蓝牙kext（默认为该配置文件）
- `config-AWAC-WiFiBT.plist`: 有通用AWAC补丁，有Wifi和蓝牙kext
- `config-NoAWAC-NoWiFiBT.plist`: 无AWAC补丁，无Wifi和蓝牙kext
- `config-AWAC-NoWiFiBT.plist`: 有通用AWAC补丁，无Wifi和蓝牙kext

**建议 GIGABYTE B360M AORUS PRO 使用“无AWAC补丁 ”的版本，原因如下文2。**

1. 使用 [AirportItlwm.kext](https://github.com/OpenIntelWireless/itlwm) 和 [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) 来驱动CNVi无线网卡蓝牙模块 —— AC9560。
2. **为什么删除SSDT-AWAC.aml**：几乎网上所有教程都说300系主板一定需要该补丁，但我在 `技嘉 B360M AORUS PRO` 通过SSDTime发现其并没有AWAC时钟，不需要该补丁。而且更重要的是AWAC补丁会导致BIOS时间不更新，进而造成Win时间不走（非时区问题）。如果之前使用过AWAC，而且出现了win下时间不走的问题，只能扣主板电池重置BIOS。
3. 通过删除config中的 `PciRoot(0x0)/Pci(0x2,0x0)`下的`AAPL,slot-name`,实现了HD630 4k HEVC加速。

## 硬件配置

- 已通过启动参数`-wegnoegpu`屏蔽独显，只驱动核显，使用主板接口输出
- 强烈建议使用DP输出

---

- 引导： OpenCore-0.6.7-RELEASE
- 系统： macOS 11.2.3 Big Sur
- CPU： i7-8700 睿频正常
- 集显：UHD630 1536MB 支持4k HEVC加速
- 内存：4*8G 2666MHZ 正常
- 有线网卡：正常
- 无线网卡+蓝牙：AC9560 CNVi模块，正常
- 音频：正常
- USB：正常
- 睡眠：正常

![硬件参数](readme_images/硬件参数.png)

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

> boot item: BOOT/BOOTx64.efi

## 关闭 SIP 和 authenticated-root (可选)

以下方法二选一

### 1. 通过原生Recovery（推荐）

从opencore的启动菜单中进入Recovery，打开实用工具-终端，执行下面两句命令：

关闭SIP:

```shell
csrutil disable
```

关闭 authenticated-root：

```shell
csrutil authenticated-root disable
```

重启即可。

### 2. 通过修改config.pdist（不推荐）

若无法通过opencore启动菜单进入Recovery，可使用配置项关闭SIP，但**有可能**无法关闭authenticated-root。

参考：[disabling-sip](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html#disabling-sip)

通过设置`config.plist`中的`csr-active-config`来开启或关闭SIP，默认为`00000000`。

```shell
NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> csr-active-config
```

- `00000000` - 完全开启 SIP。SIP completely enabled (0x0).
- `03000000` - 关闭 kext 签名和文件系统保护。Disable kext signing (0x1) and filesystem protections (0x2).
- `FF030000` - 关闭 High Sierra 的 SIP。 Disable all flags in macOS High Sierra (0x3ff).
- `FF070000` - 关闭 Mojave 和 Catalina 的 SIP。Disable all flags in macOS Mojave and in macOS Catalina (0x7ff) as Apple introduced a value for executable policy.
- `FF0F0000` - 关闭 Big Sur 的 SIP。Disable all flags in macOS Big Sur (0xfff) which has another new flag for authenticated root.

## 关闭所有日志输出

- Misc -> Debug -> AppleDebug = NO
- Misc -> Debug -> ApplePanic = NO
- Misc -> Debug -> Target = 0
- NVRAM -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args: 删除 `-v debug=0x100` 等参数

## 截图

![关于本机](images/../readme_images/关于本机.png)

![图形卡显示器](images/../readme_images/图形卡显示器.png)

![UHD630 4K HEVC](images/../readme_images/UHD630%204K%20HEVC.png)

![以太网卡](images/../readme_images/网络.png)

![Intel Power Gadget](images/../readme_images/Intel%20Power%20Gadget.png)

![USB](images/../readme_images/USB.png)
