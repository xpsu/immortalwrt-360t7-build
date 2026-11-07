# immortalwrt-360t7-build

用 GitHub Actions 云编译 **奇虎 360T7** 的 ImmortalWrt 固件（PadavanOnly 系，闭源 `mt_wifi` 无线驱动）。

## 参数

| 项目 | 值 |
|---|---|
| 源码 | `padavanonly/immortalwrt-mt798x-6.6` |
| 分支 | `openwrt-24.10-6.6` |
| 锁定提交 | `ec9ef10` |
| 设备 | `qihoo_360t7`（UBI @0x580000，108MB 布局） |
| 无线 | 闭源 `mt_wifi` + `mtkhnat` 硬件加速 |
| 插件 | 上游 defconfig 全套，**去掉 ksmbd，加上 upnp** |
| 主题 | **Argon**（`luci-theme-argon` + `luci-app-argon-config`，默认主题已通过 uci-defaults 固定） |

## 触发

1. 仓库 **Actions** → `Build ImmortalWrt 360T7` → **Run workflow**
2. 勾选 `dry_run` 可只做配置校验（约 8 分钟，不编译）
3. 正式编译约 **90–150 分钟**
4. 产物在 **Artifacts**（保留 30 天）和 **Releases**（长期）

## 编译产物

```
immortalwrt-mediatek-filogic-qihoo_360t7-squashfs-sysupgrade.bin
SHA256SUMS.txt
config.buildinfo
```

## 刷机

```bash
# 备份
ssh root@192.168.6.1 "sysupgrade -b /tmp/backup.tar.gz"
scp root@192.168.6.1:/tmp/backup.tar.gz .

# 刷写
scp immortalwrt-mediatek-filogic-qihoo_360t7-squashfs-sysupgrade.bin root@192.168.6.1:/tmp/
ssh root@192.168.6.1 "cd /tmp && sha256sum *.bin && sysupgrade -v *.bin"
```

或在 LuCI：**系统 → 备份/升级 → 刷写新固件**。

> 跨内核小版本升级建议**取消勾选"保留配置"**，刷完重新配；或者保留但先备份。

## 改配置

只改 `configs/360t7.extra`，push 后自动重新编译。

常用包名示例：

```conf
CONFIG_PACKAGE_luci-app-upnp=y
CONFIG_PACKAGE_luci-app-ddns=y
CONFIG_PACKAGE_luci-app-argon-config=y
CONFIG_PACKAGE_luci-app-openclash=y
```

未定义的包名会被 kconfig 静默忽略，改完看 Actions 日志里 `=========== 无线 / 插件 ===========` 那一段确认生效。

## 注意

- 上游 `defconfig/mt7981-ax3000.config` 是从 21.02 仓库继承的老文件，里面 `mt7981-360-t7-108M` 等机型在 6.6 分支**并不存在**，subtarget 符号也写成了 `mediatek_mt7981`。workflow 里已经自动修正，别直接拿它当 `.config` 用。
- 不要 `opkg upgrade` 内核模块，会破坏闭源无线驱动，只能整包 sysupgrade。
