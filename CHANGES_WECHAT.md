# Changes vs menglimi/astrbot_plugin_private_companion@7.0.4a

| File | Change |
|------|--------|
| `metadata.yaml` | +`wechat_robot`; version `7.0.4a-wechat.1`; keep name `astrbot_plugin_private_companion` |
| `companion_v2/adapters/astrbot/native.py` | `SUPPORTED_PLATFORM_NAMES` + wechat; `wechat_platform_name`; `event_account_id` / `native_user_subject_id` |
| `astrbot_host/session_topology.py` | comment: wechat uses platform instance id as account key |
| `astrbot_host/binding.py` | wechat binding account → platform instance id |
| `pages/宿主管理/app.js` | platform label「微信 · wechat-robot」 |
| `pages/关系与权限/*.js` | wechat 时显示「微信」而非硬编码 QQ |
| `WECHAT_BRIDGE.md` | Chinese install / protocol notes |
| `README.md` | fork banner |

Not changed: QZone / OneBot poke / stranger notice / recall / native bubbles（仍 QQ/OneBot 专属）。
主动投递非 aiocqhttp 已走 `AstrBotPlatformSender` → `Platform.send_by_session`，微信无需另写发送器。
记忆桥 `MEMORY_PLUGIN_ID=astrbot_plugin_memory_companion` 未改。

## 7.0.4a-wechat.4 (2026-10-30)
- H5: admin conversation_scheduler soft-skip override excludes `confidence_skip` (only `insufficient_context` / `no_natural_opening` / `other`).
