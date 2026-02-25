# 飞书权限参考

## 权限分类

### 消息 (im)

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `im:message` | 读取和发送消息 | 核心消息功能 |
| `im:message:send_as_bot` | 以机器人身份发送消息 | 发送任何消息 |
| `im:message.group_at_msg` | 读取群内 @机器人 消息 | 接收 @提及 |
| `im:message.group_at_msg:readonly` | 读取群内 @机器人 消息（只读） | 接收 @提及 |
| `im:message.p2p_msg` | 读取单聊消息 | 一对一机器人对话 |
| `im:message.p2p_msg:readonly` | 读取单聊消息（只读） | 一对一机器人对话 |
| `im:resource` | 访问消息资源 | 上传/下载图片和文件 |

### 群聊管理 (im:chat)

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `im:chat` | 管理群聊 | 创建/更新/解散群聊 |
| `im:chat:readonly` | 读取群信息 | 列出机器人所在群、获取群信息 |
| `im:chat.member` | 管理群成员 | 添加/移除群成员 |
| `im:chat.member:readonly` | 读取群成员 | 列出群成员 |

### 通讯录 (contact)

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `contact:user.id:readonly` | 读取用户 ID | 解析用户标识符 |
| `contact:user.base:readonly` | 读取用户基本信息 | 获取用户名称、头像 |
| `contact:user.email:readonly` | 读取用户邮箱 | 通过邮箱匹配用户 |
| `contact:user.phone:readonly` | 读取用户电话 | 通过电话匹配用户 |
| `contact:department.base:readonly` | 读取部门信息 | 基于部门的操作 |

### 日历

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `calendar:calendar` | 管理日历 | 创建/编辑日历事件 |
| `calendar:calendar:readonly` | 读取日历 | 查询日历事件 |

### 文档

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `docs:doc` | 管理文档 | 创建/编辑文档 |
| `docs:doc:readonly` | 读取文档 | 读取文档内容 |
| `sheets:spreadsheet` | 管理电子表格 | 修改表格数据 |
| `wiki:wiki:readonly` | 读取知识库 | 访问知识库 |

### 多维表格

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `bitable:app` | 管理多维表格 | 多维表格完整 CRUD |
| `bitable:app:readonly` | 读取多维表格 | 查询多维表格数据 |

### 审批

| 权限范围 | 说明 | 用途 |
|----------|------|------|
| `approval:approval` | 管理审批 | 创建/管理审批流程 |
| `approval:approval:readonly` | 读取审批 | 查询审批状态 |

---

## 常用权限组合

### 最小机器人（接收并回复）
```
im:message:send_as_bot
im:message.group_at_msg:readonly
im:message.p2p_msg:readonly
```

### 通知机器人（向群发送）
```
im:message:send_as_bot
im:chat:readonly
```

### 交互式机器人（消息 + 卡片 + 文件）
```
im:message
im:message:send_as_bot
im:message.group_at_msg:readonly
im:message.p2p_msg:readonly
im:resource
im:chat:readonly
```

### 群管理机器人
```
im:message:send_as_bot
im:chat
im:chat.member
contact:user.id:readonly
contact:user.base:readonly
```

### 全功能机器人
```
im:message
im:message:send_as_bot
im:message.group_at_msg:readonly
im:message.p2p_msg:readonly
im:resource
im:chat
im:chat:readonly
im:chat.member
im:chat.member:readonly
contact:user.id:readonly
contact:user.base:readonly
```

---

## 事件订阅

### 机器人常用事件

| 事件 | event_type | 说明 |
|------|-----------|------|
| 接收消息 | `im.message.receive_v1` | 机器人收到消息（群内 @提及或单聊） |
| 消息被撤回 | `im.message.recalled_v1` | 消息被撤回 |
| 消息已读 | `im.message.message_read_v1` | 消息已读回执 |
| 机器人入群 | `im.chat.member.bot.added_v1` | 机器人被添加到群 |
| 机器人出群 | `im.chat.member.bot.deleted_v1` | 机器人被移出群 |
| 用户入群 | `im.chat.member.user.added_v1` | 用户加入了机器人所在的群 |
| 用户出群 | `im.chat.member.user.deleted_v1` | 用户离开了机器人所在的群 |
| 群解散 | `im.chat.disbanded_v1` | 群聊被解散 |
| 群信息更新 | `im.chat.updated_v1` | 群名称/描述等变更 |
| 用户进入单聊 | `im.chat.access_event.bot_p2p_chat_entered_v1` | 用户打开与机器人的单聊 |
| 卡片交互 | `card.action.trigger` | 用户点击了卡片中的按钮/操作 |
| 表情回应 | `im.message.reaction.created_v1` | 用户对消息添加了表情回应 |

### 事件格式 (v2.0)

```json
{
    "schema": "2.0",
    "header": {
        "event_id": "unique_event_id",
        "event_type": "im.message.receive_v1",
        "create_time": "1234567890",
        "token": "verification_token",
        "app_id": "cli_xxx",
        "tenant_key": "tenant_key"
    },
    "event": {
        "sender": {
            "sender_id": {
                "open_id": "ou_xxx",
                "user_id": "xxx",
                "union_id": "on_xxx"
            },
            "sender_type": "user",
            "tenant_key": "xxx"
        },
        "message": {
            "message_id": "om_xxx",
            "root_id": "",
            "parent_id": "",
            "create_time": "1234567890",
            "chat_id": "oc_xxx",
            "chat_type": "group",
            "message_type": "text",
            "content": "{\"text\":\"@_user_1 hello\"}"
        }
    }
}
```

### URL 验证

首次配置事件订阅 URL 时，飞书会发送一个验证请求：

```json
{
    "challenge": "ajls384kdjx98XX",
    "token": "verification_token",
    "type": "url_verification"
}
```

应答：
```json
{"challenge": "ajls384kdjx98XX"}
```
