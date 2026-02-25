# 飞书交互式卡片参考

交互式卡片是飞书中最强大的消息格式。支持按钮、选择器、日期选择和动态内容更新。

## 卡片 JSON 结构

```json
{
    "config": {
        "wide_screen_mode": true,
        "enable_forward": true
    },
    "header": {
        "title": {"tag": "plain_text", "content": "Card Title"},
        "template": "blue"
    },
    "elements": []
}
```

### 头部模板（颜色）

`blue` | `wathet` | `turquoise` | `green` | `yellow` | `orange` | `red` | `carmine` | `violet` | `purple` | `indigo` | `grey`

---

## 元素类型

### 文本块 (div)

```json
{
    "tag": "div",
    "text": {"tag": "lark_md", "content": "**Bold** and `code` and [link](https://example.com)"},
    "extra": {
        "tag": "button",
        "text": {"tag": "plain_text", "content": "Click Me"},
        "type": "primary",
        "value": {"key": "value"}
    }
}
```

文本标签：`plain_text`（无格式）| `lark_md`（支持 Markdown 子集）

Lark_md 支持的语法：
- `**加粗**`、`*斜体*`、`~~删除线~~`
- `` `行内代码` ``
- `[链接文本](url)`
- `<at id=ou_xxx>名称</at>` — 提及用户
- Emoji 缩写：`:smile:`、`:thumbsup:`

### 分割线

```json
{"tag": "hr"}
```

### 图片

```json
{
    "tag": "img",
    "img_key": "img_xxx",
    "alt": {"tag": "plain_text", "content": "description"},
    "title": {"tag": "plain_text", "content": "Image Title"},
    "mode": "fit_horizontal",
    "preview": true
}
```

模式：`crop_center` | `fit_horizontal` | `large` | `medium` | `small` | `tiny`

### 备注（底部）

```json
{
    "tag": "note",
    "elements": [
        {"tag": "plain_text", "content": "Footer text"},
        {"tag": "img", "img_key": "img_xxx", "alt": {"tag": "plain_text", "content": "icon"}}
    ]
}
```

### 多列布局 (Column Set)

```json
{
    "tag": "column_set",
    "flex_mode": "bisect",
    "background_style": "default",
    "columns": [
        {
            "tag": "column",
            "width": "weighted",
            "weight": 1,
            "elements": [
                {"tag": "div", "text": {"tag": "lark_md", "content": "左侧列"}}
            ]
        },
        {
            "tag": "column",
            "width": "weighted",
            "weight": 1,
            "elements": [
                {"tag": "div", "text": {"tag": "lark_md", "content": "右侧列"}}
            ]
        }
    ]
}
```

flex_mode: `none` | `stretch` | `flow` | `bisect` | `trisect`

---

## 交互元素

所有交互元素放在 `action` 块内：

```json
{
    "tag": "action",
    "actions": [/* interactive elements */]
}
```

### 按钮

```json
{
    "tag": "button",
    "text": {"tag": "plain_text", "content": "提交"},
    "type": "primary",
    "value": {"action": "submit", "data": "custom_payload"},
    "confirm": {
        "title": {"tag": "plain_text", "content": "确认？"},
        "text": {"tag": "plain_text", "content": "确定要执行此操作吗？"}
    }
}
```

类型：`default` | `primary` | `danger`

### 下拉选择

```json
{
    "tag": "select_static",
    "placeholder": {"tag": "plain_text", "content": "请选择"},
    "value": {"key": "select_1"},
    "options": [
        {"text": {"tag": "plain_text", "content": "选项 A"}, "value": "a"},
        {"text": {"tag": "plain_text", "content": "选项 B"}, "value": "b"}
    ]
}
```

选择类型：`select_static` | `select_person`（人员选择）| `multi_select_static` | `multi_select_person`

### 溢出菜单（更多操作）

```json
{
    "tag": "overflow",
    "options": [
        {"text": {"tag": "plain_text", "content": "编辑"}, "value": "edit"},
        {"text": {"tag": "plain_text", "content": "删除"}, "value": "delete"}
    ],
    "value": {"key": "overflow_1"}
}
```

### 日期选择器

```json
{
    "tag": "date_picker",
    "placeholder": {"tag": "plain_text", "content": "选择日期"},
    "value": {"key": "date_1"},
    "initial_date": "2024-01-01"
}
```

另有：`picker_time` | `picker_datetime`

### 输入框（文本字段）

```json
{
    "tag": "input",
    "name": "input_field",
    "placeholder": {"tag": "plain_text", "content": "请输入..."},
    "max_length": 200,
    "label": {"tag": "plain_text", "content": "名称"}
}
```

---

## 卡片回调处理

### 配置步骤

1. 开发者后台 → 应用功能 → 机器人 → 消息卡片请求网址
2. 设置为：`https://your-domain.com/webhook/card`

### 回调负载

```json
{
    "open_id": "ou_xxx",
    "user_id": "xxx",
    "open_message_id": "om_xxx",
    "open_chat_id": "oc_xxx",
    "tenant_key": "xxx",
    "token": "verification_token",
    "action": {
        "value": {"action": "approve", "id": "123"},
        "tag": "button",
        "option": "",
        "timezone": ""
    }
}
```

### 响应选项

**返回空** `{}` → 不更新卡片

**返回新的卡片 JSON** → 替换整个卡片内容

> 以下为片段示例，实际项目中需补充必要导入（如 `request` / `jsonify` / `datetime`）与应用初始化代码。
> 生产环境请先完成回调验签/验 token（或直接使用官方 SDK 事件处理器）后再处理业务逻辑。

```python
@app.route("/webhook/card", methods=["POST"])
def handle_card():
    data = request.get_json()
    action = data.get("action", {})
    value = action.get("value", {})
    user_id = data.get("open_id")

    if value.get("action") == "approve":
        return jsonify({
            "config": {"wide_screen_mode": True},
            "header": {
                "title": {"tag": "plain_text", "content": "✅ 请求已审批"},
                "template": "green"
            },
            "elements": [{
                "tag": "div",
                "text": {"tag": "lark_md", "content": f"由 <at id={user_id}>用户</at> 于 {datetime.now().strftime('%Y-%m-%d %H:%M')} 审批"}
            }]
        })

    return jsonify({})
```

---

## 通过 API 更新卡片

更新已发送的卡片消息：

```python
def update_card(client: FeishuClient, message_id: str, card: dict):
    """更新已发送的交互式卡片。"""
    return client.request("PATCH", f"/im/v1/messages/{message_id}", json={
        "msg_type": "interactive",
        "content": json.dumps(card),
    })
```

---

## 常用卡片模式

### 审批卡片

```python
def build_approval_card(title: str, content: str, approval_id: str) -> dict:
    return {
        "config": {"wide_screen_mode": True},
        "header": {"title": {"tag": "plain_text", "content": f"🔔 {title}"}, "template": "orange"},
        "elements": [
            {"tag": "div", "text": {"tag": "lark_md", "content": content}},
            {"tag": "hr"},
            {"tag": "action", "actions": [
                {"tag": "button", "text": {"tag": "plain_text", "content": "✅ Approve"}, "type": "primary", "value": {"action": "approve", "id": approval_id}},
                {"tag": "button", "text": {"tag": "plain_text", "content": "❌ Reject"}, "type": "danger", "value": {"action": "reject", "id": approval_id}},
            ]}
        ]
    }
```

### 状态卡片

```python
def build_status_card(service: str, status: str, details: str) -> dict:
    color = "green" if status == "ok" else "red"
    icon = "✅" if status == "ok" else "🔴"
    return {
        "config": {"wide_screen_mode": True},
        "header": {"title": {"tag": "plain_text", "content": f"{icon} {service}"}, "template": color},
        "elements": [
            {"tag": "div", "text": {"tag": "lark_md", "content": details}},
            {"tag": "note", "elements": [{"tag": "plain_text", "content": f"Updated: {datetime.now().strftime('%H:%M:%S')}"}]}
        ]
    }
```

### 进度卡片

```python
def build_progress_card(task: str, current: int, total: int) -> dict:
    pct = int(current / total * 100)
    bar = "█" * (pct // 5) + "░" * (20 - pct // 5)
    return {
        "config": {"wide_screen_mode": True},
        "header": {"title": {"tag": "plain_text", "content": f"📊 {task}"}, "template": "blue"},
        "elements": [
            {"tag": "div", "text": {"tag": "lark_md", "content": f"`{bar}` **{pct}%**\n{current}/{total} completed"}},
        ]
    }
```

---

## 卡片搭建工具

使用官方卡片搭建工具进行可视化设计并导出 JSON：
- 飞书（新版）：https://open.feishu.cn/cardkit
- 飞书（旧版）：https://open.feishu.cn/tool/cardbuilder
- Lark：https://open.larksuite.com/tool/cardbuilder
