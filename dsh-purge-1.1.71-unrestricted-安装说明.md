# dsh-purge 1.1.71 + 无限制模式（移植版安装包）

## 这是什么

按 2026-11-15 移植记录，把 community 1.6.88 的「无限制模式」移植进 dsh-purge 1.1.71 后重新打的包。原移植成品已被 community 1.7.x 覆盖，本包于 2026-11-18 依据备份重建。

| 项 | 值 |
|---|---|
| 包名 | `dsh-purge` |
| 版本 | `1.1.71` |
| 包文件 | `D:\DeepSeek-Harness\apps\dsh-purge-1.1.71-unrestricted.tgz`（1.4 MB） |
| SHA256 | `b97906f87ed5a3f7a63997c7d3986f3bf277e5ff4ef7fd1c0325da3486c860bf` |
| 解包源目录 | `D:\DeepSeek-Harness\apps\dsh-purge-1.1.71-unrestricted\` |
| 基底 | 11-15 备份 `plugin-dsh-purge-1.1.71`（移植前原始包） |
| 无限制模式来源 | community 1.6.88 备份（`dsh-purge-community/backups/2026-11-11T09-38-12-239Z-dfc7de1f/`） |

## 含移植内容

6 个文件 SHA-256 与 community 1.6.88 源逐项一致：

- `lib/presets-plugin.js`（3381 B）— 注册节点，等 registry 就绪后 `agentPresets.register()`
- `lib/presets.js`（3121 B）— preset 定位与落盘
- `lib/preset-compat.js`（11185 B）— 子预设迁移
- `lib/remove-tree.js`（2955 B）— 兼容层依赖
- `presets/unrestricted/preset.yml`（252 B）— 名称「无限制模式」、order 20
- `presets/unrestricted/agent.cordis.yml`（15190 B）— 预设定义（与移植记录记载大小吻合）

配置改动两处：

- `package.json` → `exports["./presets-plugin"] = "./lib/presets-plugin.js"`
- `cordis.patch.yml` → insert 组新增 `- id: dsh-purge-presets` / `name: dsh-purge/presets-plugin`

保留 1.1.71 自带 `lib/core.js`（337874 B）与 `lib/redteam/preset-composition.js`，未用 community 版替换（按移植记录）。

## 安装（不覆盖现有 web profile）

当前 web profile 用的是 `dsh-purge-community` 1.7.14 + `ds-bot`，**不要原地替换**。两种方式：

**方式 A — 新建独立 profile（推荐，零风险）**

```
dsh profile create unrestricted --from-default-profile web
dsh add file:D:/DeepSeek-Harness/apps/dsh-purge-1.1.71-unrestricted.tgz --profile unrestricted
```

**方式 B — 在目标 profile 的 package.json 加依赖**

```json
"dependencies": {
  "dsh-purge": "file:D:/DeepSeek-Harness/apps/dsh-purge-1.1.71-unrestricted.tgz"
}
```

若用 file: 指向解包目录而非 tgz，把路径换成 `D:/DeepSeek-Harness/apps/dsh-purge-1.1.71-unrestricted`。

**卸载**：`dsh remove dsh-purge`（只影响装了本包的 profile）。

## 验证是否生效

启动后调 RPC 看预设是否注册（`unrestricted` order 20，无 `broken` 字段）：

```
curl -s -b <cookie> -X POST http://127.0.0.1:<端口>/api/agentPresets/list \
  -H "Content-Type: application/json" \
  -d '{"type":"client-request","rpcId":"r1","method":"agentPresets/list","payload":{"args":{}}}'
```

鉴权：先 `?token=<启动日志 token>` 带 cookie jar 访问一次拿 `dsh-auth-*` cookie。详见 `[[40-归档/2026-11-15-DSH-SuperAgent/移植记录_Web端无限制模式_2026-11-15]]` 第三节。

## 边界

- 包名沿用 `dsh-purge`，与 community 版（`dsh-purge-community`）不同名，可共存；当前 web profile 无同名包。
- `preset-compat.js` 内迁移路径硬编码 `dsh-purge-community/preset-migrations`（源包原样保留），只影响历史配置迁移，不影响无限制模式功能。
- 本包只含插件代码，不含任何 token / 密钥 / cookie（已全包扫描确认）。
- 未做任何 profile 改动，web 端 1.7.14 保持原样。

## 来源

- 基底：`D:\DeepSeek-Harness\apps\.backup\20261115-144200-web-unrestricted\plugin-dsh-purge-1.1.71\`
- 移植文件：`D:\DeepSeek-Harness\dsh-home\dsh-purge-community\backups\2026-11-11T09-38-12-239Z-dfc7de1f\plugins\dsh-purge-community\`
- 操作依据：`[[40-归档/2026-11-15-DSH-SuperAgent/移植记录_Web端无限制模式_2026-11-15]]`
