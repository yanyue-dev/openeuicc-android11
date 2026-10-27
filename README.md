# OpenEUICC for Android 11 (API 30) — 5ber eSIM 卡的免费替代方案

> **非官方构建。** 本仓库与 [estkme-group/openeuicc](https://github.com/estkme-group/openeuicc)、
> PeterCxy / gitea.angry.im 无任何隶属或授权关系。

**关键词**：5ber · 5ber eSIM · 5ber 跑路 · 5ber app 无法登录 · iFree · 5ber 替代方案 ·
eSIM 转实体卡 · 可移动 eSIM · OpenEUICC · EasyEUICC · lpac · Android 11 · 免 root

---

## 你可能是因为这些原因找到这里的

### 如果是「5ber 的 app 登不上了」

那你的卡大概率还是好的。往下读 [5ber 用户专区](#5ber-用户专区--你的卡没坏)。

### 如果是「Android 11 装不上 OpenEUICC」

那是 `minSdk = 31` 挡的。本仓库给了打好补丁的构建 —— 但**先看下面这一节**，
你可能根本不需要 root。

---

## ⚡ 免 root 方案（先看这个）

**大多数人不需要 root。** 本仓库的 Magisk 模块只是「已 root 用户的省事路径」，
不是唯一路径。

### 方案 A：EasyEUICC —— 官方非特权版，直接当普通 app 装

OpenEUICC 项目本身就提供**免 root 变体**，包名 `im.angry.easyeuicc`：

| 项 | 值 |
|:--|:--|
| 名称 | **EasyEUICC** |
| `minSdk` | **28**（Android 9+，**比 OpenEUICC 的 31 宽容得多**） |
| 安装方式 | **普通 APK，直接安装，不需要 root** |
| 官方下载 | <https://gitea.angry.im/PeterCxy/OpenEUICC/releases>（选 `unpriv-v*` 那个 tag） |

**⚠️ 前提**：非特权 app 要访问 eUICC 芯片，卡的 **ARA-M 里必须包含该 app 的签名证书哈希**。

上游给厂商的说明：希望自家卡被官方 EasyEUICC 支持，需在 ARA-M 里加入
`2A2FA878BC7C3354C2CF82935A5945A3EDAE4AFA`。

> **关于 5ber 卡**：我们实测到 `/data/misc/omod/` 为空，据此**推断** EasyEUICC 大概率无法访问它。
> 但**没有在 5ber 卡上直接跑过 EasyEUICC 验证**——这一条是推断，不是实测结论。
> 如果你手上有 5ber 卡和一台未 root 的手机，**这是最值得先试的一条路**：
> 装 EasyEUICC 试 30 秒就知道结果，成本几乎为零。
> 若有效请开 issue 告知，我会把结论改成实测。

### 方案 B：USB CCID 读卡器 ✅ 完全免 root，但需要硬件

EasyEUICC 支持通过 **USB CCID 读卡器**访问 eSIM 芯片 —— 这条路上
**ARA-M 限制完全不适用**，任何符合 SGP.22 标准的卡都能读。

- 需要：一个 CCID 读卡器 + 对应的 eSIM 卡转接器（几十元）
- 优点：不碰系统、不 root、不动手机
- 缺点：切卡要把卡插到读卡器上（手机支持 OTG 则可直连）

### 方案 C：Mac / PC + lpac / EasyLPAC ✅ 完全免 root

- [lpac](https://github.com/estkme-group/lpac)（740★，跨平台 LPA 命令行）
- [EasyLPAC](https://github.com/creamlike1024/EasyLPAC/releases)（lpac 的图形界面）
- 需要同一个 CCID 读卡器

### 方案 D：Xesim —— 闭源，但实测免 root 可用 ⚠️

5ber 原技术团队的转身之作（`com.elink.xesim`），**普通 app 装即可，不需要 root**。

**实测（2026-10-27，Android 11）**：无需登录，直接识别 5ber 卡，
能管理卡上已有 profile（切换/启用/禁用/删除），**完全免费**。

- 局限：下载新 profile 要买次数（约 $5/10 次），且依赖 `api.lpas.xesim.net`
- 权限审查：无 SMS / 通讯录 / BOOT_COMPLETED / INSTALL_PACKAGES，只有相机、位置、悬浮窗等常规项

> ⚠️ 闭源、依赖第三方服务器 —— 和 5ber 是同一类风险模型，只是目前还活着。
> 什么时候它也跑了，你就又需要一次今天这样的迁移。

> 📌 **一个尚未查清的疑点**：Xesim 同为非特权 app，却实测能访问 5ber 卡；
> 而按 ARA-M 规则，非特权 app 本不该能访问。二者如何对上，我们没查清，
> 也不作猜测。这不影响 Xesim 可用这个事实，但意味着「非特权应用能否访问 5ber 卡」
> 这个问题本身还没定论 —— 也正因如此，方案 A 值得你先花 30 秒试一次。

### 什么时候才真的需要 root（也就是本仓库的 Magisk 模块）

- 你**不想**用闭源的 Xesim，**且**
- 你没有读卡器，**且**
- 你就是想在手机上直接、开源地把卡管起来

⇒ 三条同时成立，才需要 root + 本仓库模块。**否则不必 root。**

---

## 5ber 用户专区 —— 你的卡没坏

### 发生了什么

5ber 是一个把 eSIM 写进**可移动物理卡**的产品，让没有 eSIM 功能的老手机也能用 eSIM。
它的管理 app（`com.ifreegroup.moesim`）负责把运营商 profile 下载、写入、启停卡上的 eUICC 芯片。

2026 年，母公司 iFree 资金链断裂，`api.5ber.com` 关闭。于是：

| 你的东西 | 状态 |
|:--|:--|
| 卡本身（物理） | ✅ **好的** |
| 已写进卡里的 profile | ✅ **还在卡上** |
| 卡里的 eSIM 套餐 | ✅ **还能正常用**（打电话/上网不受影响） |
| 管理 app（`com.ifreegroup.moesim`） | ❌ **登不进去，废了** |

**核心事实：服务器只管登录、配额计数、订单。写卡能力本来就在本地。**
所以卡没坏，你只是失去了一个「遥控器」。

### 你能做什么

| 你的情况 | 走哪条路 |
|:--|:--|
| 只想管已有 profile（启停/切换） | **Xesim**（普通 app，免费，已实测）或 **读卡器 + EasyLPAC** |
| 还想下载新 profile | **读卡器 + lpac/EasyLPAC**（无配额限制）或 **Xesim**（付费次数） |
| 想要开源 + 不 root | **EasyEUICC**（先花 30 秒试，见方案 A）或 **读卡器 + lpac** |
| 想要开源 + 只在手机上 + 不用读卡器 | 只能 root → 本仓库的 Magisk 模块 |
| 其他品牌可移动 eSIM（9ESIM/eSTK 等） | **EasyEUICC 普通 app 即可** |

### 为什么本仓库选 OpenEUICC 而不是改 5ber app

- 5ber app 的写卡能力全在本地原生插件（OMAPI 直连芯片），服务器只做鉴权与计数；
- 登录逻辑编译在 Flutter 的 `libapp.so` 里，没有可靠的 patch 手段；
- ⇒ **换一个开源 LPA 就行，卡还是那张卡。**

### 已知的 5ber 卡特性（踩坑表）

- **启用逻辑非标准**：lpac 曾专门修过（`es10c_enable_profile_iccid` 需 `RefreshFlag=0`）。OpenEUICC 新版已处理。
- **固件会放大 profile**：写入的 profile 比标准大约 10–15 KB。
- **切卡后 modem 可能不刷新**：部分机型需重启才认新卡（机型 modem 固件问题，非 LPA 缺陷）。
- **ARA-M 疑似不放行第三方**：`/data/misc/omod/` 为空。但我们只据此推断，
  并未在 5ber 卡上实跑 EasyEUICC 验证（Xesim 同为非特权却能访问，此处存在疑点，详见方案 D）。

---

## 本仓库提供什么

给 **Android 11（API 30）** 设备用的 [OpenEUICC](https://github.com/estkme-group/openeuicc) 适配构建。

上游 `app/build.gradle.kts` 里 `minSdk = 31`，至今未变：

- **Android 11 装不上** —— 安装器返回 `INSTALL_FAILED_OLDER_SDK`；
- 强装后崩在 `NoSuchMethodError: getRoundedCorner` ——
  不是配置问题，是代码真的调用了 API 31 才引入的方法。

上游三个第三方 Magisk 分支里两个已归档
（[AndroPlus-org](https://github.com/AndroPlus-org/magisk-module-openeuicc)、
[liuran001](https://github.com/liuran001/OpenEUICC_Magisk_Module)），
一个已停更（[hzy132](https://github.com/hzy132/OpenEUICC_for_Magisk)，2026-09）。

**"给 Android 11 用的（特权版）OpenEUICC" 这个位置目前是空的。** 本仓库补的就是这个缺口。

> **Android 12+（API ≥31）请直接用上游官方构建**，你不需要这里的东西。

### 补丁内容

两处 `smali` 层 SDK 守卫，由 `apktool` 反编译上游 APK 后手工修改：

| 补丁 | 位置 | 改动 |
|:--|:--|:--|
| **A** | `androidx/core/view/WindowInsetsCompat$Impl20.createDisplayShape` | `SDK_INT < 31` 时返回 `null` |
| **B** | `androidx/core/view/DisplayCompat.getRoundedCorner` | `SDK_INT < 31` 时返回 `null` |

两处返回值本来就是调用方**必须处理 null** 的分支——所以我们只是提前走到那个分支，
没有改变任何控制流语义。完整 diff 见 [`patches/`](patches/)。

### 上游版本

- 仓库：`estkme-group/openeuicc`
- 提交：`055d1e2`（2026-10-26，"feat: Profile download API via EuiccService (#366)"）
- 版本号：`v1.7.2-6-g055d1e2`
- 许可证：GPL-3.0（本构建继承同一许可）

---

## 安装（需要 root）

### 前置条件

1. **已 root 的设备**（Magisk / KernelSU）——本方式安装的是系统 priv-app，需要写 `/system`。
2. **Android 11（API 30）**。其他版本请用上游官方构建或 EasyEUICC。
3. 已卸载设备上任何旧版 OpenEUICC（见「故障排查」）。

### 步骤

1. 下载本仓库 Releases 里的 `openeuicc-android11-magisk.zip`；
2. Magisk → 模块 → 从本地安装 → 选择该 zip → 重启；
3. 重启后桌面上应出现 OpenEUICC 图标；
4. 首次打开会请求若干特权，全部允许。

### 校验

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

`customize.sh` 里的 `pm install -r` 会在 `/data/app` 留一份副本，
导致系统看到两个 codePath、判定为受限应用。这是上游模块的既有设计，非本补丁引入。

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
| 需 root | 特权版必须装进 `/system`；免 root 请走上面的 EasyEUICC / 读卡器路径 |
| 需独立签名 | 与上游不兼容，无法互相覆盖安装 |
| ARA-M 限制 | 5ber 卡疑似不放行非特权 app（未实跑 EasyEUICC 验证，见方案 A/D） |
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
- root 会清除设备数据、可能影响保修，**请先备份**；
- 本项目按 GPL-3.0 「AS IS」提供，不附带任何担保。

---

## 许可证

本构建基于 GPL-3.0 授权的上游代码衍生物，整体以 **GPL-3.0** 发布，
详见 [LICENSE](LICENSE)。

上游版权归 estkme-group / PeterCxy 及其贡献者所有。
本仓库的补丁与文档部分版权归 yanyue-dev 所有，同样以 GPL-3.0 授权。

---

## 相关

- 上游项目：<https://github.com/estkme-group/openeuicc>
- 上游原始仓库（含 EasyEUICC 官方发布）：<https://gitea.angry.im/PeterCxy/OpenEUICC>
- lpac（跨平台 LPA）：<https://github.com/estkme-group/lpac>
- EasyLPAC（lpac 图形界面）：<https://github.com/creamlike1024/EasyLPAC>
- 背景：[`docs/background.md`](docs/background.md) —— 这个构建是怎么来的

---

<p align="center"><sub>Made for Android 11 users who were left behind by <code>minSdk = 31</code>.</sub></p>
