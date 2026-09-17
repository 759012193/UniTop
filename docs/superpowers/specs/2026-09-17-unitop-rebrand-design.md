# UniTop — TencentCloud/Octop 重命名派生 · 设计稿

| | |
|---|---|
| **状态** | Draft — 待用户 review 后进入 plan 阶段 |
| **日期** | 2026-09-17 |
| **作者** | Claude (brainstorming 协作产出) |
| **目标仓库** | `https://github.com/759012193/UniTop` |
| **本地路径** | `/Volumes/MacMini/unitop` |
| **上游** | `https://github.com/TencentCloud/Octop` (MIT) |
| **派生方式** | GitHub fork，保留全部 git 历史；通过 `unitop/branding` 长期分支承载所有品牌相关提交 |

---

## 0. 目标与非目标

### 0.1 目标（本次 spec 必须完成）
1. 将 `TencentCloud/Octop` 派生为 `759012193/UniTop`，保留全部上游 git 历史以利于后续 `rebase upstream/main`
2. **仅外观层**重命名 — 用户可见的"Octop"全部显示为"UniTop"；运行时标识（CLI `octop`、包名 `octop`、路径 `~/.octop/`、镜像名、Docker 镜像等）**保持不变**
3. 保留上游所有发布形式：Windows / macOS / Linux 桌面二进制、PyPI wheel、Docker 镜像、FnOS `.fpk`、一键安装脚本、便携版 zip
4. 桌面 App **新增** macOS 代码签名 + 公证，以及 Windows 代码签名（原本上游未签名）
5. 移除腾讯云主动推广内容（GitHub 组织、`octop.cloud`、COS 安装源 URL `finnie-1258344699.cos.ap-guangzhou.myqcloud.com`、WeCom 客服 QR、Tencent 套件连接器默认开启），**保留** `LICENSE` 文件、MIT 版权声明、致谢上游
6. 全部 CI / Release 目标由 `TencentCloud/Octop` 改为 `759012193/UniTop`

### 0.2 非目标（明确不做）
- 不重写 / 不重构上游业务逻辑、agent 协议、API、数据库 schema
- 不引入新的业务功能
- 不改变 Python 包名、CLI 命令、数据路径、环境变量前缀、Docker 镜像名（在功能层面），仅在用户可见层面替换品牌名
- 不剥离 MIT 要求的版权与原作者信息

---

## 1. 架构总览

### 1.1 项目本质
UniTop 是 Octop 的**派生品牌** — 同一份代码同一份架构，仅外观/品牌/发布目标不同。本 spec 不引入新业务功能，"架构"这一层几乎没有新东西，重点在"发布/打包管线"。

### 1.2 代码构成（沿用 Octop，不改动）
- 后端：Python 3.12 + FastAPI + SQLite/Postgres
- 前端：React 18 + TS + Vite + Ant Design
- 桌面打包：**Wails**（Go + WebView），打 `.dmg` / `.exe` / `.AppImage` / `.deb`
- 服务器打包：PyPI wheel `pip install octop`（包名不动）
- Docker：`docker pull <image>:<tag>`
- 一键安装：`install.sh`（bash）/ `install.ps1`（PowerShell）/ `install.bat`，从 GitHub Releases 拉桌面二进制

### 1.3 改动组织方式
本 spec **不引入新的顶层目录**。UniTop 与 Octop 的差异全部通过 **in-place 编辑原文件** 实现 — `README.md`、`desktop/src/build/darwin/Info.plist`、`pyproject.toml` 等文件就地修改，而不是创建 `unitop/branding/` 之类的覆盖层。

理由：你选了"最小改动"。in-place 编辑让 `git diff upstream/main` 直观显示所有 UniTop 增量；将来 `git rebase upstream/main` 时冲突发生在具体的文件里，不会被覆盖层抽象掉。

改动按用途归类到 4.4 的 10 个 commit，每个 commit 都可独立 `git revert` / `git rebase -i`。

### 1.4 数据流 / 错误处理 / 测试
与上游完全一致 — 任何业务行为不变。本项目不引入新数据流。

---

## 2. 文件级改动清单

按改动类别分组。强度：🟢 文本替换 / 🟡 改文件 + 引用 / 🔴 改构建产物（图标/图片）。

### A. 用户面向品牌 — 把"Octop"显示为"UniTop"
| 文件 | 改动 | 强度 |
|------|------|------|
| `README.md` + `README_CN.md` | 项目名、标题、副标题、banner 图片引用改为 UniTop；保留 `TencentCloud/Octop` 作为上游致谢链接 | 🟢 |
| `CHANGELOG.md` | 顶部加一条 `## [Unreleased] - UniTop fork`，说明本 fork 的去腾讯化与重命名 | 🟢 |
| `docs/assets/octop-banner.png`、`docs/assets/octop-logo.svg`、`docs/assets/readme-banner.png` | 替换为 UniTop banner / logo（用户提供素材后做） | 🔴 |
| `dashboard/src/index.html` | `<title>` 改为 UniTop；`og:title`、`description`、`apple-touch-icon` 引用替换 | 🟢 |
| `dashboard/src/assets/app/...`（应用内 Logo、设置页、关于页） | 替换图片/品牌字样 | 🟡 |
| `dashboard/src/i18n/locales/*` | UI 文案中"Octop"改为"UniTop"（**保留** `~/.octop/` 等路径字符串） | 🟢 |
| `dashboard/src/components/Layout/Sidebar*.tsx` 等 | 顶栏品牌字样替换 | 🟢 |

### B. 桌面客户端品牌
| 文件 | 改动 | 强度 |
|------|------|------|
| `desktop/src/build/darwin/Info.plist` + `Info.dev.plist` | `CFBundleName`、`CFBundleDisplayName`、`CFBundleIdentifier` 改为 `com.unitop.desktop` 之类 | 🟡 |
| `desktop/src/build/darwin/dmg-background.jpeg` | 替换为 UniTop dmg 背景 | 🔴 |
| `desktop/src/build/config.yml` + `windows/info.json` | 桌面 App 名称字段 | 🟡 |
| `desktop/src/build/windows/nsis/project.nsi` | NSIS 安装器显示名称、卸载器名 | 🟡 |
| `desktop/src/build/appicon.png`、`appicon-macos.png`、`tray-icon*.png`、`octop-mascot-*.webp` | 替换为 UniTop 图标 / 吉祥物 | 🔴 |
| `dashboard/public/apple-touch-icon.png` | 替换 | 🔴 |
| `desktop/src/build/darwin/Taskfile.yml`、`windows/Taskfile.yml`、`linux/Taskfile.yml` | 输出文件名前缀 `Octop-` → `UniTop-` | 🟢 |

### C. NAS / FnOS 包
| 文件 | 改动 | 强度 |
|------|------|------|
| `fnos/docker/manifest`、`fnos/docker/ICON.PNG`、`fnos/docker/ICON_256.PNG` | 包名、图标 | 🟡 |
| `fnos/native/...` 同上 | 同上 | 🟡 |
| `scripts/build-fpk.sh` | 输出包名 | 🟢 |

### D. 一键安装 — 重写 README 里的 COS one-liner URL
| 文件 | 改动 | 强度 |
|------|------|------|
| `README.md`、`README_CN.md`（4 处代码块：macOS/Linux curl、Windows PowerShell `irm`、Windows `.bat`、extras 变体） | 把 `https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.{sh,ps1,bat}` 替换为 `https://github.com/759012193/UniTop/releases/latest/download/install.{sh,ps1,bat}` | 🟢 |
| `scripts/install.sh`、`scripts/install.ps1`、`scripts/install.bat` | **不动** — 它们装的是 uv + `pip install octop[browser]`，走 PyPI，与品牌无关 | — |
| `scripts/install-octop.sh` | **不动**（这是 pip 安装用的，PyPI 包名仍是 `octop`） | — |
| 每个 GitHub Release 的 assets | 上传 `install.sh` + `install.ps1` + `install.bat` 作为 release assets，让上面 README 里的 URL 能拉到 | 🟢 |

**前置：UniTop 的每个 release 都必须包含 `install.sh` / `install.ps1` / `install.bat` 三个附件。** 这通过在 `release.yml` 的发布流程里 `actions/upload-artifact` 或直接包含在 release 工作流里实现。

### E. 剥离腾讯云主动推广（保留 LICENSE / NOTICE / 上游致谢）
| 文件 | 改动 | 强度 |
|------|------|------|
| `README.md`、`README_CN.md` | 删除 `## 💬 WeCom Customer Group (CN)` 整段及 `qrcode.png` 引用 | 🟢 |
| `README.md`、`README_CN.md` | `https://discord.gg/jPas5J8Ua` — **TODO** 占位（用户后续决定保留/删除） | 🟢 |
| `docs/assets/qrcode.png` | 删除文件 | 🔴 |
| `dashboard/src/assets/connectors/tencent-*.{png,svg}`、`tencent.svg`、`dashboard/src/assets/channels/octop.svg`、`yuanbao.svg` 等腾讯系连接器图标 | **不动文件**（属于功能资产，第三方仍可使用），但 `connectorDefs.tsx` 里把这些 connector 的 `defaultEnabled` 改为 `false`（不再预装） | 🟢 |
| `src/octop/infra/agents/experts/library/tencentcloud-api/`、`cvm-cluster-doctor/`、`cvm-ai-doctor/` | **不动文件**（属于内置技能库，与品牌无关） | — |
| `src/octop/infra/agents/experts/library/wechat-ops/` | **不动**（功能资产） | — |
| `dashboard/src/api/modules/connectors.ts` 等任何 `tencent-suite` 预设 bundle 配置 | 改为空 bundle 或删除"默认勾选腾讯套件" | 🟢 |

### F. 元数据修正（仅影响"对外可见的产品身份"，不动运行时）
| 文件 | 改动 | 强度 |
|------|------|------|
| `pyproject.toml` 的 `description`、`authors` | 改为 UniTop（注意：`name = "octop"` **不动** — 最小改动约束） | 🟢 |
| `dashboard/package.json` 的 `name`、`productName` | 改为 `unitop-dashboard`、`UniTop` | 🟢 |
| `desktop/` 下若有 `package.json` / `wails.json` / Go module name | 显示名改 UniTop；Go module path **不动** | 🟢 |
| `AGENTS.md` | 第一行加 "本仓库为 TencentCloud/Octop 的 UniTop 重命名派生" 声明 | 🟢 |

### G. CI / Release 目标
| 文件 | 改动 | 强度 |
|------|------|------|
| `.github/workflows/release.yml`、`octop-desktop.yml`、`docker-publish.yml`、`fnos-build-fpk.yml` | Release target 由 `TencentCloud/Octop` 改为 `759012193/UniTop` | 🟢 |
| `.github/workflows/ci.yml` | Badge URL 改 | 🟢 |
| `Makefile` | 任何 `gh release create ... --repo TencentCloud/Octop` 改为 `759012193/UniTop` | 🟢 |
| `.github/workflows/release.yml`（`publish` job）| **禁用 PyPI 发布** — `octop` 包在 PyPI 上由 TencentCloud 所有，UniTop 没有上传权；删除或 `if: false` 该 job。UniTop 用户继续 `pip install octop`（包名不变，即上游发布版本） | 🟢 |

### H. 明确**不动**的（最小改动约束）
- 任何 `src/octop/**/*.py` 里作为 Python 标识符的 `octop`（包路径、模块名、类名）
- `~/.octop/`、`/opt/octop/`、`OCTOP_*` 环境变量、CLI `octop` / `octop-update`
- Docker Hub 镜像路径段（如启用，见第 3 节 Q2 与 3.3）
- `octop` 出现在 `/octop/install.sh` 路径段中作为二进制名字的引用 — 替换为你自己的下载源，但 `octop` CLI 命令本身不动
- 上游 `LICENSE`、贡献者头像服务 `contrib.rocks/image?repo=tencentcloud/octop`（**保留**作为致谢）
- `pyproject.toml` 的 `name = "octop"`、`[project.scripts] octop = "octop.cli.main:cli"`

---

## 3. CI / Release 管线适配

### 3.0 关键发现
1. `octop-desktop.yml` 当前**没有签名/公证步骤** — 所有 `Octop-desktop-*.dmg` / `.exe` 都是裸包。本 spec 等于**新增**这条能力，不是保留。
2. 桌面栈是 **Wails**（Go + WebView），不是 Tauri — 签名入口是 `desktop/src/build/{darwin,windows}/Taskfile.yml`。
3. `docker-publish.yml` 同时推 Docker Hub 与 GHCR：
   - Docker Hub：`tencentcloud/octop`（依赖你账号下的 Docker Hub namespace — `<TODO>` 是否有 `759012193` 这个 Docker Hub 账号？如没有，沿用默认值：只推 GHCR）
   - GHCR：`ghcr.io/tencentcloud/octop` → 改为 `ghcr.io/759012193/unitop`
4. PyPI 发布走 `release.yml`，包名仍是 `octop` — 不动。

### 3.1 macOS 签名 + 公证（新增）

**需要的 GitHub Secrets：**

| Secret 名 | 内容 |
|-----------|------|
| `APPLE_TEAM_ID` | `<TODO>` 10 位 Apple Developer Team ID |
| `APPLE_KEY_ID` | App Store Connect API Key ID |
| `APPLE_ISSUER_ID` | App Store Connect API Issuer ID |
| `APPLE_API_KEY_P8` | `.p8` 文件的 base64 内容（`base64 -i AuthKey_XXXX.p8`） |
| `APPLE_CERT_P12` | "Developer ID Application: `<TODO>` 证书持有者姓名` (TEAMID)" 证书导出为 `.p12` 后的 base64 |
| `APPLE_CERT_PASSWORD` | `.p12` 密码 |
| `KEYCHAIN_PASSWORD` | 临时 keychain 的密码（自取任意强密码，例 `openssl rand -base64 32`） |

**追加到 `octop-desktop.yml` 的 `darwin-*` jobs 末尾：**

```yaml
- name: Import signing certificate
  env:
    P12: ${{ secrets.APPLE_CERT_P12 }}
    P12_PASSWORD: ${{ secrets.APPLE_CERT_PASSWORD }}
    KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
  run: |
    KEYCHAIN=build.keychain-db
    security create-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN
    security set-keychain-settings -lut 21600 $KEYCHAIN
    security unlock-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN
    echo "$P12" | base64 --decode > /tmp/cert.p12
    security import /tmp/cert.p12 -k $KEYCHAIN -P "$P12_PASSWORD" -T /usr/bin/codesign
    security set-key-partition-list -S apple-tool:,apple: -s -k "$KEYCHAIN_PASSWORD" $KEYCHAIN

- name: Codesign .app
  run: |
    # `<TODO>` 替换为 APPLE_CERT_P12 证书中的 "Common Name" 字段值
    # （通常是 "Developer ID Application: Your Name (TEAMID)" 中括号前的部分）
    SIGN_IDENTITY="Developer ID Application: <TODO: cert CN> (${{ secrets.APPLE_TEAM_ID }})"
    codesign --deep --force --options runtime --timestamp \
             --sign "$SIGN_IDENTITY" \
             UniTop.app

- name: Notarize .dmg
  env:
    APPLE_KEY_ID: ${{ secrets.APPLE_KEY_ID }}
    APPLE_ISSUER_ID: ${{ secrets.APPLE_ISSUER_ID }}
    APPLE_API_KEY_P8: ${{ secrets.APPLE_API_KEY_P8 }}
  run: |
    echo "$APPLE_API_KEY_P8" | base64 --decode > /tmp/AuthKey_$APPLE_KEY_ID.p8
    xcrun notarytool submit UniTop.dmg \
      --key /tmp/AuthKey_$APPLE_KEY_ID.p8 \
      --key-id "$APPLE_KEY_ID" --issuer "$APPLE_ISSUER_ID" \
      --wait
    xcrun stapler staple UniTop.dmg
```

### 3.2 Windows 代码签名（路径：**W1 Azure Trusted Signing**）

| 路径 | 适用 | 状态 |
|------|------|------|
| **W1 · Azure Trusted Signing（默认）** | 无 EV USB Key；`signtool` 通过 OIDC 登录 Azure | 默认采用 |
| W2 · 传统 OV `.pfx` 证书 | 已有 `.pfx` | 备选 |
| W3 · EV 证书 + HSM | 最佳 SmartScreen 体验 | 备选 |

> `<TODO>` 用户后续确认采用 W1 / W2 / W3

**W1 需要的 GH Secrets：**

| Secret | 内容 |
|--------|------|
| `AZURE_TENANT_ID` | Azure AD tenant |
| `AZURE_CLIENT_ID` | 注册应用的 client ID |
| `AZURE_CLIENT_SECRET` | 对应 client secret |
| `AZURE_SUBSCRIPTION_ID` | 含 Trusted Signing 账号的订阅 |
| `AZURE_DLIB_PATH` | `.dlib` 上传至 repo 后的相对路径（如 `desktop/src/build/windows/trusted.dlib`） |

**追加到 `octop-desktop.yml` 的 `windows-*` jobs：**

```yaml
- name: Azure login
  uses: azure/login@v2
  with:
    creds: '{"clientId":"${{ secrets.AZURE_CLIENT_ID }}","clientSecret":"${{ secrets.AZURE_CLIENT_SECRET }}","subscriptionId":"${{ secrets.AZURE_SUBSCRIPTION_ID }}","tenantId":"${{ secrets.AZURE_TENANT_ID }}"}'

- name: Sign .exe
  uses: azure/trusted-signing-action@v0
  with:
    azure-signing-path: ${{ secrets.AZURE_DLIB_PATH }}
    endpoint: https://eus.codesigning.azure.net/
    files:
      - UniTop.exe
      - UniTop-portable-windows-amd64.exe
    file-digest: SHA256
    timestamp-digest: SHA256
    timestamp-rfc3161: http://timestamp.digicert.com
```

### 3.3 Docker 镜像发布
**改动 `docker-publish.yml`：**
- GHCR：`ghcr.io/tencentcloud/octop` → `ghcr.io/759012193/unitop`
- Docker Hub：
  - 默认：`<TODO>` 用户是否有 `759012193` 的 Docker Hub 账号？如无 → 删掉 Docker Hub login + tag 步骤，只推 GHCR
  - 如有：保留 login，但路径 `${DOCKERHUB_USERNAME}/octop` → `${DOCKERHUB_USERNAME}/unitop`（需 Docker Hub 上有 `unitop` 命名空间）
- 镜像标签：`{version}` + `latest`（保留）

### 3.4 FnOS `.fpk` 包
`fnos-build-fpk.yml` 触发条件是 `workflow_run` on `release.yml` 完成 — 自动，无需改 trigger。但要把：
- `manifest` 文件里 `name` 改为 `unitop`
- `ICON.PNG` 替换（见第 2 节 C 类）
- workflow 内任何对 `TencentCloud/Octop` 的引用改为 `759012193/UniTop`

### 3.5 GitHub Release 产物重命名
`octop-desktop.yml` 的产物前缀：
```
Octop-portable-{plat}-{VER}.zip      →  UniTop-portable-{plat}-{VER}.zip
Octop-desktop-{plat}-{VER}.{ext}     →  UniTop-desktop-{plat}-{VER}.{ext}
```
文本替换 — 把 `octop-desktop.yml` 里所有 `Octop-` 改为 `UniTop-`。

### 3.6 桌面 App 自更新 endpoint
上游桌面 App 通过 GitHub Releases 检查更新。Release URL 已自动跟随 `${{ github.repository }}` → 改为 `759012193/UniTop`，无额外改动。

---

## 4. Fork 与本地克隆流程

### 4.1 在 GitHub 上 fork
任选：
- (a) 浏览器到 `https://github.com/TencentCloud/Octop` → 点 **Fork** → owner `759012193` → 仓库名 `UniTop`
- (b) CLI：
  ```bash
  gh repo fork TencentCloud/Octop --org 759012193 --name UniTop --remote
  ```

### 4.2 本地克隆（默认 `/Volumes/MacMini/unitop`）
```bash
mkdir -p /Volumes/MacMini
cd /Volumes/MacMini
gh repo clone 759012193/UniTop unitop
cd unitop
git remote add upstream https://github.com/TencentCloud/Octop.git
git fetch upstream
```

### 4.3 分支策略
- 主分支 `main`（fork 默认）
- 长期分支 `unitop/branding` — 所有品牌/腾讯剥离/CI 适配的提交都进这里
- 上游新版本同步：`git fetch upstream && git checkout main && git merge upstream/main`，然后 `git checkout unitop/branding && git rebase main`

### 4.4 提交粒度（便于 review 与 rebase）
```
unitop/branding
├── 1. docs: rebrand README + add upstream attribution
├── 2. desktop: rename Octop → UniTop in build configs (Info.plist, NSIS, Taskfile)
├── 3. desktop: swap app icons + mascot + dmg background + tray icons
├── 4. dashboard: rebrand UI copy + title + i18n
├── 5. install: redirect README one-line install URLs from COS to UniTop releases (scripts themselves unchanged)
├── 6. ci: retarget release/docker/fnos workflows to 759012193/UniTop
├── 7. ci: add macOS codesign + notarytool steps
├── 8. ci: add Windows Azure Trusted Signing step
├── 9. tencent: disable Tencent connector default + remove WeCom QR
└── 10. metadata: update pyproject description + dashboard package name
```

每步单独 commit，每个 commit 都可独立 `git revert`，未来 `git rebase upstream/main` 冲突也只发生在 1~2 个文件里。

---

## 5. 验证清单

| 验证项 | 怎么跑 | 通过标准 |
|--------|--------|----------|
| **Python wheel** | `make build` → 检查 `dist/` | wheel 名字仍是 `octop`；`pip install dist/octop-*.whl --force-reinstall` 成功；`octop --version` 输出当前版本 |
| **Docker 镜像** | `docker build -f docker/Dockerfile .` | 构建成功；`docker run` 能启动 web dashboard |
| **macOS `.dmg`** | `git tag v0.0.1-unitop-test && git push origin v0.0.1-unitop-test` → 触发 GH Actions | 文件名改为 `UniTop-desktop-darwin-arm64-*.dmg`；`codesign -dv` 输出 Developer ID；`spctl -a -t install -v UniTop.dmg` 输出 `accepted` |
| **Windows `.exe`** | 同上 + Windows runner | 文件名改为 `UniTop-desktop-windows-amd64-*.exe`；`signtool verify /pa UniTop.exe` 输出 `Signed` |
| **Windows 便携版** | 同上 | `UniTop-portable-windows-amd64-*.zip` 解压后双击 `UniTop.exe` 能起服务 |
| **Linux `.AppImage` / `.deb`** | `linux-amd64` + `linux-arm64` runner | chmod +x 后在 Ubuntu 22.04/24.04 上启动 |
| **FnOS `.fpk`** | 由 `release.yml` 自动触发 `fnos-build-fpk.yml` | artifact 上传成功；FnOS 模拟器或真机可装 |
| **一键安装脚本（README URL）** | `grep -E "finnie.*myqcloud" README.md README_CN.md` 应无输出；`grep -E "759012193/UniTop/releases/latest/download/install" README.md` 至少 3 处 | URL 字符串替换完整 |
| **一键安装脚本（脚本未改）** | `diff scripts/install.sh <upstream>/scripts/install.sh` 应无差异；本地 bash `bash scripts/install.sh --help` 仍打印正常用法 | 脚本本体未受品牌改动影响 |
| **Release assets** | `curl -I -L https://github.com/759012193/UniTop/releases/latest/download/install.sh` | HTTP 200，文件可拉取 |
| **Web 面板文案** | 启动后浏览器打开 `http://localhost` | 顶栏、设置、关于页全部显示 "UniTop"；品牌图标正确 |
| **PyPI 发布** | **UniTop 不发布**（包归 TencentCloud）；`release.yml` 的 `publish` job 应被 `if: false` 或删除 | `pip install octop==<ver>` 由上游发布，UniTop 用户无变化 |
| **PyPI 步骤已被禁用** | 推送一个 tag → 检查 GH Actions 日志 | `release.yml` 中 `publish` job 不再执行，没有 403/鉴权失败 |

### 5.1 上线顺序（推荐）
1. **先本地跑通 Docker + Python wheel**（无需签名，最快反馈循环）
2. **打测试 tag `v0.0.1-unitop-test1`** → 跑 macOS / Windows 签名验证 → 检查签名后 `.dmg`/`.exe` 是否能正常打开
3. **触发 FnOS `.fpk`** 验证 NAS 打包链路
4. **修好任何 CI 报错 → 打正式 `v1.0.0-unitop.1`** → 第一次正式发版

### 5.2 验证不过的 rollback
- 不动 `git tag -d` 那个错的 tag（已经推出去的 tag 无法物理删除）
- 删除 GitHub Release（不要 force-push `main`，会断历史）
- `git revert <bad commit>` 推送修复，重新打 tag

---

## 6. 待用户填的占位（Open Questions）

> 这些在 plan 阶段会作为前置依赖列出来；实施前必须填。

| # | 项 | 当前默认 | 用户需提供 |
|---|----|----------|------------|
| Q1 | Windows 签名路径 | **W1 Azure Trusted Signing** | 确认采用 / 改用 W2（OV `.pfx`） |
| Q2 | Docker Hub 账号 | **不推 Docker Hub，仅 GHCR** | 是否有 `759012193` Docker Hub 账号？有 → 改 Docker Hub 路径 |
| Q3 | Apple Developer Team ID | `<TODO>` | 10 位 Team ID |
| Q4 | Discord 链接保留 | **保留** | 保留 / 删除 README 中 `https://discord.gg/jPas5J8Ua` |
| Q5 | UniTop 品牌素材 | `<TODO>` | banner、logo、app icon、tray icon、mascot、dmg 背景图 |
| Q6 | Wails Windows `.dlib` | `<TODO>` | Azure Trusted Signing 颁发的 `.dlib` 文件 |

---

## 7. 引用

- 上游仓库：`https://github.com/TencentCloud/Octop`
- 上游 LICENSE：`https://github.com/TencentCloud/Octop/blob/main/LICENSE`（MIT）
- 上游 docs：`https://github.com/TencentCloud/Octop/tree/main/docs`
- 上游 release workflow：`https://github.com/TencentCloud/Octop/blob/main/.github/workflows/release.yml`
- 上游 desktop workflow：`https://github.com/TencentCloud/Octop/blob/main/.github/workflows/octop-desktop.yml`
- 上游 docker workflow：`https://github.com/TencentCloud/Octop/blob/main/.github/workflows/docker-publish.yml`
- Apple notarytool 文档：`https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution`
- Azure Trusted Signing action：`https://github.com/Azure/trusted-signing-action`