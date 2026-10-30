# wechat_robot / 微信 bridge 适配说明

本目录是上游 [`menglimi/astrbot_plugin_private_companion`](https://github.com/menglimi/astrbot_plugin_private_companion)（v7.0.4a）的 **微信 bridge 适配 fork**。

**本 fork 发布仓库：** [`CyL-Cly/astrbot_plugin_private_companion_wechat`](https://github.com/CyL-Cly/astrbot_plugin_private_companion_wechat)

这里的「微信 bridge」指：

```
微信 ↔ wechat-robot-client（服务端+客户端）
         ↕ webhook / HTTP
      AstrBot 平台适配器 `wechat_robot`
         （插件目录 astrbot_plugin_wechat_robot）
         ↕ AstrBot 事件 / 核心回复 / 主动投递
      本插件 astrbot_plugin_private_companion
         ↕ MEMORY_PLUGIN_ID
      astrbot_plugin_memory_companion（微信 fork）
```

**不是** dsh-wechat-bridge（8787）那条旧桥。

## 为什么要改

上游 `metadata.yaml` 与 `SUPPORTED_PLATFORM_NAMES` 只声明 `aiocqhttp` / QQ 官方。未列入的平台会被：

- 拓展页账号发现跳过
- `event_account_id` / `native_user_subject_id` 直接抛错
- 私聊/群路由与主动消息准入拒绝

微信会话 ID 是：

| 场景 | AstrBot UMO | 裸 session_id | Bot 账号键（本 fork） |
|------|-------------|---------------|----------------------|
| 私聊 | `wechat_robot:FriendMessage:wxid_xxx` | `wxid_xxx` | 平台实例 id（如 `wechat_robot`） |
| 群聊 | `wechat_robot:GroupMessage:123@chatroom` | `123@chatroom` | 同上 |

说明：消息里的 `self_id` 可能是 `robot_wxid`，未配置时还会回落成对端 wxid，因此 **不能** 用 `self_id` 当稳定 Bot 账号键；与 QQ 官方一样，统一用 AstrBot 平台配置里的 `id`。

## 本 fork 改动（相对上游 7.0.4a）

1. **metadata**：`support_platforms` 增加 `wechat_robot`；版本 `7.0.4a-wechat.3`；**插件 `name` 仍为 `astrbot_plugin_private_companion`**。
2. **`companion_v2/adapters/astrbot/native.py`**：平台白名单 + wechat；账号投影 / 用户 subject 前缀 `wechat-user:`。
3. **`astrbot_host/binding.py`**：绑定命令读取身份时 wechat 账号 = 平台实例 id。
4. **宿主管理 UI**：平台族显示「微信 · wechat-robot」。
5. **主动消息**：非 `aiocqhttp` 已使用 `AstrBotPlatformSender`（`send_by_session`），微信适配器实现了该方法即可投递文本（图片依赖适配器 Image 组件）。

## 仍不可用 / 弱支持（微信侧）

| 能力 | 原因 |
|------|------|
| QQ 空间 | 依赖 QQ Cookie / OneBot |
| 戳一戳 / 陌生人通知 / 消息撤回监听 | OneBot 专用 |
| 原生气泡分段（native bubbles） | 仅 `aiocqhttp` |
| OneBot 合并转发 / 引用拉取 | 微信无对应接口；群捕获走「非 OneBot」文本骨架 |
| Reality 手机 / 部分游戏原生能力 | 与平台无关或仍偏 QQ 假设，需单独验收 |
| 流式边生成边发 | `wechat_robot` `support_streaming_message=False` |

## 与 memory_companion 接线

- Companion 内 `MEMORY_OWNER_PLUGIN_ID` / `MEMORY_PLUGIN_ID` = `astrbot_plugin_memory_companion`（**未改**）。
- 请部署微信版 memory fork 到同名目录：`/opt/astrbot/data/plugins/astrbot_plugin_memory_companion/`。
- 建议启动顺序：先 Memory，再 Companion；仪表盘两边都启用。
- 两边都按 **UMO**（含 `wechat_robot:…`）隔离记忆与窗口，不会和 QQ 串台。


## 公众号（gh_*）策略

微信 **公众号 / 系统号不是私聊好友**：

1. `wechat_robot` 适配器默认 `ignore_official=true`：`gh_*` 与系统 stub **不进入** AstrBot FriendMessage / LLM 主链 / Companion observe / Memory 私聊窗口。
2. 适配器会把近期公众号推送放进进程内缓冲；LLM 仅可通过工具查询：
   - `wechat_official_list` — 列出近期出现过的公众号
   - `wechat_official_lookup` — 按 `gh_wxid` / 关键词查正文
3. Companion 绑定、会话目录、主动私信都会拒绝 `gh_*` 作为 participant。
4. 需要公众号内容时，在正常私聊里让模型调用上述工具，而不是把公众号当成陪伴对象。

## 安装到 AstrBot（home）

1. 已启用 `astrbot_plugin_wechat_robot`，webhook 指向 AstrBot（如 `http://astrbot:6196/wechat-robot/callback`）。
2. **强烈建议**在适配器配置填写 `robot_wxid`（机器人自己的微信号 id）。
3. 部署本目录为：

   ```text
   /opt/astrbot/data/plugins/astrbot_plugin_private_companion/
   ```

   目录名必须与插件 `name` 一致，不要用 `..._wechat` 当 AstrBot 插件目录名。
4. 确认 wechat 事件把 HTTP 客户端挂到 `event.bot`（与 `platform.get_client()` 同一对象），否则 Companion 会报 `host_client_mismatch`。本环境的 wechat 适配器已补 `self.bot = client`。
5. 关掉 wechat-robot **客户端内置 AI**，否则双回。
6. `docker restart astrbot` 后在仪表盘启用「我会永远陪着你」，按「陪伴小屋 → 开始使用」绑定 **wechat_robot** 平台实例与私聊窗口。

## 自检

```bash
python3 -c "
from companion_v2.adapters.astrbot.native import (
    supported_platform_name, wechat_platform_name, native_user_subject_id, route_id,
)
assert supported_platform_name('wechat_robot')
assert wechat_platform_name('wechat_robot')
assert native_user_subject_id('wechat_robot', 'wxid_ab') == 'wechat-user:wxid_ab'
assert route_id('123@chatroom') == '123@chatroom'
print('ok')
"
```

私聊发一句非命令文本后，AstrBot 日志应出现 Companion / Memory 相关记录；拓展页宿主管理应能看到「微信 · wechat-robot」。
