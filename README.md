# 兆能 ZN-M2 自定义 OpenWrt 固件云编译

> 基于 [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt) 源码，通过 GitHub Actions 在线编译适用于**兆能 ZN-M2** 路由器的 ImmortalWrt 固件。
>
> 固件内置 **OpenClash + iStore 应用商店 + 网络向导 + WiFi**，精简至可放入 94MB NAND 可用空间。

---

## 固件特性

| 功能 | 说明 |
|------|------|
| OpenClash | 科学上网核心，支持多种协议 |
| iStore 应用商店 | 在线安装/管理 LuCI 插件 |
| iStoreX (网络向导) | 图形化网络配置向导，新手友好 |
| WiFi (ath11k) | 2.4G / 5G 双频无线，板级校准文件已内置 |
| LuCI 面板 | Aurora 主题，中文界面 |
| 系统工具 | htop, curl, wget, blkid, fdisk 等 |

**已砍掉的组件**（因 NAND 容量限制）：

| 组件 | 原因 |
|------|------|
| Docker 全家桶 | 约 60MB，放入 NAND 后固件超容 |
| gecoosac / libimobiledevice / upnp / wolplus / coremark | 非核心大包，合计约 19MB |
| homeproxy / passwall / passwall2 / momo / nikki / tailscale | 避免与 OpenClash 冲突，减少体积 |

> Docker 可在系统启动后通过 U 盘外接存储，使用 `apk` 安装 `dockerd` + `luci-app-dockerman`。

---

## 设备信息

| 项目 | 详情 |
|------|------|
| 设备型号 | 兆能讯通 ZN-M2 |
| SoC | Qualcomm IPQ6018 / IPQ6000 (ARM Cortex-A53) |
| 内存 | 512MB（原装）/ 1GB（改内存后） |
| 闪存 | 128MB NAND |
| NAND 可用容量 | **约 94MB**（772 PEB - 20 PEB 坏块保留 = 752 PEB × 128KB） |
| U-Boot | 暗云闭源 u-boot（支持网页刷机救砖） |
| 编译目标 | `qualcommax / ipq60xx / zn_m2` |
| 源码 | VIKINGYFY/immortalwrt (main 分支，内核 6.18.x) |

---

## 快速开始

### 第一步：Fork 本项目

1. 点击右上角 **Fork** 按钮，将本项目 Fork 到你自己的 GitHub 账号下。

### 第二步：运行编译

1. 进入你 Fork 后的仓库页面。
2. 点击 **Actions** 选项卡。
3. 在左侧选择 **zn-m2-wifi-oc** 工作流。
4. 点击 **Run workflow** 按钮。
5. 根据需要填写参数：

   | 参数 | 说明 |
   |------|------|
   | `PACKAGE` | 手动调整插件包，多个用换行隔开（可留空） |
   | `TEST` | 勾选后仅输出配置文件，不编译固件（建议首次勾选验证） |

6. 点击绿色 **Run workflow** 按钮开始编译。

> 编译时间约 1~2 小时，取决于 GitHub Actions 服务器负载。

### 第三步：下载固件

1. 编译完成后，进入对应的 workflow run 页面。
2. 在页面底部的 **Releases** 或 **Artifacts** 区域下载固件文件。

下载的文件包括：

| 文件 | 用途 | 刷入方式 |
|------|------|----------|
| `*-factory-*.ubi` | 全量刷机镜像 | 暗云 u-boot 网页刷机 |
| `*-sysupgrade-*.bin` | 系统内升级镜像 | LuCI 系统升级 / SSH sysupgrade |

### 第四步：刷入固件

#### 方式一：u-boot 网页刷机（推荐，适用于首次刷机或救砖）

1. 断电，用网线连接电脑到路由器 **LAN 口**。
2. 按住路由器 **RESET 按键** 不放，接通电源。
3. 保持按住约 5~10 秒，直到路由器指示灯出现异常闪烁后松开。
4. 电脑浏览器访问 `192.168.1.1`（暗云 u-boot 默认地址）。
5. 在网页中选择 **固件更新** → 上传 `factory.ubi` 文件 → 点击刷入。
6. 等待刷写完成并自动重启（约 2~3 分钟）。

#### 方式二：系统内升级（适用于已运行 ImmortalWrt/OpenWrt 的路由器）

1. 登录 LuCI 管理界面 `http://192.168.10.1`。
2. 进入 **系统** → **备份/刷机**。
3. 上传 `sysupgrade.bin` 文件。
4. 勾选 **保留配置**（可选），点击 **执行刷机**。
5. 或通过 SSH 执行：`sysupgrade /tmp/sysupgrade.bin`

---

## 刷机后登录

| 项目 | 值 |
|------|------|
| 管理地址 | `http://192.168.10.1` |
| 用户名 | `root` |
| 密码 | 空（未设置） |
| WiFi 名称 | `OWRT` |
| WiFi 密码 | `12345678` |

> 首次登录后请及时修改密码和 WiFi 密码！

---

## 文件结构

```
.
├── .github/workflows/
│   ├── zn-m2.yml                # 主编译工作流（手动触发）
│   └── WRT-CORE2.yml            # 核心编译流程（被 zn-m2.yml 调用）
├── Config/
│   └── IPQ60XX-ZN-M2-OC.txt     # 设备配置文件（定义编译目标和软件包）
├── Scripts/
│   ├── Packages.sh              # 插件包管理（含 iStore feed 源添加）
│   └── Settings.sh              # 系统设置（主题、IP、WiFi 名称密码）
├── README.md                    # 本文件
└── Packages-iStore-addition.sh  # iStore feed 源添加脚本（参考）
```

---

## 自定义配置

### 修改编译目标

编辑 `Config/IPQ60XX-ZN-M2-OC.txt`，核心配置项如下：

```bash
# 设备平台
CONFIG_TARGET_qualcommax=y
CONFIG_TARGET_qualcommax_ipq60xx=y
CONFIG_TARGET_DEVICE_qualcommax_ipq60xx_DEVICE_zn_m2=y

# === 保留的功能 ===
CONFIG_PACKAGE_luci-app-openclash=y        # OpenClash
CONFIG_PACKAGE_luci-app-istorex=y          # iStoreX 网络向导
CONFIG_PACKAGE_luci-app-istore=y           # iStore 应用商店
CONFIG_PACKAGE_luci-app-quickstart=y       # 快速启动向导

# === 砍掉的功能（NAND 放不下）===
CONFIG_PACKAGE_dockerd=n                   # Docker 守护进程
CONFIG_PACKAGE_docker=n
CONFIG_PACKAGE_containerd=n
CONFIG_PACKAGE_docker-compose=n
CONFIG_PACKAGE_luci-app-dockerman=n
```

### 修改主题 / IP / WiFi

编辑 `.github/workflows/zn-m2.yml` 中的参数：

```yaml
WRT_THEME: aurora              # 主题
WRT_NAME: OWRT                 # 主机名
WRT_SSID: OWRT                 # WiFi 名称
WRT_WORD: 12345678             # WiFi 密码
WRT_IP: 192.168.10.1           # 管理地址
```

### 添加自定义插件包

在 **Actions** → **Run workflow** 时，在 `PACKAGE` 参数中填入需要额外安装的包名，多个用换行隔开。

---

## NAND 容量说明（重要）

ZN-M2 路由器使用 128MB NAND 闪存，但实际可用于固件的空间有限：

```
NAND 物理总量:     128 MB
MTD16 分区总量:    772 PEB × 128KB = 96.5 MB
坏块保留区:        20 PEB × 128KB = 2.5 MB
────────────────────────────────────
实际可用空间:      752 PEB × 128KB = 94.0 MB
```

**如果固件超过 94MB，将无法刷入！** u-boot 和 sysupgrade 都会拒绝写入。

本固件砍掉 Docker 等大包后，体积约 **36~45MB**，可轻松放入可用空间。

> 如需使用 Docker，建议外接 U 盘/移动硬盘，将 Docker 数据目录指向外部存储后安装。

---

## 常见问题

### Q: 刷机后找不到管理页面？

检查以下几点：
- 电脑是否通过网线连接路由器 **LAN 口**
- 电脑 IP 是否设为自动获取 (DHCP)
- 浏览器访问的是 `192.168.10.1`（不是 192.168.1.1）
- 路由器指示灯是否正常（非异常闪烁）

### Q: WiFi 搜不到信号？

- 本固件已内置 ath11k WiFi 驱动和板级校准文件
- 若仍搜不到，可能是个别批次硬件差异，尝试在 LuCI → 网络 → 无线 中手动启用 radio
- 确认刷入的是 `factory.ubi`（完整镜像），而非仅 `sysupgrade.bin`

### Q: 固件刷不进去 / u-boot 报错？

- 确认下载的文件是 `factory.ubi`（u-boot 刷机用），不是 `sysupgrade.bin`
- 确认固件大小未超过 94MB（本仓库编译的固件通常 36~45MB）
- 如果之前刷过其他固件导致分区异常，可用暗云 u-boot 重新刷入 `factory.ubi` 恢复

### Q: 如何恢复到官方固件或其他固件？

- 暗云 u-boot 支持网页刷机，随时可以刷入其他 `factory.ubi` 镜像
- 刷机步骤与本 README 中「u-boot 网页刷机」一致

### Q: TEST 模式是做什么的？

- 勾选 `TEST=true` 后，工作流只生成 `.config` 配置文件并上传到 Release，不编译固件
- 建议修改配置后首次运行时勾选，确认配置无误再正式编译
- 检查 Release 中的配置文件是否包含你需要的包

### Q: 编译失败怎么办？

- 查看 Actions 运行日志，定位失败步骤
- 常见原因：网络问题导致源码克隆失败、配置文件中有不存在的包名
- 可重新运行 workflow（GitHub Actions 偶发失败重试即可）

---

## 致谢

- [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt) — ImmortalWrt 源码
- [xdm2211/ZN-M2-VIKINGYFY](https://github.com/xdm2211/ZN-M2-VIKINGYFY) — 原始云编译项目
- [vernesong/OpenClash](https://github.com/vernesong/OpenClash) — OpenClash 插件
- [linkease/istore](https://github.com/linkease/istore) — iStore 应用商店
- [linkease/nas-packages-luci](https://github.com/linkease/nas-packages-luci) — 网络向导 (quickstart)
- [linkease/nas-packages](https://github.com/linkease/nas-packages) — quickstart 后端

---

## License

MIT
