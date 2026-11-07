# WhateverXe

[日本語](README_ja.md)

> **Fork focus — Intel Core i5-12450H, 48 EU (Alder Lake-H, `8086:46A3`)**
>
> This fork is dedicated to the 48-EU i5-12450H internal eDP configuration. It adds the VBT, eDP-source, port-mode, cold-CLCDK and 2880-wide aperture fixes required on this machine.
> Verified on a 2880×1880 eDP panel at 60 Hz with Metal 2, using WhateverXe Graphics 0.4.0 and `-wex-edp-keep-loose`.
>
> **本分支专用于 i5-12450H（Alder Lake-H / UHD Graphics 48 EU / PCI ID `0x46A3`）内屏 eDP 方案。**
> 已在 2880×1880 内屏、60 Hz、Metal 2 下验证；必须搭配 WhateverXe Graphics 0.4.0 和 `-wex-edp-keep-loose`。
> 120 Hz、修改分辨率、显示睡眠会黑屏，需要重启。

WhateverXe is a Lilu plugin, derived from WhateverGreen, that adapts Apple graphics drivers for selected Intel Xe-LP integrated GPUs.
Use the latest releases of [WhateverXe](https://github.com/b00t0x/WhateverXe/releases) and [WhateverXe Graphics](https://github.com/b00t0x/WhateverXe-Graphics/releases) together. See [CHANGELOG](CHANGELOG.md) for release notes.

### Verified i5-12450H 48-EU configuration

| Item | Value |
| --- | --- |
| CPU | Intel Core i5-12450H (Alder Lake-H) |
| iGPU | Intel UHD Graphics, 48 EU, PCI ID `8086:46A3` |
| Internal panel | 2880×1880 eDP |
| macOS | Tahoe 26.x / Sequoia 15.8.x |
| Graphics stack | WhateverXe Graphics 0.4.0 |
| Boot arguments | `-allow3d -disablegfxfirmware -igfxdvmt -wex-edp-keep-loose` |
| OpenCore | `UEFI > Output > Resolution = Max`, `csr-active-config=03080000` |
| Result | Metal 2, 60 Hz, internal panel |
| Not supported | 120 Hz, resolution changes, display/system sleep with the kept link |

The 60 Hz kept-link path is the working configuration. Changing the refresh rate or resolution retrains the eDP link and leaves the panel dark until restart.

## Compatibility

| macOS | Required version |
| --- | --- |
| Big Sur | 11.7.11 |
| Monterey | 12.7.6 |
| Ventura | 13.7.8 |
| Sonoma | 14.8.9 |
| Sequoia | 15.8.x |
| Tahoe | 26.7 or newer |

Sequoia and Tahoe accept compatible updates within their OS family. Future Apple changes may require an update to WEX or Graphics.

### Tested on hardware

- Tiger Lake-U: 48- and 80-EU configurations.
- Alder Lake-N: 16- and 24-EU configurations.
- Twin Lake: 32-EU configurations.
- Alder Lake-H: 48- and 96-EU configurations, including i5-12450H (`46A3`) 48 EU with internal eDP, and single-channel operation.
- Alder Lake-S: UHD Graphics 770.

See [tested features and known issues](#tested-features-and-known-issues) for OS coverage and display configurations. PCI IDs are listed in [device selection](docs/device-selection.md).

### Expected to work, not yet tested on hardware

- Other supported Tiger Lake-U/H35 configurations, including 96 EU, and Tiger Lake-H45.
- Other supported Alder Lake-N / Twin Lake configurations.
- Other supported Alder Lake-H/P/U and Raptor Lake-H/P configurations.
- Other supported Alder Lake-S/HX and Raptor Lake-S/HX configurations.
- Internal eDP on Alder Lake-N / Twin Lake and Alder Lake-H/P/U / Raptor Lake-H/P is implemented; on hardware it has been tested only with the firmware's panel link kept (`-wex-edp-keep` or `-wex-edp-keep-loose`, see [other settings](#other-settings)).

### Further implementation required

- Raptor Lake-U: dedicated display-clock handling is not implemented.
- Rocket Lake-S: display control requires additional support.

### No support planned

- Xe-LPG, including Meteor Lake integrated graphics.
- Xe2 and later graphics architectures.

## Installation

1. Add Lilu, `WhateverXe.kext` and [AMFIPass 1.4.1 Mod](https://github.com/b00t0x/WhateverXe-Graphics/blob/main/extras/AMFIPass-v1.4.1-Mod-RELEASE.zip) to OpenCore, with WEX after Lilu. Use the WEX **RELEASE** ZIP; DEBUG is for troubleshooting.
2. Set `csr-active-config=03080000` (`0x803`) and apply the [configuration](#configuration).
3. Boot the target macOS, install the latest [WhateverXe Graphics pkg](https://github.com/b00t0x/WhateverXe-Graphics/releases), then restart. [Package installation and removal](https://github.com/b00t0x/WhateverXe-Graphics#install).

Use the linked **Mod** AMFIPass instead of upstream AMFIPass; do not load both. WhateverGreen and WhateverXe must not be enabled together. `amfi=0x80` and `ipc_control_port_options=0` are unnecessary.

Graphics installs under `/Library` and rebuilds AuxKC without patching System files or snapshots. Restore an old Xe System patch before installing. If you need OCLP for other root patches, use the [custom OCLP build](https://github.com/b00t0x/OpenCore-Legacy-Patcher) with the matching Mod signing chain.

## Configuration

### Required boot arguments

Add to OpenCore's `boot-args`:

```text
-allow3d -disablegfxfirmware
```

### DVMT

Set **DVMT Pre-Allocated to 96MB** in BIOS/UEFI where possible.
If it is fixed at **60MB**, also add `-igfxdvmt` and set these iGPU properties under `DeviceProperties > Add` (usually `PciRoot(0x0)/Pci(0x2,0x0)`):

| Property | Type | Value (hex) |
| --- | --- | --- |
| `framebuffer-patch-enable` | Data | `01000000` |
| `framebuffer-stolenmem` | Data | `0000C003` |

These settings are unnecessary at 96MB and do not increase firmware-allocated memory.

### Other settings

- Add `SSDT-PNLF` for internal-display brightness control.
- GPU selection is automatic; a `device-id` override is normally unnecessary.
- Alder Lake-N / Twin Lake and Alder Lake-H/P/U / Raptor Lake-H/P select tiling automatically. Do not use `-yTileDisable`; `AAPL,GfxYTile` is unnecessary.
- If an Alder Lake-H/P/U / Raptor Lake-H/P internal panel goes dark when macOS takes over the boot screen, add `-wex-edp-keep` and set OpenCore `UEFI > Output > Resolution` to `Max`. WEX then keeps the panel link the firmware trained instead of training it again. Display off/on is not supported in this mode, so disable display sleep.
- For the 48-EU i5-12450H 2880×1880 panel, use `-wex-edp-keep-loose` (it implies `-wex-edp-keep`). It keeps the firmware link even when the donor's lane count or bpc differs. Keep `Resolution = Max` and disable display sleep. The working configuration is 60 Hz; 120 Hz retrains the link and is not supported yet.

## Tested features and known issues

The common regression checks cover Metal, OpenGL, OpenCL, hardware H.264 encode/decode and HEVC decode. The table summarizes OS coverage and configuration-specific display features.

| Configuration | Verified coverage |
| --- | --- |
| Tiger Lake mini PCs | Big Sur–Tahoe: 80 EU, two 1080p60 displays, display off/on, Safari video and Widevine |
| Tiger Lake laptops | Big Sur–Tahoe: 48 EU, internal eDP and brightness, HDMI/USB-C, up to three displays and display off/on |
| Alder Lake-N mini PCs | Big Sur–Tahoe: 16/24 EU, two 1080p60 HDMI displays and display off/on |
| Alder Lake-N / Twin Lake laptops | MIPI DSI on MicroPC 2 (N250) and MiniBook X N100. Big Sur: brightness, rotation, display off/on and external HDMI/USB-C coexistence; MicroPC 2 also has HiDPI and up to three displays. Tahoe: MicroPC 2 internal/external displays, brightness, rotation and HiDPI |
| Alder Lake-H | Big Sur–Tahoe: 96 EU, three displays using HDMI and a DP-to-HDMI adapter. Single-channel `4626` graphics acceleration checked on Big Sur |
| Alder Lake-H laptops | Tahoe: 48 EU i5-12450H (`46A3`), 2880×1880 eDP at 60 Hz with `-wex-edp-keep-loose`, Metal 2. 120 Hz, resolution changes and display sleep are unsupported. Also 48 EU internal 1920×1200 OLED eDP with `-wex-edp-keep` |
| Alder Lake-S | Big Sur–Tahoe: UHD 770, HDMI/DP/USB-C and up to three displays; Safari video, with Widevine checked on Big Sur/Tahoe |

MIPI DSI support is **experimental and tested only on GPD MicroPC 2 and CHUWI MiniBook X N100**. Do not expect it to work on other devices.

### Known issues and limits

- DisplayPort reconnection can cause black screens or freezes. Tiger Lake USB-C reconnection is also unreliable; Alder Lake-S DP output/reconnection can be slow.
- Some image-conversion operations can crash applications; App Store video can freeze the GPU on Sequoia.
- Safari video may occasionally drop frames. FairPlay-protected playback is unsupported.
- Up to three displays are supported. Resolutions beyond the tested configurations and full system sleep/wake remain unverified.
- With `-wex-edp-keep` or `-wex-edp-keep-loose`, display sleep, a resolution or refresh-rate change, or system sleep leaves the internal panel dark until restart.
- Some connector layouts and backlight methods, MST and Thunderbolt-managed outputs are unsupported.

### Roadmap

- Implement Raptor Lake-U display-clock support.
- Complete DisplayPort, USB-C and internal-display support on Alder Lake-N / Twin Lake and Alder Lake-H/P/U / Raptor Lake-H/P.
- Improve display reconnection reliability and investigate full system sleep/wake.

## Build and test

Requires macOS, Xcode command-line tools, Python 3.9+, MacKernelSDK and Lilu 1.7.2 development headers/library. Place `MacKernelSDK/` and `Lilu.kext/` in the repository root, or pass `--sdk` and `--lilu`.

```sh
python3 tools/build.py --release
python3 tools/test.py
```

Output is under `build/`. AppleIntelTGLCompat is built here and distributed through Graphics. Optional exact-donor tests accept `--donor` and `--accelerator` paths.

## License and credits

WhateverXe retains WhateverGreen's copyright notices and BSD-3-Clause license.
See [LICENSE.txt](LICENSE.txt). Lilu and MacKernelSDK are separate dependencies.

Thanks to [JulioNectar/XPS13Plus](https://github.com/JulioNectar/Hackintosh-XPS13Plus-IrisXe-Tahoe) for sharing its Xe graphics work. Its compiler adapter and LLVM bridge informed the OS-native AIR conversion used in WhateverXe Graphics.

Internal-panel work also draws on [WhateverXe-RaptorLakeP](https://github.com/samranahmad09/WhateverXe-RaptorLakeP), [X1Carbon-G11](https://github.com/dssoftx/Hackintosh-X1Carbon-G11-IrisXe-Tahoe), XPS13Plus and the [Linux i915 display sources](https://github.com/torvalds/linux/tree/v6.12/drivers/gpu/drm/i915/display) for hardware behavior and implementation comparisons. Keeping the firmware's panel link follows [NootedGreen](https://github.com/sgiammori/NootedGreen)'s firmware-link mode and Linux i915 fastboot.
