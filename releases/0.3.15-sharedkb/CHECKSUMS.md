# CHECKSUMS — dsh-raganything-kb 0.3.15-sharedkb

分支：8.6 多用户网关线（sharedkb）。基线：`0.3.14-sharedkb`。

## 发布件

| 文件 | md5 | sha256 | size (B) |
| --- | --- | --- | --- |
| `dsh-raganything-kb-0.3.15-sharedkb-portable.tar.gz` | `3fd9a3fda88fa0361c5235dd0949d16d` | `93c72f3636278334fe1034c1323aed7828a4f0c779973879dd991941ead8f974` | 10725530 |
| `dsh-raganything-kb-0.3.15-sharedkb.tgz` | `5087278c121b9ca1d05bc31e6aa587cb` | `3f34456b58b01af4d92995ff89fa06b8a6a0825db1fa49f235ffaae399783549` | 5346221 |

## 构建口径

- **基线**：`releases/0.3.14-sharedkb/dsh-raganything-kb-0.3.14-sharedkb.tgz`
  （md5 `60b23c2f4354a60166160b97c1965ee0`，195 成员）
- **确定性**：GNU_FORMAT tar + 固定 mtime/uid/gid/uname/gname/mode + 成员按名排序 +
  gzip `filename=""` `compresslevel=9` `mtime=0`；连打两次 md5 相同。
  ★ 便携包的 `portable_build` 时间戳已改为 `--build-stamp` 冻结参数（默认
  `2026-09-20T18:30:00+0800`）——旧构建器用 `now()`，导致便携包**不可复现**
  （实测两次 `a2a8e507…` vs `00e61567…`）。
- **打包器**：`build/build_rel0315.py`（发行 tgz）/ `build/build_portable_0315.py`（便携包）
- **载荷来源**：8.6 宿主活副本
  `/home/lgsj/.dsh/profiles/web/node_modules/dsh-raganything-kb/`
  · `server.py` md5 `75559557795d26e881fb218ffd53fc02`（375559 B）
  · `lib/client.js` md5 `526ed56d7d44daeae0fcefb40eebb371`（410089 B）
  ★ 宿主 + 11 份用户副本 **12 份 server.py md5 全一致** ⇒ 0.3.14 的 host-aware 合成
  已在活副本上收敛为单一权威版本，本版无需再合成。

## 与基线的差异（全部）

| 成员 | 变更 |
| --- | --- |
| `package/sidecar/server.py` | 替换为 9/19–9/20 全量 server 侧热修版（相对基线 +839/-26 行） |
| `package/lib/client.js` | 替换为同期前端热修版 |
| `package/package.json` | 版本 `0.3.14` -> `0.3.15` |
| `package/RELEASE-NOTES.md` | 替换为 0.3.15 说明 |

其余 191 个成员（含 `lib/index.js`、`cordis.patch.yml`、`kb/config.json.template`、
README、`sidecar/viewer/` 187 项资源）**逐字节不变**。

## 断言（打包器内置，全部通过）

1. 基线版本 == `0.3.14`
2. 其余成员逐字节不变（gone / added / changed 三集合为空，有意替换 4 项）
3. 产物内无 `.bak`
4. 修复标记齐备：server 8 项（`_ainsert_dup_safe` / `kb-sessions.json` /
   `_route_scope_query` / `_shared_is_local` / `_VROOT_PREFIXES` / `_silent_dup` /
   `semantic_title` / `_find_doc_ids_by_display`）、client 6 项
   （`buildSharedTree` / `flattenSharedRows` / `activeSessId` / `executeCopyTo` /
   `sharedVisibleRows` / `kbFeat86-shareddocid`）
5. `py_compile`（server）+ `node --check`（client）通过；无 CRLF 泄漏
6. 确定性 md5 复核通过
7. 注入的 `server.py` / `client.js` 与 8.6 活副本 md5 一致

## 便携包额外断言

- `payload/sidecar/viewer/` 187 个普通文件在包内（缺则 `/viewer/*` 404）
- `install.sh` / `uninstall.sh` / `verify.sh` 语法检查按**退出码**判定，全部 ok
- `manifest.json` 记录 204 条目；保留 `changes_0.3.13/0.3.14`，新增 `changes_0.3.15`
- `plugin/dsh-raganything-kb-0.3.15.tgz` 与发行 tgz 逐字节相同

## 本版内容（相对 0.3.14 新增）

**server.py**
- 重试解析状态转正：`_silent_dup` 自愈（收尾事实校验，无 processed/processing 切片则
  删 `dup-` 脏行 + 加出处行重插）、`_find_doc_ids_by_display` 双侧 vroot 归一
  （共享库 doc 找不到导致自愈早退）、孪生一起入队（同词干 FAILED 全入队）
- 共享库覆盖保目录树与落点；回收箱滚动 / 全部还原 / 全部删除
- 对话产物跨会话账本（`kb-sessions.json`）加固
- 产物语义化命名（`semantic_title`）
- `/index/reingest` 入队化（去全局锁，立即返回 `started:true`）

**lib/client.js**
- 回收箱滚动 + 整树顶层
- 产物语义化命名触发
- 共享目录树 / 产物栏会话过滤细化

## 部署状态（重要）

- 本版 `server.py` 为**单一权威版**（宿主与用户实例同一份，12 份 md5 已一致）。
  宿主 `_shared_is_local()` 为 True（PORT==SHARED_BASE 端口）-> 本地检索；
  用户实例为 False（3200N != 17321）-> 转发宿主 sidecar。
- ⚠️ **8.6 的 12 份 profile 依赖当前仍指向 0.3.14**：本包已出，但**尚未执行依赖切换**。
  在切换前，线上运行的仍是活副本热修代码；任何 `pnpm install` / 重装 / 回滚
  都会按 0.3.14 还原，**抹掉本版全部修复**。需另跑 profile 依赖同步（改各 profile
  `package.json` 依赖 + 已装包 `package.json`，勿跑 `pnpm install`）。
- 100.100.6.55 与本机 DSH Desktop 未切换。

## 与 2026-09-20「共享 ZeroClaw @ 范围检索」修复的关系

该问题是**数据层**问题（LightRAG 内容哈希去重 → 目标 `file_path` 0 切片），
修法是离线把已有切片克隆到精确 `file_path`（脚本 `tools/_zc_clone.py`），
**不含任何 payload 代码变更**，故本版未体现该修复——属运维操作，不进发行包。
