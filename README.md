# android_kernel_xiaomi_spinel

Prebuilt (GKI) kernel tree for **Redmi Note 15 4G** — codename `spinel`, MediaTek **MT6789** (Helio G100 Ultra).

> Path B (prebuilt) as defined in the android-device-tree-builder workflow: no `spinel` branch has been
> published in Xiaomi's kernel-opensource repos yet, so the stock boot images are kept as opaque prebuilt
> binaries. This boots a custom Android userspace immediately; re-check
> `git ls-remote --heads https://github.com/MiCode/Xiaomi_Kernel_OpenSource.git | grep -i spinel`
> periodically to migrate to Path A (real kernel source) when it lands.

## Kernel identity

| Field | Value |
|---|---|
| Version | `5.10.236-android12-9-00020-gf997514b333d-ab13743839` |
| GKI | Yes — Android Common Kernel 5.10 (android12-9 branch), generic boot ramdisk (`GSI on ARM64`) |
| Build date | Mon Jul 7 16:32:37 UTC 2025 |
| Toolchain | Android clang 12.0.5 (LLD 12.0.5) |
| Arch | arm64 (ARM64 boot executable Image, little-endian, 4K pages) |
| Platform | MT6789 (MediaTek Helio G100 Ultra) |
| A/B device | Yes (virtual A/B), Treble: yes |
| Target build | `missi-user 15 AP3A.240905.015.A2 OS2.0.212.0.VPGMIXM` |

Kernel cmdline (from stock `vendor_boot.img`):
```
bootopt=64S3,32N2,64N2
```

Boot image layout (verified by direct parse of stock images):
- `boot.img` — Android boot image **header v4**, kernel only (19.6 MB gzipped), ramdisk = generic GKI boot ramdisk, `dtb_size=0`, no signature.
- `vendor_boot.img` — Android vendor_boot **header v4**, page size 4096, single 32 MB vendor ramdisk (LZ4), `dtb_size=0`. cmdline: `bootopt=64S3,32N2,64N2`.
- `dtbo.img` — DTBO table with a single entry (ID 0, rev 0): the `spinel` device-tree overlay (~58 KB, big-endian table header), applied on top of the MTK base DTB by the bootloader.

## Prebuilt files

| File | Origin | Purpose |
|---|---|---|
| `prebuilt/Image.gz` | extracted from stock `boot.img` | `TARGET_PREBUILT_KERNEL` |
| `prebuilt/dtbo.img` | stock partition image | `BOARD_PREBUILT_DTBOIMAGE` |
| `prebuilt/vendor_boot.img` | stock partition image | `TARGET_PREBUILT_VENDOR_BOOTIMAGE` |
| `prebuilt/spinel.dtb` | extracted DTB from `dtbo.img` (reference) | contains `foursemi,fs1815` audio amp + `aw87xxx` overlays |

`Image.gz` sha1 (verified against dump): `3108216b350775731495f7c2be6b8e7849c1efeb`

## Usage (device tree `BoardConfig.mk`)

```makefile
TARGET_PREBUILT_KERNEL := $(TARGET_KERNEL_SOURCE)/prebuilt/Image.gz
BOARD_PREBUILT_DTBOIMAGE := $(TARGET_KERNEL_SOURCE)/prebuilt/dtbo.img
BOARD_INCLUDE_DTB_IN_BOOTIMG := true
TARGET_PREBUILT_VENDOR_BOOTIMAGE := $(TARGET_KERNEL_SOURCE)/prebuilt/vendor_boot.img
```

The stock kernel has no embedded DTB; the device-specific DTB comes from `dtbo.img`, so `BOARD_INCLUDE_DTB_IN_BOOTIMG` should stay `true` for the boot image repack to be consistent with stock behavior.
