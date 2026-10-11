# DS Bot 安装教程

把 [DS Bot](https://github.com/FeiZhuLulu/DeepSeek-Bot)（`ds-bot`）装进 **DeepSeek Harness 桌面端**，从零到能跟第一个 Bot 说话。

DS Bot 是 DSH 的插件，装上之后 DSH 就变成一支 Bot 团队：一个主 Bot（Chief）、若干成员 Bot、群聊、Bot 互相传话、长期记忆。

> 本文在 macOS 桌面端 `0.2.0-rc.2` 上实机验证过，包含国内网络环境下最常踩的坑。
> 如果你只是想要最省事的路径，直接跳到 [快速开始](#快速开始)。

---

## 目录

- [开始之前](#开始之前)
- [快速开始](#快速开始)
- [方式 A：桌面端点选安装](#方式-a桌面端点选安装)
- [方式 B：命令行安装](#方式-b命令行安装)
- [装完怎么确认](#装完怎么确认)
- [常见问题](#常见问题)
- [卸载](#卸载)
- [数据在哪](#数据在哪)

---

## 开始之前

两件事必须先满足，否则后面会卡住：

1. **装好 DSH 桌面端，并且在里面配好至少一个模型。**
   DS Bot 不自带模型，它用的就是 DSH 里的模型。打开 DSH，随便发一句话确认能正常回复。

2. **确认你能访问 npm 源。**
   这是国内最常见的一道坎。本机如果直连 `registry.npmjs.org` 超时（很多网络环境下会），就需要走国内镜像源 `registry.npmmirror.com`。两种安装方式都能指定镜像，下面分别说明。

检查一下你的网络：

```sh
curl -s -o /dev/null -w "npmjs: %{http_code} %{time_total}s\n" https://registry.npmjs.org/ds-bot
curl -s -o /dev/null -w "mirror: %{http_code} %{time_total}s\n" https://registry.npmmirror.com/ds-bot
```

两个都是 `200` 最好；只有 mirror 返回 `200` 也没问题，走方式 A 时把**安装源**选成「中国大陆镜像源」，走方式 B 时加 `--registry` 参数即可。两个都超时说明网络需要代理，先解决网络。

---

## 快速开始

**桌面端（推荐，绝大多数人走这条）**

1. 侧边栏 → **插件** → **添加插件**
2. **包名或地址** 填 `ds-bot`
3. **安装源** 选 **中国大陆镜像源**（国内网络建议直接选它）
4. 点 **安装**，装完点 **立即启用**
5. 如果提示「更改将在下次启动生效」，**完全退出 DSH 再打开**
6. 侧边栏出现 **DS Bot**，主 Bot **Chief** 已经在里面等你

**命令行**

```sh
dsh plugin --profile desktop add ds-bot --registry=https://registry.npmmirror.com/
```

然后重启 DSH。

---

## 方式 A：桌面端点选安装

不需要终端，全部在界面里完成。

### 1. 打开插件页

侧边栏点击 **插件**（英文界面是 **Plugins**）。这个页面用来「安装、启用和配置插件」；只读的内置插件清单在 **设置 → 内置插件** 里，两者不是一回事。

### 2. 点「添加插件」

在弹出的对话框里填两处：

| 字段 | 填什么 |
|---|---|
| **包名或地址** | `ds-bot` |
| **安装源** | **中国大陆镜像源** |

关于**安装源**：DSH 在你第一次打开这个对话框时，会同时 ping 官方源和中国大陆镜像，谁先返回 2xx 就用谁，所以它可能会自动帮你选好。如果你所在网络访问官方源很慢或不通，**手动选「中国大陆镜像源」**。这个选择会被浏览器记住，下次打开沿用。

关于**包名或地址**：这里接受 npm 包名、GitHub 地址、tarball 直链或本地目录。我们填最省事的 npm 包名 `ds-bot`。如果 npm 源整个不通，也可以改用下面两种替代写法：

- 下载 [Releases](https://github.com/FeiZhuLulu/DeepSeek-Bot/releases) 里的 `ds-bot-0.1.0.tgz`，把**它的完整路径**填进去；
- 或者填 `github:FeiZhuLulu/DeepSeek-Bot#v0.1.0&path:/ds-bot`（需要本机能访问 GitHub 且装了 Git）。

### 3. 安装 → 立即启用

点 **安装**。DSH 会先向源查询这个包（预览名称、版本、简介），确认无误后才真正下载。安装过程中可以展开 **查看安装详情** 看 pnpm 的实时输出。

装完后按钮会变成 **立即启用**，点它，插件就被加进当前 profile 的 bundle 列表并关闭对话框。

> 如果这里手滑关掉了对话框：包已经装上了，只是还没启用。回到插件列表，在 **已安装** 分组里找到 `ds-bot` 打开即可。

### 4. 重启 DSH

如果界面提示 **更改将在下次启动生效**，说明这个 profile 没开热加载，需要**完全退出 DSH 再重新打开**（不是关窗口，是退出应用）。

即使没提示重启，也建议重开一次，让客户端那半边界面代码加载进来。

---

## 方式 B：命令行安装

适合批量部署、脚本化，或者你想确认每一步到底改了什么。

### 1. 让 `dsh` 命令可用

桌面端自带 CLI，但**默认不在 PATH 上**。先确认：

```sh
dsh --version
```

如果提示 `command not found`，不用去折腾 npm 全局安装，桌面端自己就能装这个命令：

**菜单栏 →「DeepSeek Harness」菜单 → 管理 dsh 命令…** → 点 **安装**。

它会往你的终端里注册一个指向桌面端内置运行时的 `dsh`。**装完请新开一个终端窗口**，再跑 `dsh --version` 验证。这个对话框里同时会显示当前命令路径、是否被其他安装遮挡（`Another dsh takes precedence`），以及 **修复 / 移除** 按钮。

### 2. 安装插件

```sh
dsh plugin --profile desktop add ds-bot --registry=https://registry.npmmirror.com/
```

几点说明：

- `--profile desktop` 是桌面端使用的 profile 名，**必填**。
- `--registry=...` 指向国内镜像。你的网络能直连官方源的话可以省略。
- 这个命令可以在任何终端里跑，包括在 DSH 会话内部。**但不要试图用 `dsh --profile desktop` 直接启动这个 profile**——它由桌面端独占管理，会被拒绝：`profile "desktop" is managed exclusively by the Electron application`。安装插件用的是 `dsh plugin --profile desktop ...`，这条路径不受限制。
- 跑之前**建议**先完全退出 DSH。DSH 自己会和这条命令抢 profile 的写锁（它会串行等待，不会写坏），但退出后再装最干净，也避免插件在装到一半时被热加载。

命令成功时会看到类似输出：

```
dependencies:
+ ds-bot 0.1.0

Added 1 entry to minimumReleaseAgeExclude in pnpm-workspace.yaml
Done in 833ms using pnpm v11.7.0
```

它会做两件事：把 `ds-bot` 写进 profile 的 `dependencies`，并把 `ds-bot` **追加到 `dsh.profile.bundles` 列表末尾**（也就是顺便帮你启用了，不用再手工改配置）。

### 3. 重启 DSH

完全退出再打开。

---

## 装完怎么确认

### 界面确认

重启后侧边栏应该出现 **DS Bot**，里面已经有一个主 Bot **Chief**（Chief of Staff），用你 DSH 的默认模型。跟它说句话，它就能带你继续配其他 Bot 和群聊。

**如果侧边栏没看到 DS Bot**：先按 `⌘R` 刷新页面；还不行就确认是真的**完全退出**了应用再打开，而不是只关了窗口。

### 命令行确认

如果你走了方式 B，可以顺手核对落盘结果：

```sh
cd ~/.dsh/profiles/desktop
python3 -c "
import json
d = json.load(open('package.json'))
print('依赖:', d['dependencies'].get('ds-bot'))
print('bundles:', d['dsh']['profile']['bundles'][-3:])
"
ls node_modules/ds-bot
```

期望看到 `ds-bot` 的版本号、`bundles` 列表末尾有 `ds-bot`，且 `node_modules/ds-bot` 目录存在。

再确认插件真的在跑：

```sh
ls ~/.dsh/bot
```

看到 `state.json` 就说明 host 半边已经起来了（`state.json` 里记录着 Bot 团队的状态）。用文本编辑器打开它，能看到 `Chief` 这个名字。

---

## 常见问题

### 安装报「无法访问 GitHub」/「GitHub 连接超时」

你填的是 GitHub 地址（`github:...#v0.1.0&path:/ds-bot`）。镜像源只能提供 npm 上的包，**帮不了 GitHub 仓库本身**。点对话框里的 **改用国内镜像**，然后把**包名或地址**改回 npm 包名 `ds-bot` 重新装。

### 报「找不到该插件」/ 所有源都无法访问

说明当前选的源里没有这个包，或者源本身不通。换 **中国大陆镜像源** 重试；如果你公司/学校有私有源，也可以直接填自定义源地址。

### 安装时卡住 / 一直转圈

国内直连官方源常见。点 **取消安装**，把**安装源**改成 **中国大陆镜像源** 再装一次。

如果日志里出现大段 `GET https://registry.npmjs.org/... error (23)`，就是官方源不可达，同一个解法。

### 提示「以下包声明了安装脚本，pnpm 默认不运行」

pnpm 出于安全默认不执行依赖的安装脚本。如果确实需要放行，点 **允许这些脚本并重试**。DS Bot 本身不需要构建脚本，正常安装不会遇到这一步。

### 提示版本不兼容（`incompatible-version`）

DS Bot 对 DSH 版本有要求（它声明了 `@deepseek-ai/cordis`、`dsh-llm`、`dsh-tools`、`dsh-web` 等 peer 依赖）。**首选做法是升级 DSH 桌面端**，而不是硬装。

实在要强行安装，可以用版本豁免：

```sh
dsh plugin --profile desktop allow-version ds-bot@0.1.0 --dsh-version <你的DSH版本> --accept-risk
```

⚠️ 这是**自担风险**的操作：不兼容的插件可能导致崩溃或数据损坏，而且豁免只对「这一个包 + 这一个 DSH 版本」生效，升级 DSH 后不继承。

### 插件列表里 `ds-bot` 显示有问题标签

说明这个依赖装了，但没有被选为 bundle（只装不启用）。在插件页找到它，启用即可；或者手工确认它出现在 `~/.dsh/profiles/desktop/package.json` 的 `dsh.profile.bundles` 里。

### 界面报错 / 侧边栏出不来

先确认桌面端版本。DS Bot 官方标注实测版本是 `dsh 0.2.1-alpha.1`，比它旧的桌面端（例如 `0.2.0-rc.2`）host 半边通常能正常加载，但界面可能出现异常。**升级桌面端**是首选解法。

### 升级 DS Bot

**插件管理页不会自动更新已安装的插件**（官方提示：「该插件已安装。如需升级，请卸载后重新安装」）。

不过 DS Bot 自带更新检查：有新版本时，**侧边栏右上角会出现一个云朵按钮**，点它就会通过 DSH 的插件管理器重装新版本，**下次启动 DSH 后生效**。

如果你是手工装的、或者不想等它检测，也可以卸载后重装：

```sh
dsh plugin --profile desktop remove ds-bot
dsh plugin --profile desktop add ds-bot --registry=https://registry.npmmirror.com/
```

---

## 卸载

**桌面端**：侧边栏 → **插件** → 在 **已安装** 里找到 `ds-bot` → 卸载。

**命令行**：

```sh
dsh plugin --profile desktop remove ds-bot
```

然后重启 DSH。

卸载**不会**删除 `~/.dsh/bot` 里的团队数据（Bot、群聊、记忆都还在）。想彻底清干净，卸载后手工删掉那个目录。

---

## 数据在哪

| 内容 | 路径 |
|---|---|
| Bot 团队状态（Bot、群聊、偏好） | `~/.dsh/bot/state.json` |
| 用量统计 | `~/.dsh/bot/usage.sqlite` |
| Bot 的工作目录 | `~/.dsh/bot/workspace/` |
| Bot 的会话 | `~/.dsh/sessions/` |
| 安装位置（profile） | `~/.dsh/profiles/desktop/` |

所有数据都在你自己的机器上。插件文档明确说明：卸载插件不会删除这些数据。

---

## 参考

- DS Bot 仓库：<https://github.com/FeiZhuLulu/DeepSeek-Bot>
- 插件源码与详细文档：[`ds-bot/README.md`](https://github.com/FeiZhuLulu/DeepSeek-Bot/blob/main/ds-bot/README.md)
- DeepSeek Harness：<https://github.com/deepseek-ai/deepseek-harness>

## 许可

本教程为社区文档，Apache-2.0。DS Bot 本身是独立社区项目，**不是 DeepSeek 官方产品**，与 DeepSeek 无隶属或背书关系。
