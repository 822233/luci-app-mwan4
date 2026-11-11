# luci-app-mwan4

[English documentation](README_EN.md)

这是一个用于配置和管理 **MWAN4（多 WAN 负载均衡/故障切换）** 的 LuCI Web 管理界面。它是前端管理插件，不包含 MWAN4 后端本身。路由器需要先安装并正确配置兼容版本的 `mwan4` 后端。

本仓库额外提供 GitHub Actions 云端编译流程，可为 **ImmortalWrt 24.10、MediaTek Filogic（aarch64）** 目标编译 `luci-app-mwan4` 的 IPK 安装包。无需在路由器上安装编译工具链，也无需自行搭建 OpenWrt 编译环境。

## 主要功能

- **运行总览**：查看 MWAN4 服务状态、WAN 链路数量、路由规则和连接统计，并提供常用服务操作入口。
- **WAN 链路管理**：添加、编辑、启用或停用 WAN 接口，查看 IPv4/IPv6 状态、健康检查结果及链路历史。
- **健康检查**：提供敏感、均衡、宽容等预设，可配置检查间隔、失败/恢复阈值、检测目标和保持时间。
- **路由规则**：按域名、目标地址、设备、协议和端口等条件配置流量走向，支持规则模板、排序、匹配状态和连接统计。
- **路由策略**：配置默认出口、主备切换或负载分担策略。
- **流量测试**：输入目标地址及可选设备、协议和端口，检查流量会匹配哪条规则以及当前使用的链路。
- **PBR 导入**：在后端支持且权限允许时，将 PBR 策略预览并导入 MWAN4。
- **诊断工具**：查看接口、路由、规则、nftables、防火墙服务及运行状态等诊断信息。
- **高级配置**：提供 WAN 链路、优先级、路径、路由规则、全局设置、MPTCP 等高级配置页面。
- **迁移辅助**：协助迁移仍使用旧版 `member`、`policy` 配置段名称的 MWAN4 配置。

部分高级界面依赖后端提供相应 RPC 方法；如果当前后端版本不支持，对应功能会自动隐藏或显示为不可用。

## 云端编译 IPK（推荐）

本仓库已配置 GitHub Actions 工作流：

`.github/workflows/build-ipk.yml`

工作流使用 GitHub 提供的 Ubuntu 云端运行环境，并下载 ImmortalWrt 24.10 MediaTek Filogic SDK 编译本插件。

### 自动编译

向仓库推送代码变更后，满足工作流路径触发条件时会自动开始编译。

### 手动编译

1. 打开本仓库 GitHub 页面。
2. 进入顶部的 **Actions**。
3. 在左侧选择 **Build luci-app-mwan4 IPK**。
4. 点击 **Run workflow**，再确认运行。
5. 等待任务完成；绿色勾表示工作流成功，红色叉表示编译失败。

### 下载 IPK

1. 打开对应的 Actions 运行记录。
2. 在运行页面底部找到 **Artifacts**。
3. 下载 `luci-app-mwan4-immortalwrt-24.10-filogic`。
4. 解压下载的 ZIP 文件，取得其中的 `.ipk` 安装包。

Artifact 默认保留 14 天。构建成功后请及时下载保存。

## 路由器安装

将 IPK 上传到路由器（例如放在 `/tmp`），通过 SSH 执行：

```sh
opkg install /tmp/luci-app-mwan4_*.ipk
```

也可以在 LuCI 的软件包页面上传并安装 IPK。

安装后刷新 LuCI 页面，通常可在 **状态（Status）→ MultiWAN 4** 查看状态，并在服务或网络相关菜单中进入配置页面；具体菜单位置可能因 LuCI 版本和固件菜单布局而不同。

## 简体中文界面

仓库提供独立的简体中文语言包，包名为 `luci-i18n-mwan4-zh-cn`。安装主程序后，另外安装该语言包即可在 LuCI 语言设置为简体中文时显示对应翻译；英文界面仍可保留。

GitHub Actions 构建成功后，Artifact 中会同时包含：

- `luci-app-mwan4_*.ipk`：MWAN4 LuCI 主界面
- `luci-i18n-mwan4-zh-cn_*.ipk`：简体中文翻译

在路由器上安装中文语言包：

```sh
opkg install /tmp/luci-i18n-mwan4-zh-cn_*.ipk
```

然后在 LuCI 的系统语言设置中选择“简体中文”，刷新页面。未完成翻译的文本会继续显示英文。

## 安装前注意事项

1. **必须有 MWAN4 后端**：本插件依赖 `mwan4` 和 `luci-base`，它本身不是 MWAN4 服务端。请先安装并配置后端。
2. **固件与 SDK 尽量匹配**：当前工作流面向 ImmortalWrt 24.10 的 MediaTek Filogic 目标。其他固件、分支、目标平台或软件包格式不保证兼容。
3. **内核模块不是本插件的构建内容**：LuCI 插件 IPK 不会替你解决后端所需的内核模块问题。内核模块必须与路由器当前固件的内核版本及 ABI 匹配。
4. **先备份配置**：安装或升级前，建议备份 `/etc/config/mwan4`。插件包含旧配置格式迁移逻辑，迁移前会尝试创建备份文件，但仍建议自行备份。
5. **不要仅凭能安装判断功能正常**：安装成功后，还需要确认 `mwan4` 服务正常运行、WAN 接口存在且路由策略配置正确。

## 依赖

Makefile 声明的主要运行依赖：

- `luci-base`
- `mwan4`

连接统计、链路历史、PBR 导入等部分功能还需要后端提供对应的 rpcd 方法或状态字段。

## 项目测试

在具备 Node.js 的开发环境中，可运行：

```sh
for t in tests/*.test.js; do node "$t"; done
```

项目还包含 Playwright 端到端测试。模拟环境测试位于 `tests/e2e/*.mock.spec.js`；针对真实路由器的测试需要单独配置，不属于普通模拟测试。

## 项目说明

- 上游项目：<https://github.com/brauliobo/luci-app-mwan4>
- MWAN4 后端：<https://github.com/brauliobo/mwan4>
- 本仓库：<https://github.com/822233/luci-app-mwan4>
- 云端编译记录：<https://github.com/822233/luci-app-mwan4/actions>

欢迎提交 Issue 或 Pull Request。反馈问题时请尽量附上固件名称与版本、设备目标平台、MWAN4 后端版本，以及相关日志。
