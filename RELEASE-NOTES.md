# dsh-raganything-kb 0.3.17-sharedkb

分支：8.6 多用户网关线（sharedkb），基线 = 0.3.16-sharedkb。

## 与基线差异（仅四处，其余成员逐字节不变）

| 成员 | 变更 |
| --- | --- |
| `lib/client.js` | 回灌 2026-09-22 以来全部 client 侧线上热修（410089 B -> 415617 B）：① **0.1.7 图标兼容**（`kbIcons071`：`IconX14/16` 尺寸后缀 -> `IconXRegular` 字重后缀，随 DSH 0.1.7-alpha.2 升级落地的适配，缺了即 React #130 整条 UI 消失）；② **KB 面板与 DSH 导航栏联动**：`closeKbPanel`（Esc/关闭 = 回主页并展开侧栏）、`softCloseKb`（点其他面板即轻量收起覆盖层）、主面板切换监视（`usePanelInfo`/`activePanelId`）、导航栏捕获阶段点击兜底；③ **侧栏跟随**：打开期间跟踪 DSH 侧栏列宽发布为 `--dsh-kb-left`，fixed 覆盖层始终位于导航栏右侧；④ 工件栏/模型面板等 CSS 增量 |
| `lib/index.js` | **工具 schema 修复**（对齐 `tools/patch_kb_tool_schema86.py`）：`rag_query` / `rag_ingest` / `rag_note` / `rag_delete` 等工具把属性对象内部的 `required: true` 移到 `parameters` 顶层 `required` 数组——旧写法会被 OpenAI/Gemini 严格端点判 400（`Invalid schema for function ...: True is not of type 'array'`） |
| `package.json` | 版本 0.3.16 -> 0.3.17 |
| `RELEASE-NOTES.md` | 替换为本文件 |

## 不变项（显式断言）

- `sidecar/server.py` 与 0.3.16 逐字节相同（`cc60b669…`，383404 B）—— 本版未动 server 侧。
- 其余成员（cordis.patch.yml / kb/config.json.template / sidecar/viewer/ 等）逐字节不变。

## 为什么要出这一版

0.3.16 截止于 2026-09-22；此后 8.6 上又落了两类只存在于活副本的热修：
① 2026-09-23 DSH 升级 0.1.7-alpha.2 后的图标字重后缀适配（漏了即知识库面板整条消失）；
② 2026-09-24 的 KB 工具 schema 修复（严格 OpenAI 兼容端点 400 的真根因）。
任何 `pnpm install` / 重装 / 回滚都会把这些修复整体抹掉 —— 本版把它们固化进发行件。

## 验证

- 包内 `lib/client.js` / `lib/index.js` 与 8.6 宿主活副本逐字节一致（LF-clean，无 CRLF）。
- 修复标记齐备（见 build_rel0317.py 断言 5）。
- `node --check` 通过；无 `.bak` 混入；确定性 md5 复核通过（连跑两次同 md5）。

## 部署注意

换用本包需同步改 12 份 profile 的依赖（宿主 + 11 用户 home）并重算 pnpm-lock integrity，
否则重装仍会按旧 tgz 还原。宿主重启 `systemctl restart raganything-sidecar`（独立服务）；
用户实例用 `POST :3200N/models/restart` 或整服务重启 `dsh-web`。
