# Our Free Model 安装教程

把 [dsh-our-free-model](https://github.com/Ebony-Vinyl/dsh-our-free-model)（`dsh-our-free-model`）装进 **DeepSeek Harness 桌面端**，装完就能在模型选择器里直接选到它的模型。

这个插件给 DSH 加一条**免密免费车道**：不用登录、不用注册、不用填 API Key，装上就有模型可用。桌面端另有一条需要授权的 EAC 协付渠道。

> 本文在 macOS 桌面端 `0.2.0-rc.2` + 插件 `2.1.0` 上实机验证过（2026-10-11）。
> 只想最省事，直接跳到 [快速开始](#快速开始)。

---

## 目录

- [装之前先想清楚](#装之前先想清楚)
- [开始之前](#开始之前)
- [快速开始](#快速开始)
- [方式 A：桌面端点选安装](#方式-a桌面端点选安装)
- [方式 B：命令行安装](#方式-b命令行安装)
- [装完怎么确认](#装完怎么确认)
- [EAC 渠道：桌面端专属，需额外授权](#eac-渠道桌面端专属需额外授权)
- [常见问题](#常见问题)
- [卸载](#卸载)
- [数据在哪](#数据在哪)

---

## 装之前先想清楚

这个插件和 DS Bot 不一样：它是**模型来源**，不是功能插件。三件事值得先知道。

1. **免费不等于私有。** 免费车道的上游之一是 Kilo AI 的公共网关，它的免费池模型卡原文写着：prompt 可能被上游提供方记录并用于改进其服务。**不要通过免费车道发送密码、密钥、个人身份信息或商业机密。**
2. **免费不等于稳定。** 模型集合与额度由上游单方面决定，随时可能限流、下线或转为收费；按地区返回 `403`、超量返回 `429` 都属正常。
3. **它是第三方插件，代码跑在宿主进程里。** 且默认开启应用内自升级（升级清单需通过插件内 pin 的 Ed25519 公钥验签）。介意的话，装完可以在设置页关掉更新检查。

---

## 开始之前

两件事：

1. **装好 DSH 桌面端，并且里面至少有一个能用的模型。** 这个插件是"再加几条来源"，不是替代品。
2. **确认本机能访问 GitHub。** 这比 DS Bot 那条更关键：**这个插件主要从 GitHub 仓库安装**，npm 镜像帮不了 GitHub。

```sh
git ls-remote https://github.com/Ebony-Vinyl/dsh-our-free-model.git HEAD
```

能打印出一行 `<sha>\tHEAD` 就通过了。

两个常见坑：

- **本机 git 配了 `insteadOf` 重写规则**（把 https 改写成 ssh），会让下载在开始前就失败。检查 `git config --global --get-regexp insteadof`，有输出就按需清理。
- **npm 源通不通，跟这里没关系**——但反过来，GitHub 通不通，npm 镜像也救不了。

> 顺带一提：这个包在 npm 上也有（包名 `dsh-our-free-model`），但截至撰写时 **npm 上是 `2.0.0`，仓库已经是 `2.1.0`**。想要新版就走 GitHub 地址，别用包名。

---

## 快速开始

**桌面端（推荐）**

1. 侧边栏 → **插件** → **添加插件**
2. **包名或地址** 填 `https://github.com/Ebony-Vinyl/dsh-our-free-model`
3. 点 **安装**，装完点 **立即启用**
4. 按 `⌘R` 刷新页面，模型选择器里会出现 **Our Free Model** 分组

**命令行**

```sh
dsh plugin --profile desktop add github:Ebony-Vinyl/dsh-our-free-model
```

`dsh` 命令本身怎么装，见 [方式 B](#方式-b命令行安装) 第 1 步。

---

## 方式 A：桌面端点选安装

不需要终端，全部在界面里完成。

### 1. 打开插件页

侧边栏点击 **插件**（英文界面是 **Plugins**）。

### 2. 点「添加插件」

在弹出的对话框里填一处即可：

| 字段 | 填什么 |
|---|---|
| **包名或地址** | `https://github.com/Ebony-Vinyl/dsh-our-free-model` |
| **安装源** | 不用改 |

关于**安装源**：它只决定 npm 包名去哪个源查。你填的是 GitHub 地址，文件从 GitHub 拉，所以这一项填什么都不影响。**不要因为国内网络就把地址改回 npm 包名**——那样会装到旧版 `2.0.0`。

关于**地址写法**：`github:Ebony-Vinyl/dsh-our-free-model` 和 `https://github.com/Ebony-Vinyl/dsh-our-free-model` 都行；界面里推荐后者，更直观。

### 3. 安装 → 立即启用

点 **安装**。DSH 会先做一次 GitHub 可达性检查（超时 5 秒），通过后才交给 pnpm 拉取。安装过程可以展开 **查看安装详情** 看 pnpm 的实时输出。想固定版本，在地址后加 `#<commit>`。

装完按钮变成 **立即启用**，点它，插件就被写进当前 profile 的 bundle 列表。

> 手滑关掉了对话框也不要紧：包已经装上了，只是没启用。回到插件列表，在 **已安装** 分组里找到它启用即可。

### 4. 刷新或重启

实测在 `0.2.0-rc.2` 上，**装完 host 半边会自己加载，不需要重启**：一分钟内 `~/.dsh/our-free-model/` 就会出现，模型清单与可用性探测都已在跑。

但**客户端那半边界面**要等页面重新加载，所以按一次 `⌘R`。如果模型选择器里仍然没有分组，再**完全退出 DSH 重开**（不是关窗口，是退出应用）。

---

## 方式 B：命令行安装

适合批量部署、脚本化，或者你想确认每一步到底改了什么。

### 1. 让 `dsh` 命令可用

```sh
dsh --version
```

提示 `command not found` 的话，**不要用 npm 全局安装**：

**菜单栏 →「DeepSeek Harness」菜单 → 管理 dsh 命令…** → 点 **安装**，然后**新开一个终端窗口**再验证。

这一点在这里比在 DS Bot 那里更要紧。桌面端的 profile 由应用独占，只有桌面端自己装的那个 `dsh` 能碰它：

- 桌面端装的 `dsh` → 最终执行 `app.asar/dsh/node_modules/@deepseek-ai/dsh-desktop-host/lib/cli.js`，带 `manageDesktopProfile` 特权，`plugin --profile desktop ...` 正常工作；
- npm 或其它来源的 `dsh` → 被拒绝：`profile "desktop" is managed exclusively by the Electron application`。

注意这个限制只针对 `desktop` 这个保留 profile；`dsh --profile desktop` 直接启动 profile 同样会被拒，那是设计如此。

### 2. 安装插件

```sh
dsh plugin --profile desktop add github:Ebony-Vinyl/dsh-our-free-model
```

几点说明：

- `--profile desktop` 必填。
- 想锁定版本：`github:Ebony-Vinyl/dsh-our-free-model#b8369d5`。
- 跑之前**不必**退出 DSH。这条命令和 DSH 抢的是同一把 profile 写锁，会串行等待，不会写坏；想最干净就先退出。
- 安装日志在 `~/.dsh/profiles/desktop/.plugin-manager/logs/`。

实测输出（插件 2.1.0）：

```
dependencies:
+ dsh-our-free-model github:Ebony-Vinyl/dsh-our-free-model#b8369d53de6eab91593104001d941086ee714c45

Done in 29.8s using pnpm v11.7.0
```

它做两件事：把 `dsh-our-free-model` 写进 profile 的 `dependencies`，并把它**追加到 `dsh.profile.bundles` 列表末尾**（也就是顺便帮你启用了）。

### 3. 刷新

同方式 A：先 `⌘R`，必要时完全重启。

---

## 装完怎么确认

### 界面确认

模型选择器里出现 **Our Free Model** 分组，展开能看到模型。分组里的名字是插件自己起的（例如 Space Bunny、LongCat 2.5 Preview、MiMo V2.6 Flash 等），不一定和上游厂牌同名。

### 命令行确认

核对落盘结果：

```sh
cd ~/.dsh/profiles/desktop
python3 -c "
import json
d = json.load(open('package.json'))
print('依赖:', d['dependencies'].get('dsh-our-free-model'))
print('bundles:', d['dsh']['profile']['bundles'][-2:])
"
ls node_modules/dsh-our-free-model
```

期望看到：依赖里是 `github:...#<sha>`，`bundles` 末尾有 `dsh-our-free-model`，目录存在且包含 `index.js`、`client.js`、`adapter/`、`src/`。

再确认插件真的在跑：

```sh
ls ~/.dsh/our-free-model/
```

看到 `catalog.json`、`availability.json`、`feed.json`、`settings.json`，就说明 host 半边已经起来了。

实测在 `0.2.0-rc.2` 上，免费车道当时拉到 12 个模型，探测结果 **7 个 available、2 个 region-blocked、2 个 unavailable、1 个已下线**。你看到的数字会随上游和你的出口地区变化。

---

## EAC 渠道：桌面端专属，需额外授权

插件仓库标题里的 **DeepSeek V4.1 Flash、Kimi K3** 走的是 EAC 协付渠道，它**只存在于桌面宿主**：命令行、纯 `dsh web` 里根本没有这条通道（没有模型、没有请求、也不会尝试解密）。

宿主这一侧的解锁条件在 DSH 桌面端是自动满足的（桌面端用的就是保留的 `desktop` profile）。但它另有一道**按人计的 GitHub 授权闸门**：

1. 打开 **设置 → Our Free Model → EAC 渠道授权**
2. 在浏览器里用 GitHub 登录，并给插件仓库点一个 star
3. 网关确认 star 状态后签发专属令牌，插件自动领取，不用复制粘贴

**没授权时你只有免费车道**，看不到上面那两个模型。授权用的是你自己的 GitHub 账号，插件只申请 `read:user` 最小权限，且网关默认每 12 小时复查一次 star 状态。

---

## 常见问题

### 装的时候报「无法访问 GitHub」/「GitHub 连接超时」

你填的是 GitHub 地址，走的就是 GitHub。先按 [开始之前](#开始之前) 那两条检查：本机能否访问 GitHub、有没有 `insteadOf` 重写规则。**换 npm 镜像解决不了这个问题。**

### 提示「找不到该插件」

说明你填的是包名而不是 GitHub 地址，而当前选的源里查不到它。改填 `https://github.com/Ebony-Vinyl/dsh-our-free-model`。

### 装完模型选择器里没有 Our Free Model

先按 `⌘R` 刷新；再确认是真的**完全退出**应用后重开。还不行，就按上面的"命令行确认"核对 `bundles` 里有没有这一项。

### 有模型，但显示不可用 / `region-blocked`

正常。免费车道按出口地区判定，部分模型会返回"你所在的国家不可用"；也可能有模型已被上游下线。插件会把结果如实标出来，不需要你做任何处理。

### 报 429 / 提示额度用完

上游限流。免费额度由上游单方面提供，等一会儿或换个模型即可。

### 只看到免费车道的模型，没有 DeepSeek V4.1 Flash / Kimi K3

EAC 渠道还没授权，见上一节。

### 用包名装，装到了 2.0.0

npm 上的版本落后于仓库。卸载后用 GitHub 地址重装。

### 报 `ERR_PNPM_VIRTUAL_STORE_DIR_MAX_LENGTH_DIFF`

这是 profile 里 pnpm 状态的问题，跟插件无关（报错发生在下载插件之前）：profile 中已有的 `node_modules` 由旧版 pnpm 生成，而 DSH 更新后内置的 pnpm 版本变了。关闭 DSH，删掉该 profile 的 `node_modules` 与 `pnpm-lock.yaml` 让它重建，再装一次。

### 提示「以下包声明了安装脚本，pnpm 默认不运行」

本插件不需要构建脚本，正常安装不会遇到这一步。真遇到了，先确认你装的是这个插件，再决定是否放行。

### 提示版本不兼容（`incompatible-version`）

插件对 `@deepseek-ai/*` 的 peer 版本有要求。**首选做法是升级 DSH 桌面端**，不要硬装。实在要装：

```sh
dsh plugin --profile desktop allow-version dsh-our-free-model@2.1.0 --dsh-version <你的DSH版本> --accept-risk
```

⚠️ 自担风险：不兼容的插件可能导致崩溃或数据损坏，且豁免只对"这一个包 + 这一个 DSH 版本"生效，升级 DSH 后不继承。

### 报 `profile "desktop" is managed exclusively by the Electron application`

你的 `dsh` 不是桌面端装的那个。见 [方式 B](#方式-b命令行安装) 第 1 步，用菜单里的"管理 dsh 命令…"重装。

### 插件会自己更新吗

会。它默认开启应用内自升级（`distribution: self`），升级清单要通过插件内 pin 的公钥验签才会被安装。它同时会按间隔轮询仓库的公告 feed。不想让它自动查，可以在设置页关掉。

---

## 卸载

**桌面端**：侧边栏 → **插件** → 在 **已安装** 里找到 `dsh-our-free-model` → 卸载。

**命令行**：

```sh
dsh plugin --profile desktop remove dsh-our-free-model
```

然后重启 DSH。

卸载**不会**删除 `~/.dsh/our-free-model/` 里的数据；想彻底清干净，卸载后手工删掉那个目录。

---

## 数据在哪

| 内容 | 路径 |
|---|---|
| 插件设置 | `~/.dsh/our-free-model/settings.json` |
| 模型清单 | `~/.dsh/our-free-model/catalog.json` |
| 可用性探测结果 | `~/.dsh/our-free-model/availability.json` |
| 公告 feed 缓存 | `~/.dsh/our-free-model/feed.json` |
| 安装位置（profile） | `~/.dsh/profiles/desktop/` |
| 安装日志 | `~/.dsh/profiles/desktop/.plugin-manager/logs/` |

按插件文档：所有状态只写在本机，用量与设置不上传。推理请求会按你选的模型发往对应上游——这一点和调用任何模型 API 一样。

---

## 参考

- 插件仓库：<https://github.com/Ebony-Vinyl/dsh-our-free-model>
- 插件 README（安装、上游来源、安全边界）：<https://github.com/Ebony-Vinyl/dsh-our-free-model#readme>
- DeepSeek Harness：<https://github.com/deepseek-ai/deepseek-harness>

## 许可

本教程为社区文档，Apache-2.0。`dsh-our-free-model` 是独立社区项目，**不是 DeepSeek 官方产品**，与 DeepSeek 无隶属或背书关系。免费额度由各上游提供方单方面提供，使用前请自行确认其条款。
