# DSH 手机连接配置手册

在**另一台 Mac** 上，把 DSH 配成手机 App 可以连接的状态。

本文所有命令与结论都来自 2026-10-09 在 macOS 27 + DeepSeek Harness `0.2.0-rc.2` 上的实测，
踩过的坑都写在「已知坑速查表」里。

---

## ⚠️ 先读：版本兼容性

**手机 App 目前只兼容 DSH `0.1.x`。跑在 `0.2.x` 上，手机连得上、配得上对，但用不了。**

App 是按 `dsh-v0.1.1-rc.2` 的 RPC 方法表写的（点号命名，如 `session.list`）。
DSH 从 `0.2.0` 起把整套接口改成了**斜杠命名**（`session/list`），而且不只是分隔符变了——
方法还改了名、有的直接移除。实测对照（App 用到的 30 个方法 vs DSH `0.2.0-rc.2`）：

| | |
|---|---|
| 新版本里同名存在 | **17** 个 |
| 新版本里**已不存在** | **13** 个 |

不存在的 13 个：

```
agentPreset.list     goal.clear       goal.edit        goal.pause
goal.resume          host.createDirectory               host.describe
host.listDirectory   llm.providers    session.history  session.models
skill.list           workspace.list
```

部分在新版里换成了别的名字，例如
`host.listDirectory` → `directoryPicker/list`、`llm.providers` → `llm/listProviders`、
`session.models` → `session/modelCatalog`、`agentPreset.list` → `agentPresets/list`。

**失败表现**：手机扫码能配对成功，隧道也建起来了（Relay 上能看到活动会话），
但界面一直停在 **「扫码连接正在同步，暂不能发送」**，永远发不出消息。

原因在 App 的同步基线：它必须**先成功调用 `host.describe`、`workspace.list`、`session.list`**
才会进入就绪状态（`isProductReady()`）；其中任意一个 404，基线就永远建不起来。

**所以**：要用这个手机 App，电脑端的 DSH 请用 **`0.1.x`**（App 作者实测的是 `dsh-v0.1.1-rc.2`）。
跑 `0.2.x` 时不要再去排查网络、Relay、nginx、证书——那些都是好的，问题在接口层。

> 本文其余部分（插件安装、Relay 配置、nginx、隧道验证）在 `0.2.x` 上**同样适用**，
> 因为它们只涉及传输层。不兼容的只有 App 与 DSH 之间的 RPC 接口。

---

## 0. 这套东西怎么工作

```
电脑                                          手机
DSH 只监听 127.0.0.1:<port>   ← 永远不对外暴露
        ↑ 本机回环
    dsh-scan-remote 插件 ──出站 WSS──→ Relay ←──出站 HTTPS/WSS── App 扫码
                                     ↑
                            二维码里的 publicRelayUrl
                            必须是手机能访问到的地址
```

三个要点：

1. **Relay 只是会合点**，它自己够不到你电脑的回环端口。必须由电脑上的**插件主动连出去**。
2. **二维码里的 `pairId` + `masterKey` 由插件生成**。手机不会内置任何域名，一律读二维码 fragment。
3. 隧道是端到端加密的（`sealed-tunnel-v1`），Relay 只转发密文，看不到 DSH 业务内容。

---

## 1. 前置条件

| 项 | 说明 |
|---|---|
| Mac | 装了 DeepSeek Harness（GUI App 即可，自带 `dsh` CLI） |
| Relay | 一个手机能访问的地址。**公网部署必须 HTTPS/WSS**（iOS ATS 会拦明文 HTTP） |
| iPhone | 装了 DSH Mobile，且已信任开发者证书 |
| 网络 | 装插件时需要能访问 GitHub 与 npm |

---

## 2. 第 1 步：把插件装进一个 DSH profile

### 2.1 先解决 npm 源（否则必然卡死）

两个坑叠在一起：

- `registry.npmjs.org` 在国内直连不通，pnpm 会**卡在 SYN_SENT 很久**
- **实测 `export npm_config_registry=https://registry.npmmirror.com` 在这里不生效**，pnpm 照样去连 npmjs

唯一可靠的做法是写 **profile 级 `.npmrc`**：

```sh
# 把 <profile> 换成你要用的 profile 名，例如 web
printf 'registry=https://registry.npmmirror.com/\n' > ~/.dsh/profiles/<profile>/.npmrc
```

### 2.2 选哪个 profile

| profile | CLI 装插件 | CLI 启动 | 说明 |
|---|---|---|---|
| `web` | ✅ | ✅ | **本文推荐**：用一个独立的 headless 实例当 host |
| `desktop` | ✅ | ❌ | 桌面端 GUI 用的 profile。能装插件，但**配对会被版本检查挡住**（见第 3 节末尾），且不能从命令行启动 |
| 其它自建 | ✅ | ✅ | `dsh --from-default-profile web` 可基于模板新建 |

关于 `desktop` 的限制，**只针对「启动这个 profile」，不针对插件管理**：

```sh
dsh plugin --profile desktop list     # ✅ 可以，插件管理畅通
dsh plugin --profile desktop add ...  # ✅ 可以
dsh --profile desktop                 # ❌ error: profile "desktop" is managed exclusively by the Electron application
```

### 2.3 安装

```sh
DSH="/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh"
"$DSH" plugin --profile web add "github:yukiykchen/dsh-scan-remote#v0.0.1"
```

> 如果这台机器是从源码跑 DSH，等价命令是 `pnpm dsh plugin --profile web add ...`；
> 如果能正常访问 npm 源，也可以是 `npx @deepseek-ai/dsh plugin ...`。

**装成功的标志**：`~/.dsh/profiles/web/package.json` 里出现

```json
"dependencies": { "@yukiykchen/dsh-scan-remote": "github:yukiykchen/dsh-scan-remote#v0.0.1" },
"dsh": { "profile": { "bundles": [ ..., "@yukiykchen/dsh-scan-remote" ] } }
```

**注意 `bundles` 那一项**：如果装完只有 `dependencies`、`bundles` 里没有，插件不会被加载，
需要手动把 `"@yukiykchen/dsh-scan-remote"` 加进 `bundles` 数组。

### 2.4 如果 git 报 `SSL_ERROR_SYSCALL`

pnpm 会用**全局 git 代理配置**去 clone。代理没开就会这样：

```
fatal: unable to access 'https://github.com/...': LibreSSL SSL_connect: SSL_ERROR_SYSCALL
```

两个选择：把代理打开；或者对这一次安装绕过代理（不改全局配置）：

```sh
export GIT_CONFIG_COUNT=2 \
       GIT_CONFIG_KEY_0=http.proxy  GIT_CONFIG_VALUE_0= \
       GIT_CONFIG_KEY_1=https.proxy GIT_CONFIG_VALUE_1=
```

---

## 3. 第 2 步：让插件连上你的 Relay

两个环境变量分工不同，**别搞混**：

| 变量 | 作用 | 远程 Relay 时的值 |
|---|---|---|
| `DSH_RELAY` | **电脑端插件**连 Relay 的地址 | `https://<你的域名>` |
| `PUBLIC_RELAY_URL` | 写进**二维码**、给手机用的 origin | 同上（远程部署时两者相同） |
| `DSH_DEVICE_NAME` | App 里显示的电脑名（可选） | 例如 `我的 MacBook` |

### 方式 A：命令行启动（推荐）

```sh
cd ~          # ← 重要，见下面的坑
export DSH_RELAY=https://relay.example.com
export PUBLIC_RELAY_URL=https://relay.example.com
export DSH_DEVICE_NAME="我的 MacBook"

"/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh" \
  --profile web --port 3080 --no-open
```

然后浏览器打开启动日志里打印的地址（形如 `http://127.0.0.1:3080/?token=...`），
进 **Settings → Remote Access**，那里就是二维码。

### 方式 B：GUI 启动（没有环境变量时）

GUI App 从 Finder 启动，**拿不到你 shell 里的环境变量**，插件会退回默认的
`relay: http://127.0.0.1:8787`（错误的地址）。这时用 profile 的用户层 patch 覆盖：

编辑 `~/.dsh/profiles/<profile>/cordis.patch.yml`，追加：

```yaml
- id: dsh-scan-remote
  name: "@yukiykchen/dsh-scan-remote"
  config:
    relay: https://relay.example.com
    dshPort: !!js ctx.webStartup.port ?? 19387
```

要点：

- **patch 的 `config` 是整体替换，不是合并**，所以 `relay` 和 `dshPort` 两个必填项都要写
- `dshPort` 用 `!!js ctx.webStartup.port ?? <默认值>`，这样端口变了不用改配置
- 改完重启 DSH

### ⚠️ 必须从中性 CWD 启动（版本检查的坑）

插件里有个版本兼容检查（`dist/dsh-compat.js`）：

```js
const detectedVersion = detectDshVersion(process.cwd());   // 只看 CWD
compatible = detectedVersion === EXPECTED_DSH_VERSION;      // 插件期望 0.1.0-rc.6
compatible: detectedVersion == null ? true : compatible;    // 探测不到就算通过
```

`detectDshVersion` 只从**进程 CWD** 找 `node_modules/@deepseek-ai/dsh/package.json`。
于是同一份 DSH 代码，换个 CWD 结论就反过来（实测）：

| 启动时的 CWD | 探测结果 | 判定 |
|---|---|---|
| `~/.dsh/profiles/desktop`（GUI App 的 CWD） | `0.2.0-rc.2` | ❌ 不兼容，**拒绝新建配对** |
| `~` 或任意普通目录 | `null` | ✅ 通过 |

**所以：CLI 启动时先 `cd ~`（或任何不含 `node_modules/@deepseek-ai/dsh` 的目录）。**

这个检查**只拦「新建配对」**，不拦隧道转发和已有配对。

---

## 4. 第 3 步：服务器端 nginx（最大的坑）

**症状**：健康检查 200、网页 200，但插件日志一直刷

```
Relay disconnected; retrying in 1s / 2s / 5s / 10s / 30s
```

**原因**：WebSocket 升级头没被转发，握手被当成普通 HTTP 请求，Relay 返回
`426 Upgrade Required {"reason":"e2ee_required"}`。

插件连的是 **`/device`**（`relay-client.js` 里写死的 `url.pathname = "/device"`）。
**只给 `/client-tunnel` 配升级头是不够的** —— `/device` 会落进通用 `/` location。

`nginx.conf` 的 `http {}` 块里加 map：

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

server 块：

```nginx
location / {
    proxy_pass http://127.0.0.1:8787;

    proxy_http_version 1.1;                          # 默认 1.0 不支持 upgrade
    proxy_set_header Upgrade    $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_read_timeout 3600s;   # 隧道是持久长连接，默认 60s 会被掐断
    proxy_send_timeout 3600s;
    proxy_buffering    off;
}
```

生效：

```sh
sudo nginx -t && sudo systemctl reload nginx
```

**不用重启 DSH**，插件 30s 一次的重连会自动连上。

---

## 5. 验证（逐层，别跳步）

```sh
RELAY=https://relay.example.com
PORT=3080        # GUI App 的话是它自己的端口，例如 19387

# ① Relay 活着吗
curl -s $RELAY/health                       # 期望 {"ok":true}

# ② WebSocket 升级通吗（决定性的一条）
curl -s -i -N --http1.1 --max-time 10 \
  -H "Connection: Upgrade" -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" \
  $RELAY/device
# 期望 HTTP/1.1 101 Switching Protocols
# 若返回 426 e2ee_required → 回到第 4 步修 nginx
# ⚠️ 必须加 --http1.1，HTTP/2 下 Upgrade 头无效，会得到假阴性

# ③ 插件状态
curl -s http://127.0.0.1:$PORT/dsh-scan-remote/api/state
# 期望 relayConnection=connected、dsh=online、dshCompatible=true

# ④ 真的有隧道（最硬的证据）
lsof -nP -iTCP:443 | grep ESTABLISHED | grep -c "$(dig +short ${RELAY#https://})"
```

③ 的字段含义：

| 字段 | 期望 | 不对时看哪 |
|---|---|---|
| `relayConnection` | `connected` | `connecting`/`offline` → 第 4 步 nginx |
| `dsh` | `online` | `offline` → 插件摸不到本机 DSH，检查 `dshPort` |
| `dshCompatible` | `true` | `false` → 第 3 节的 CWD 坑 |
| `phase` | `paired` 或 `unpaired` | `unpaired` 说明还没配对过，正常 |

---

## 6. 手机端配对

1. 电脑浏览器打开 DSH → **Settings → Remote Access** → 出现二维码
2. iPhone 打开 App → **扫码连接** → **扫描电脑二维码**（首次会申请相机权限）
3. 配对成功后点 **连接已配对电脑**

### ⚠️ App 目前只保存一台电脑

换电脑前必须先点 **「移除这台电脑」**，再扫新码。

### ⚠️ 二维码有倒计时

过期就刷新页面重新生成。**改过 relay 地址或端口后必须重启 DSH 再扫新码**，
否则 Settings 里还是旧码。

---

## 7. 常驻运行

手机要随时能连，host 就得一直开着。三种做法，按省事程度排：

1. **GUI App 常开**（但 GUI 用的是 `desktop` profile，见下方注意事项）
2. 终端里 `nohup` 挂着：
   ```sh
   cd ~ && nohup env DSH_RELAY=... PUBLIC_RELAY_URL=... \
     "/Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh" \
     --profile web --port 3080 --no-open > ~/dsh-web.log 2>&1 &
   ```
3. launchd 开机自启（`~/Library/LaunchAgents/*.plist`，`RunAtLoad` + `KeepAlive`）

### ⚠️ 同一时间只能有一个 host

hostId / hostToken / E2EE 密钥存在**全局**的 `~/.dsh-scan-remote/config.json`，
**不区分 profile**。两个实例同时跑会抢同一个 hostId，Relay 上互相顶掉，
表现为状态在 `connected` / `connecting` 之间反复跳。

要切换实例，**先把旧的停掉**。

---

## 8. 已知坑速查表

| 现象 | 原因 | 处理 |
|---|---|---|
| 插件安装卡住不动，`lsof` 看到连 `registry.npmjs.org` 的 SYN_SENT | npmjs 被墙 + 环境变量没透传 | 写 profile 级 `.npmrc` 指向 npmmirror（2.1） |
| `git ls-remote ... SSL_ERROR_SYSCALL` | 全局 git 代理指向一个没开的代理 | 开代理，或用 `GIT_CONFIG_COUNT` 绕过（2.4） |
| 健康检查 200 但一直 `Relay disconnected` | nginx 没转发 WebSocket 升级头 | 第 4 步 |
| `dshCompatible: false`，无法新建配对 | CWD 在 profile 目录，版本探测到不匹配 | 从中性目录启动（第 3 节末尾） |
| `bundles` 里没有插件名 | 安装只写了 dependencies | 手动加进 `bundles` 数组 |
| 手机一直「连接重试中」 | 上面任一环没通 | 按第 5 节 ①→④ 逐层查 |
| 状态在 connected/connecting 之间跳 | 两个实例抢同一 hostId | 停掉多余实例（第 7 节） |
| 手机能连但换个 Wi-Fi 就超时 | 二维码 origin 是旧地址 | 改 `PUBLIC_RELAY_URL` → 重启 DSH → 扫新码 |
| `profile "desktop" is managed exclusively...` | CLI 拒绝**启动** desktop profile | 插件管理不受限（`dsh plugin --profile desktop ...` 可用）；要启动就用 `web` 等其它 profile |

---

## 9. 实测参考值

> 下面是本文验证时的环境与关键路径。`relay.example.com` 是**占位符**，换成你自己的域名。

```
Relay             https://relay.example.com   （nginx 1.24.0 / Ubuntu 实测）
Relay 证书         你自己的证书 CN ⚠️ 手动 DNS 验证签发的不会自动续期，到期前记得续
DSH               0.2.0-rc.2（⚠️ 只能连 0.1.x，见文首兼容性一节）
插件              @yukiykchen/dsh-scan-remote v0.0.1（期望 DSH 0.1.0-rc.6）
CLI               /Applications/DeepSeek Harness.app/Contents/Resources/runtime/cli/bin/dsh
App 自带 pnpm      11.7.0（runtime/pnpm/bin/pnpm.cjs）
本机 Relay 端口     8787（局域网备用方案）
DSH web 端口       3080（CLI）/ 19387（GUI App）
全局插件配置        ~/.dsh-scan-remote/config.json
```

### 局域网备用方案

不想折腾 nginx 时，Relay 也能跑在本机：

```sh
cd ~/dsh-scan-remote/relay && cp .env.example .env
# .env 里设 HOST=0.0.0.0 PORT=8787 PUBLIC_RELAY_URL=http://<电脑LAN IP>:8787
# JWT_SECRET 用 openssl rand -hex 32 生成
npm ci && npm run build && npm start

# 另一个终端
export DSH_RELAY=http://<电脑LAN IP>:8787
export PUBLIC_RELAY_URL=http://<电脑LAN IP>:8787
cd ~ && dsh --profile web --port 3080 --no-open
```

代价：**手机必须和电脑在同一个 Wi-Fi**，通勤/开会那种场景就没了。

---

## 10. 回退

```sh
# 卸载插件
"$DSH" plugin --profile web remove @yukiykchen/dsh-scan-remote

# 清掉配对状态（换机器/重新配对时）
rm ~/.dsh-scan-remote/config.json
```

GUI 的 `desktop` profile 若被手工改过，把 `package.json` 与 `cordis.patch.yml`
恢复原样、并确认 `dsh.profile.bundles` 里没有 `@yukiykchen/dsh-scan-remote`，
重启 App 即可。

---

## 参考

- DSH Mobile（手机 App）：<https://github.com/yukiykchen/deepseek-harness-mobile>
- 扫码插件与 Relay：<https://github.com/yukiykchen/dsh-scan-remote>
- DeepSeek Harness：<https://github.com/deepseek-ai/deepseek-harness>

## 许可

本教程为社区文档，Apache-2.0。DSH Mobile 与 `dsh-scan-remote` 均为独立社区项目，
**不是 DeepSeek 官方产品**，与 DeepSeek 无隶属或背书关系。
