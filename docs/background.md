# 背景：这个构建是怎么来的

## 一句话

手上有一张 **5ber eSIM 卡**，它的管理应用在厂商服务器下线后无法登录，
卡里的 profile 管理不了。卡本身是好的——于是换一个开源 LPA 来管它，
结果发现官方 OpenEUICC 在 Android 11 上装不上。

## 5ber 发生了什么

5ber 是一个把 eSIM 写进**可移动物理卡**里的产品——这样没有 eSIM 支持的老手机也能用 eSIM。
它的管理应用（`com.ifreegroup.moesim`）负责把运营商的 profile 下载、写入、启停卡上的 eUICC 芯片。

2026 年，其母公司 iFree 资金链断裂，`api.5ber.com` 关闭。
结果是一个很尴尬的局面：

- **卡是实体的，还在手上，芯片还能工作；**
- 用户已经写进卡里的 profile **也还在卡上**；
- 但**管理它的应用登不进去了**——因为登录走服务器。

于是「我的 eSIM 还能用，但我改不了它」成了所有 5ber 用户的共同处境。

## 为什么是「换 LPA」而不是「改 app」

最初的思路是改 5ber 应用、绕过登录。逆向之后发现这条路又难又不必要：

- **5ber app 的写卡能力全在本地**——原生插件走 OMAPI 直连卡的 eUICC 芯片，
  `lpa#getProfiles` / `enableProfile` / `deleteProfile` 等操作都不经过服务器；
- 服务器只负责三件事：**登录鉴权、写卡配额计数、订单**；
- 登录逻辑在 Flutter AOT 编译产物 `libapp.so` 里，没有可靠的 patch 手段。

⇒ **换一个开源的 LPA 客户端就行**，卡还是那张卡。

市面上能满足这个的，就是 OpenEUICC。

## 为什么这条路上又出现一道坎

OpenEUICC 的 `minSdk = 31`。手上这台设备是 **Android 11（API 30）**，差一个版本号，
安装器直接拒绝：`INSTALL_FAILED_OLDER_SDK`。

强行装进去之后又是第二个坑：启动即崩，

```
java.lang.NoSuchMethodError: No virtual method getRoundedCorner(I)
    in class Landroid/view/Display;
```

顺着堆栈往上找：

```
WindowInsetsCompat$Impl20.createDisplayShape
  → DisplayCompat.getRoundedCorner
    → Display.getRoundedCorner(I)      ← API 31 才有的方法
```

这不是配置缺失、不是资源问题——**代码真的调用了一个 API 31 才存在的方法**。
上游把 `minSdk` 设成 31 是有道理的：它确实依赖 31 的 API。

但调用方（`createDisplayShape`）本来就**必须处理 `null` 返回**——因为在支持的平台上，
`getRoundedCorner` 完全可能返回 `null`（比如设备没有圆角信息）。
于是解法很轻：**在 API < 31 时直接返回 `null`，让调用方走它本来就要走的 null 分支。**

两处补丁，各五六行 smali，返回 `null`，不碰任何业务逻辑。

## 结果

打完之后，OpenEUICC 在 Android 11 上正常启动，把卡里已有的两张 profile
（一张 CMLink GDS、一张 Lifecell）都列了出来，启停、切换都正常，
**并且实机切了一次卡，没有触发 5ber 卡已知的「切卡后 modem 不刷新」问题。**

## 为什么公开出来

因为这道坎不是一个人的坎。**所有 Android 11 的 5ber 用户都会撞上同一个 `minSdk = 31`**，
而他们的问题完全一样：卡是好的，app 没了。

上游不会为 API 30 开倒车——那是对的，维护一个已过时的 SDK 版本长期看是负担。
但这不代表用 Android 11 的人就该被落下。

所以这里放一份打好的包，以及完整的补丁文本。
如果上游将来某天把 `minSdk` 降回 30，或者官方推出兼容构建，**这个仓库就可以退休了**。

## 许可与归属

上游 OpenEUICC 以 **GPL-3.0** 授权。作为衍生分发，本构建必须、也确实是 GPL-3.0。

本仓库**不是**上游的分支，不向上游提 PR，也不代表上游立场。
它是补丁 + 打包，仅此而已。

---

*写于 2026-10-27，一台 Android 11 设备上的 5ber 卡被救活之后。*
