# WhateverXe — i5-12450H 48EU 专用方案

> [!WARNING]
> **当前只支持 60Hz。不要切换到 120Hz！**
>
> 切换到 120Hz、修改分辨率、显示睡眠、系统睡眠或合盖，都会让内屏黑屏/无背光，必须重启才能恢复。
>
> 原因：本方案使用 `-wex-edp-keep-loose` 保留 firmware 已经训练好的 **60Hz eDP link**。切换刷新率或分辨率会触发重新训练 eDP link，当前驱动路径不支持。

## 适用硬件

| 项目 | 值 |
| --- | --- |
| CPU | Intel Core i5-12450H（Alder Lake-H） |
| 核显 | Intel UHD Graphics，48 EU |
| PCI ID | `8086:46A3` |
| 内屏 | 内置 eDP，已验证 2880×1880 |
| macOS | Tahoe 26.x / Sequoia 15.8.x（0.7.0 支持范围） |
| 已验证结果 | 60Hz、Metal 2、VRAM 约 1536MB |

这是给 **i5-12450H / 48EU / `46A3`** 内屏机器用的，不要套用到其他 CPU 或其他核显配置。

## 下载

- WhateverXe.kext（本方案）：[v0.7.0-48eu-aperture](https://github.com/TigerShota/WhateverXe/releases/tag/v0.7.0-48eu-aperture)
- WhateverXe Graphics 0.4.0：[下载](https://github.com/b00t0x/WhateverXe-Graphics/releases/tag/v0.4.0)
- Lilu 1.7.3：[下载](https://github.com/acidanthera/Lilu/releases)
- AMFIPass 1.4.1 Mod：[下载](https://github.com/b00t0x/WhateverXe-Graphics/blob/main/extras/AMFIPass-v1.4.1-Mod-RELEASE.zip)

## 安装步骤

1. 解压 WhateverXe release，把 `WhateverXe.kext` 放到：
   ```text
   EFI/OC/Kexts/WhateverXe.kext
   ```
2. 用 ProperTree 或 OCAT 打开 `EFI/OC/config.plist`：
   - `Kernel > Add` 里添加 WhateverXe，确保排在 **Lilu 之后**
   - 不要加载 WhateverGreen
3. 设置 boot-args：
   ```text
   debug=0x100 keepsyms=1 -allow3d -disablegfxfirmware -igfxdvmt -wex-edp-keep-loose
   ```
4. OpenCore 设置：
   ```text
   UEFI > Output > Resolution = Max
   ```
5. 设置：
   ```text
   csr-active-config = 03080000
   ```
6. 如果 BIOS 的 DVMT Pre-Allocated 是 **60MB**，在 `DeviceProperties > Add` 的 `PciRoot(0x0)/Pci(0x2,0x0)` 下添加：

   | 属性 | 类型 | 值 |
   | --- | --- | --- |
   | `framebuffer-patch-enable` | Data | `01000000` |
   | `framebuffer-stolenmem` | Data | `0000C003` |

7. 启动 macOS，安装 **WhateverXe Graphics 0.4.0** pkg，然后重启。
8. 安装 **AMFIPass 1.4.1 Mod**，不要和上游 AMFIPass 同时加载。
9. 进入系统后关闭显示睡眠：
   ```bash
   sudo pmset -a displaysleep 0
   sudo pmset -a sleep 0
   ```

## 成功标志

- 系统信息里 Metal 支持正常
- VRAM 大约 1536MB（不是 19MB）
- Dock / 菜单栏有透明模糊
- 刷新率保持 **60Hz**

## 黑屏了怎么恢复

如果误切了 120Hz，或者改分辨率/睡眠后黑屏：

1. 强制关机再开机。firmware 通常会回到 60Hz。
2. 如果重启后 macOS 仍自动恢复 120Hz 并黑屏：
   - 用备份 EFI 启动，或
   - 在 boot-args 里临时加 `-wex-adlp-load-only` 启动，或
   - 进 OpenCore Recovery
3. 删除显示模式记忆：
   ```bash
   sudo rm -f /Library/Preferences/com.apple.windowserver*.plist
   sudo rm -f /var/db/com.apple.windowserver*.plist
   rm -f ~/Library/Preferences/ByHost/com.apple.windowserver*.plist
   rm -f ~/Library/Preferences/com.apple.windowserver*.plist
   ```
4. 重启，回到 60Hz。

## 已知限制

- **120Hz 不支持**：切换会黑屏，需要重启
- **修改分辨率不支持**：会黑屏，需要重启
- **显示睡眠 / 系统睡眠 / 合盖不支持**：会黑屏，需要重启
- 外接显示器、USB-C/HDMI 输出未验证
- 亮度调节未验证；需要时可在 OpenCore 加 `SSDT-PNLF`

## 来源与许可

本分支基于 [b00t0x/WhateverXe](https://github.com/b00t0x/WhateverXe) 0.7.0，合并了针对 48EU 内屏 eDP 的修复，包括：

- `nickk19/WhateverXe` `display13-edp-keep`
- `Thatisjigen` 的单输出 CDCLK 修复
- 上游 issue #13 / #22 / #35 的 eDP source、aperture 相关修复

License: BSD-3-Clause，保留原项目版权声明。这是一个非官方 fork，使用风险自负。
