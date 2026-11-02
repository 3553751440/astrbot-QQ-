# QQ 资料修改（astrbot_plugin_qq_profile）

通过聊天指令修改机器人 QQ 的**昵称**和**头像**，并可以查询当前 QQ 号和昵称。

> 仅支持 OneBot v11 协议端：推荐 **NapCat**，也兼容实现了 `set_qq_profile`、`set_qq_avatar` 的 Lagrange、LLOneBot 等。
> QQ 官方机器人平台（QQ 开放平台）无法修改资料，本插件会直接提示不支持。

## 功能

| 指令 | 说明 |
| --- | --- |
| `/改昵称 <新昵称>` | 修改机器人 QQ 昵称 |
| `/改头像` | 同一条消息附带图片，或引用一张图片后发送 |
| `/改头像 <图片链接>` | 用 http(s) 图片链接修改头像 |
| `/我的资料` | 查看机器人 QQ 号、昵称和头像链接 |
| `/改资料帮助` | 显示帮助 |

英文别名：`/setnick`、`/setavatar`、`/myprofile`、`/qqprofilehelp`。

示例：

```text
/改昵称 小助手
/改头像 https://example.com/avatar.png
/改头像            （同时把图片和指令一起发送）
/我的资料
```

## 安装

1. 把整个 `astrbot_plugin_qq_profile` 文件夹放进 AstrBot 的 `data/plugins/` 目录；
2. 在 WebUI「插件」页重载插件，或重启 AstrBot；
3. 在插件配置页按需调整权限和冷却。

目录结构：

```text
data/plugins/astrbot_plugin_qq_profile/
├── main.py
├── metadata.yaml
├── _conf_schema.json
└── README.md
```

无第三方 Python 依赖，不需要 `requirements.txt`。

## 配置

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `admin_only` | bool | `true` | 仅允许 AstrBot 管理员使用 |
| `extra_admins` | list | `[]` | 额外允许使用的 QQ 号（字符串） |
| `cooldown_seconds` | int | `30` | 两次修改之间的冷却秒数，`0` 表示不限制 |
| `avatar_prefer_url` | bool | `true` | 优先把图片链接交给协议端下载，失败再改用 base64 |

管理员身份由 AstrBot 判断（在 WebUI 中配置的管理员账号）。群聊里使用时，需要 @机器人 或使用唤醒前缀。

## 实现说明

- 昵称：调用 OneBot 11 的 `set_qq_profile(nickname=...)`；
- 头像：调用 NapCat 扩展接口 `set_qq_avatar(file=...)`，`file` 支持 URL、本地路径和 base64；
- 图片获取顺序：当前消息中的图片 → 引用消息里的图片 → 通过 `get_msg` 拉取被引用消息；
- 头像修改会先尝试 URL，失败后自动改用 `base64://` 重试；
- 查询资料使用标准接口 `get_login_info`。


## 在 AstrBot 泡泡版（安卓 App）中使用

泡泡版是第三方安卓 App，本质是在手机的 Ubuntu 容器里运行上游 AstrBot + NapCat，内核与官方 AstrBot 4.x 一致（安装时会拉取最新正式版，目前为 v4.29.x），因此本插件可以直接使用。

1. 启动 App，进入内置的 AstrBot WebUI；
2. 打开「插件」→「安装插件」，上传 `astrbot_plugin_qq_profile.zip`；
3. 重载插件，在插件配置里按需调整管理员、冷却等选项；
4. 如果泡泡版开了多个 QQ 账号且各自使用独立实例，每个实例的 `data` 目录是分开的，需要在对应实例里分别安装。

注意：泡泡版升级到 AstrBot 5.x 后，本插件 `metadata.yaml` 中的 `astrbot_version: ">=4.0,<5"` 会阻止加载，需要先适配再放宽版本上限。

## 常见问题

**修改失败 / 协议端报不支持**

确认 AstrBot 使用的是 `aiocqhttp` 适配器，且协议端为 NapCat / Lagrange / LLOneBot 等 OneBot v11 实现。QQ 官方机器人无法修改头像和昵称。

**提示「操作太频繁」**

插件默认有 30 秒全局冷却，可在配置中修改 `cooldown_seconds`。

**修改成功但资料没变**

QQ 资料同步有延迟，通常几秒到几分钟；可稍后刷新资料页确认。

**头像一直失败**

- 图片链接需要协议端能够访问（不能是本地文件路径、需要登录的链接或已过期链接）；
- 建议使用 jpg / png / gif，避免超大图片；
- 如果 AstrBot 与 NapCat 不在同一台机器，优先用图片链接或让插件走 base64（把 `avatar_prefer_url` 设为 `false` 可优先 base64）。

**会不会被封号**

修改资料属于敏感操作，频繁修改可能触发 QQ 风控。请保持低频使用，风险自负。

## 许可

MIT
