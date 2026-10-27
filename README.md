# OpenEUICC for Android 11 (API 30)

> **非官方构建。** 本仓库与 [estkme-group/openeuicc](https://github.com/estkme-group/openeuicc)、
> PeterCxy / gitea.angry.im 无任何隶属或授权关系。

给 **Android 11（API 30）** 设备用的 [OpenEUICC](https://github.com/estkme-group/openeuicc) 适配构建。

只做了两处 SDK 守卫，**没有改动任何业务逻辑**。

---

## 为什么需要这个

上游 OpenEUICC 的 `app/build.gradle.kts` 里 `minSdk = 31`，至今未变。这意味着：

- **Android 11（API 30）设备装不上** —— 安装器直接返回 `INSTALL_FAILED_OLDER_SDK`；
- 即便强行装进去，启动时也会崩在 `NoSuchMethodError: getRoundedCorner` ——
  这不是配置问题，是代码真的调用了 API 31 才引入的方法。

上游三个第三方 Magisk 分支里，两个已归档
（[AndroPlus-org](https://github.com/AndroPlus-org/magisk-module-openeuicc)、
[liuran001](https://github.com/liuran001/OpenEUICC_Magisk_Module)），
一个已停更（[hzy132](https://github.com/hzy132/OpenEUICC_for_Magisk)，2026-09）。

**"给 Android 11 用的 OpenEUICC" 这个位置目前是空的。** 本仓库补的就是这个缺口。

> 顺带说明：**Android 12+（API ≥31）请直接用上游官方构建**，你不需要这里的东西。

---

## 这是什么

一个打了补丁的 OpenEUICC APK，以及配套的 Magisk 模块，用于把 OpenEUICC 作为
**系统级 priv-app** 装上，让它拿到所需的 eUICC 特权。

### 补丁内容

两处 `smali` 层 SDK 守卫，均由 `apktool` 反编译上游 APK 后手工修改：

| 补丁 | 位置 | 改动 |
|:--|:--|:--|
| **A** | `androidx/core/view/WindowInsetsCompat$Impl20.createDisplayShape` | `SDK_INT < 31` 时返回 `null` |
| **B** | `androidx/core/view/DisplayCompat.getRoundedCorner` | `SDK_INT < 31` 时返回 `null` |

两处返回值本来就是调用方**必须处理 null** 的分支——所以我们只是提前走到那个分支，
没有改变任何控制流语义。

完整 diff 见 [`patches/`](patches/)。

### 上游版本

- 仓库：`estkme-group/openeuicc`
- 提交：`055d1e2`（2026-10-26，"feat: Profile download API via EuiccService (#366)"）
- 版本号：`v1.7.2-6-g055d1e2`
- 许可证：GPL-3.0（本构建继承同一许可）

---

## 安装

### 前置条件

1. **已 root 的设备**（Magisk / KernelSU）——本方式安装的是系统 priv-app，需要写 `/system`。
2. **Android 11（API 30）**。其他版本请用上游官方构建。
3. 已卸载设备上任何旧版 OpenEUICC（见下方「故障排查」）。

### 步骤

1. 下载本仓库 Releases 里的 `openeuicc-android11-magisk.zip`；
2. Magisk → 模块 → 从本地安装 → 选择该 zip → 重启；
3. 重启后桌面上应出现 OpenEUICC 图标；
4. 首次打开会请求若干特权，全部允许。

### 校验

下载后请核对文件哈希（见对应 Release 说明），并核对签名证书：

```
DN:      CN=OpenEUICC, O=YanYue, C=CN
SHA-256: 9BC8FC6261994CC3D8E7F41B166A3E4517AA75B1432C898BE1450C2303748E68
SHA-1:   D14BFBF274B20697E99A70D9FBFEA8DB8085624A
```

> ⚠️ 本 APK 使用**独立签名**，与上游官方 APK 签名不同。
> 这意味着它**不能覆盖安装**上游版本，也**不能通过应用商店升级**。

---

## 故障排查

### 启动报 `Error type 3` / 应用信息里 `restricted=true`

原因：`customize.sh` 里的 `pm install -r` 会在 `/data/app` 留一份副本，
导致系统看到两个 codePath、判定为受限应用。这是上游模块的既有设计，非本补丁引入。

处理：

```bash
adb shell pm uninstall im.angry.openeuicc
# 然后重启
```

### 切卡后 modem 不认新卡

部分机型（包括 Hisense A9 / HLTE556N）的 modem 固件不会在切卡后刷新。
**重启一次即可。** 这是机型问题，不是 LPA 缺陷。

### 装完模块后 Magisk 提示有更新

本仓库已移除模块内的 `updateJson` 字段（上游原值指向
`https://openeuicc.com/magisk/magisk-debug.json`）。

> 🔴 **这一点很重要**：若保留该字段，Magisk 会自动从上游拉取**未打补丁**的新版，
> 把 Android 11 适配整个覆盖掉，升级后立刻回到「装不上 / 闪退」。
> 如果你从别处拿到本模块的旧版本，请检查 `module.prop` 里是否还有这一行。

---

## 已知限制

| 限制 | 说明 |
|:--|:--|
| 仅 API 30 | 补丁只针对 Android 11 缺失的 API 做降级，不保证 Android 10 及以下可用 |
| 需独立签名 | 与上游不兼容，无法互相覆盖安装 |
| 一次性构建 | 见下方「关于签名」 |
| 不提供 Xesim | 本仓库不分发任何闭源第三方 APK |

---

## 关于签名

本构建使用的密钥库密码为公开值（`yanyue123`，alias `oeuicc`），
**目的是让将来任何人都能用同一把钥匙签发新版本**，从而使已安装用户能正常升级。

若你不信任公开密码的密钥库，可以选择只把本仓库的 APK 当作**一次性构建**使用，
不做版本升级。两种用法都成立，但请**不要**混用——
混用会导致签名不一致，安装时直接失败。

---

## 适用范围与免责

- **本构建是非官方产物**，与上游项目及其作者无关；
- 修改 eUICC 相关系统组件存在使设备 eSIM 功能异常的风险，**请自行评估**；
- 本项目按 GPL-3.0 「AS IS」提供，不附带任何担保；
- 若你的设备在保修期内，刷入系统级模块可能影响保修。

---

## 许可证

本构建基于 GPL-3.0 授权的上游代码衍生物，整体以 **GPL-3.0** 发布，
详见 [LICENSE](LICENSE)。

上游版权归 estkme-group / PeterCxy 及其贡献者所有。
本仓库的补丁与文档部分版权归 yanyue-dev 所有，同样以 GPL-3.0 授权。

---

## 相关

- 上游项目：<https://github.com/estkme-group/openeuicc>
- 上游原始仓库：<https://gitea.angry.im/PeterCxy/OpenEUICC>
- 背景：[`docs/background.md`](docs/background.md) —— 这个构建是怎么来的

---

<p align="center"><sub>Made for Android 11 users who were left behind by <code>minSdk = 31</code>.</sub></p>
