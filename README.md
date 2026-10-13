# Lunac 插件分发仓库（lunac-plugins）

> Lunac 宿主（[LythrumMoon/Lunac](https://github.com/LythrumMoon/Lunac)）的**官方插件分发仓库**。
> 宿主通过常量 `plugin_market::INDEX_URL` 拉取本仓库根目录的 `index.json`，据此在
> **设置 → 插件** 列出可下载的插件，并从 `packages\` 下载对应的 zip 安装。

⚠️ 本仓库是**个人非商用项目**的配套分发渠道，授权义务见下文「四、第三方授权与传递义务」。

---

## 一、仓库结构

```
lunac-plugins/
  index.json                      市场索引（宿主直接消费的清单）
  packages/
    productivity/   <id>-<版本>.zip
    media/          <id>-<版本>.zip
    ocr/            <id>-<版本>.zip
    pet/            <id>-<版本>.zip
    ai/             <id>-<版本>.zip
    system/         <id>-<版本>.zip
    other/          <id>-<版本>.zip   （未登记分类的插件落这里）
  README.md                       本文件
```

- **`index.json`**：一个 JSON 数组，每条只描述「去哪下一个插件」。字段与宿主 Rust 侧
  `plugin_market::PluginIndexEntry` 一一对应：
  `id` / `name` / `description` / `version` / `url` / `keywords` / `icon` / `homepage`。
  其中 **`url` 是必须为 https 的完整直链**（本仓库用 `raw.githubusercontent.com`）。宿主**直接用
  这一整条 url** 去下载（不做「基址 + 固定路径」拼接），所以把包放进 `packages\<分类>\` **不会**
  影响用户拉取 —— 只要 `url` 里同步带上分类目录即可。
- **`packages\<分类>\`**：插件压缩包，每个文件名是 `<id>-<版本>.zip`。分类只是**整理方式**，
  **不改变下载 / 安装 / 升级语义**。

分类与现有插件：

| 分类 | 说明 | 现有插件 |
|------|------|----------|
| `productivity` | 效率 | `convert`（文件转换）、`clipboard-history`（剪贴板历史） |
| `media` | 媒体 | `music`（音乐 · 歌词 / Spotify 控制） |
| `ocr` | 文字识别 | `ocr` |
| `pet` | 桌宠 | `pet` |
| `ai` | 本地 AI | `local-model`（本地决策模型） |
| `system` | 系统 | `proxy`（系统代理） |

> 分类由发布脚本内的 `$Categories` 表（id → 目录名）决定；新增插件先在那里登记，否则落到 `other/`。

## 二、每个插件只保留最新版本

发布脚本（应用仓库的 `scripts\publish-plugins.ps1`）在发布时会：

1. 只挑**每个插件 id 的最高版本**那个 zip 进 `packages\`；
2. 清掉 `packages\` 里**所有不在本次发布集合里**的 zip —— 既包括同一个 id 的旧版本，也包括
   早先平铺、如今已改到分类子目录的旧包。

所以本仓库**每个插件 id 恒只有一份、且是最新版的包**；用户的市场列表里不会出现同一插件的两条记录。

## 三、用户怎么拉取

1. 打开 Lunac → **设置 → 插件**；
2. 宿主**现拉** `index.json`（每次打开设置面板时拉，宿主本身不落盘缓存；唯一的外部缓存是
   GitHub raw 的 CDN —— 改完索引约几分钟内可见）；
3. 市场把「索引版本 ≠ 本机已装版本」显示为**更新**（只判「不同」，不判大小）；点「下载 / 更新」
   即由宿主从该条 `url` 下载 zip 安装，插件 `lunac-plugin.json` 里声明的 `dependencies` 会一并
   下载并按 `sha256` 校验，**任一失败则整包回滚**。

装完**立即生效，不必重启**。带大资产的插件（桌宠的 Live2D Core / ZipVoice 语音包）首次安装会额外
从本仓库的 **Release** 资产下载，见下节。

## 四、第三方授权与传递义务

本仓库分发的**插件代码本身**是 Lunac 原创代码（随应用仓库的 MIT，仅覆盖原创部分）；但部分插件在
安装时会**下载并携带**第三方组件（在各插件 `lunac-plugin.json` 的 `dependencies` 里声明）：

| 组件 | 授权 | 分发方式 | 主要义务 |
|------|------|----------|----------|
| [Live2D Cubism Core](https://www.live2d.com/en/sdk/download/web/) | **专有许可** | Release 资产 `live2dcubismcore-5.1.0`（`file` 依赖） | 保留官方 LICENSE 与可再分发文件清单（协议 5.1 随附义务） |
| [ZipVoice](https://github.com/k2-fsa/ZipVoice) 预训练权重 | **CC BY-NC 4.0（仅非商用）** | Release 资产 `zipvoice-1.13.8`（`archive` 依赖） | 署名 + 附许可全文 + 标注改动 + **不得商用** |
| [eSpeak NG](https://github.com/espeak-ng/espeak-ng) 数据 | **GPL-3.0** | 同上（随 ZipVoice 资产） | 附 GPL-3.0 全文 + 提供对应源码获取途径 |
| [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) | Apache-2.0 | 同上 | 保留版权与许可声明 |
| [ONNX Runtime](https://github.com/microsoft/onnxruntime) | MIT | 同上 | 保留版权与许可声明 |
| [Vocos](https://huggingface.co/charactr/vocos-mel-24khz) | MIT | 同上 | 保留版权与许可声明 |
| [librespot](https://github.com/librespot-org/librespot) | MIT | Release 资产 `librespot-0.8.0`（`file` 依赖） | 保留版权与许可声明 |
| [PaddleOCR-json](https://github.com/hiroi-sora/PaddleOCR-json) | Apache-2.0 | `ocr` 插件的 `archive` 依赖（上游 Release 直链） | 保留版权与许可声明 |

> **本项目（Lunac 及其官方插件）为个人项目，明确非商用，仅在 GitHub 公开分发。**
> 随插件下载的 **ZipVoice 预训练权重为 CC BY-NC 4.0，仅限非商业用途**：任何商业使用都必须自行向
> 权利方（k2-fsa / Next-gen Kaldi）取得授权，Lunac 的 MIT 许可**不**涵盖该权重。
> `espeak-ng-data` 为 GPL-3.0，其对应源码可从上游项目获取。

上述组件的**许可全文与义务清单**位于应用仓库：

- 授权全文目录：[`licenses/`](https://github.com/LythrumMoon/Lunac/tree/main/licenses)
- ZipVoice 组件声明（组件 × 授权 × 来源 × 义务 + 非商用声明）：
  [`licenses/zipvoice/THIRD_PARTY_NOTICES.md`](https://github.com/LythrumMoon/Lunac/blob/main/licenses/zipvoice/THIRD_PARTY_NOTICES.md)
- Live2D Cubism Core 官方许可与可再分发文件清单：
  [`licenses/live2d-cubism-core/`](https://github.com/LythrumMoon/Lunac/tree/main/licenses/live2d-cubism-core)

**下游商用须自行取得授权**：若你把本仓库的插件包或随插件下载的大资产用于商业用途，必须先取得相应
权利方的授权；转分发时还须**连同授权文件一起携带**（ZipVoice 资产内已含 `LICENSES/` 与
`THIRD_PARTY_NOTICES.md`；Live2D 的 `LICENSE.md` / `RedistributableFiles.txt` 随 Core 一起）。

## 五、插件作者须知

- **包命名**：`<id>-<版本>.zip`（`id` 见插件清单；`版本` 形如 `x.y.z`）。zip 内形状 = 一个插件
  目录（`lunac-plugin.json` 在 zip 根，或多一层顶层目录，两种都认）。
- **清单 `lunac-plugin.json` 必填字段**：`id`（同时是目录名）与 `entry`（ESM 入口，缺省 `index.js`）；
  建议写全 `name` / `description` / `keywords` / `icon` / `version` / `homepage`
  （`version` 是市场判「有没有更新」的判据 —— 只改了插件就必须 +1 它）。完整字段与入口契约见
  [Modules 插件开发规范](https://github.com/LythrumMoon/Lunac/blob/main/agent-templates/modules/README.md)。
- **依赖声明 `dependencies`**：插件要用的外部文件（引擎 / 模型）写在这里，宿主安装时一并下载：
  - `{"type":"file","url":"https://…","dest":"bin/x.exe","sha256":"…"}` —— 下载单个文件到 `dest`；
  - `{"type":"archive","url":"https://…","dest":"引擎目录","sha256":"…"}` —— 下载压缩包并解压到 `dest`；
  - `url` **必须 https 且免鉴权直链**；`sha256` 强烈建议写（写了就逐字节校验，不匹配即整包拒装）。
- **大资产不要塞进 git**：几十 MB 以上的二进制 / 模型（Live2D Core、ZipVoice、librespot、llama 模型…）
  走本仓库的 **Release 资产**（公开、免鉴权直链），只在插件的 `dependencies` 里写它的 Release url +
  `sha256`。仓库里只放文本（`index.json`、README）与体积很小的插件 zip。
- **发布链**（在应用仓库里执行，顺序不许跳）：
  `scripts\build-plugins.ps1`（构建 + 打包）→ 验证 → `scripts\publish-plugins.ps1`
  （按分类放进 `packages\`、更新 `index.json`、清理旧版、commit + push）。新插件需先登记分类。
- **分类**：由发布脚本内的 `$Categories` 表（id → 目录名）决定；未登记则落到 `other/`。
