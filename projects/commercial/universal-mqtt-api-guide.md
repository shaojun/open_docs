# VisionTalk MQTT 设备端消息订阅文档

## 概述

VisionTalk MQTT服务为设备端提供统一的消息下发机制，支持多种消息类型和控制指令。设备端通过订阅相应的MQTT主题来接收服务端下发的消息。

## MQTT连接配置

### 基本连接参数
- **Broker地址**: 根据部署环境配置
- **端口**: 1883 (非SSL) / 8883 (SSL)
- **QoS级别**: 1 (至少一次传递)
- **Keep Alive**: 60秒

### 认证信息
根据服务端配置提供用户名和密码

## 主题订阅规则

### 1. 设备消息主题

#### 基本消息订阅

##### 设备级主题
```
{device_level}/app/{app_id}/{type}/+  # 所有消息类型
```
##### 设备上App级主题
```
{app_level}/{type}/+  # 所有消息类型
```

##### 系统级主题
```
rpc/visitpark-uniapp/system
```

#### 具体消息类型订阅
```
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/request/dialog          # 对话消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/request/notification    # 通知消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/request/input           # 输入消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/request/list_selection  # 列表选择消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/request/context_action  # 上下文操作消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/event/{event_type}      # 事件消息
rpc/visitpark-uniapp/device/{serial_no}/app/{app_id}/control/{control_type}  # 控制消息
```

#### 广播消息主题
```
rpc/visitpark-uniapp/broadcast                           # 系统广播消息
```

## 消息格式

所有消息都采用统一的JSON格式，基于 `UnifiedMessageDto` 结构定义：

### 基本消息结构

```json
{
  "message_id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "dialog",
    "params": {
      "timeout": 30000,
      "priority": "high",
      "modal": true
    },
    "data": {
      "title": "确认操作",
      "message": "您确定要执行此操作吗？",
      "buttons": ["确认", "取消"]
    },
    "status": null,
    "error": null
  },
  "metadata": {
    "source": "api_server",
    "priority": "normal",
    "device_type": "tablet"
  }
}
```

### 消息类型枚举 (type)

| 类型 | 描述 | 使用场景 |
|------|------|----------|
| `request` | 客户端请求消息 | 服务端向设备端发送操作请求 |
| `event` | 事件通知消息 | 状态变更、系统事件通知 |
| `control` | 控制指令推送 | 设备控制指令 |

### 载荷结构 (payload)

#### action 字段
统一的动作/指令类型，定义了具体要执行的操作：

- `dialog` - 显示对话框
- `list_selection` - 列表选择
- `input` - 请求输入
- `context_action` - 上下文操作
- `notification` - 显示通知
- `device_volume` - 设备音量控制
- `device_mute` - 静音控制
- `device_restart` - 重启设备
- 
#### params 字段
动作参数，控制行为的配置参数，不同action类型有不同的参数结构。

#### data 字段
内容数据，包含文本、图片、音视频等实际内容。


## 具体消息类型示例

### 1. 对话框消息 (DialogMessageDto)

```json
{
  "message_id": "dialog_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "dialog",
    "params": {
      "timeout": 30000,
      "modal": true,
      "closable": true,
    },
    "data": {
      "title": "系统提示",
      "message": "检测到新的软件更新，是否立即更新？",
      "dialog_type": "info",
      "buttons": [
        {
          "text": "立即更新",
          "value": "update_now",
          "style": "primary",
          "button_type": "action",
          "is_primary": true
        },
        {
          "text": "稍后提醒",
          "value": "remind_later",
          "style": "secondary",
          "button_type": "action",
          "is_primary": false
        }
      ]
    }
  }
}
```

### 2. 通知消息 (NotificationMessageDto)

```json
{
  "message_id": "notification_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "payload": {
    "action": "notification",
    "params": {
      "notification_type": "info",
      "duration": 5000,
      "position": "top-right",
      "auto_close": true
    },
    "data": {
      "title": "新消息",
      "message": "您收到一条新的系统通知",
      "type": "info",
      "icon": "bell",
      "actions": [
        {
          "text": "查看详情",
          "action": "view_details"
        }
      ]
    }
  },
  "metadata": {
    "source": "notification_service"
  }
}
```

### 3. 输入消息 (InputMessageDto)

```json
{
  "message_id": "input_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "payload": {
    "action": "input",
    "params": {
      "timeout": 60000,
      "required": true,
      "validation": "text"
    },
    "data": {
      "title": "请输入信息",
      "Description": "请输入您的姓名：",
      "placeholder": "请输入姓名",
      "input_type": "text",
      "max_length": 50
    }
  },
  "metadata": {
    "source": "form_service"
  }
}
```

### 4. 列表选择消息 (ListSelectionMessageDto)

```json
{
  "message_id": "list_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "Request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "list_selection",
    "params": {
      "timeout": 45000,
      "priority": "normal",
      "require_response": true,
      "selection_mode": "Single"
    },
    "data": {
      "title": "请选择服务类型",
      "description": "请从以下选项中选择您需要的服务：",
      "placeholder": "请选择一个选项...",
      "items": [
        {
          "id": "service_1",
          "text": "咨询服务",
          "description": "提供专业咨询服务",
          "value": "consultation",
          "icon": "question-circle",
          "disabled": false,
          "metadata": {
            "department": "customer_service",
            "available_time": "09:00-18:00"
          }
        },
        {
          "id": "service_2",
          "text": "技术支持",
          "description": "技术问题解答和故障排除",
          "value": "technical_support",
          "icon": "tools",
          "disabled": false,
          "metadata": {
            "department": "technical_dept",
            "available_time": "24/7"
          }
        },
        {
          "id": "service_3",
          "text": "业务办理",
          "description": "各类业务受理和办理",
          "value": "business_process",
          "icon": "file-text",
          "disabled": false,
          "metadata": {
            "department": "business_dept",
            "available_time": "09:00-17:00"
          }
        },
        {
          "id": "service_4",
          "text": "VIP服务",
          "description": "专属VIP客户服务",
          "value": "vip_service",
          "icon": "crown",
          "disabled": true,
          "metadata": {
            "department": "vip_dept",
            "available_time": "09:00-21:00",
            "require_level": "gold"
          }
        }
      ],
    },
    "status": null,
    "error": null
  },
  "metadata": {
    "source": "service_selector",
    "priority": "normal",
    "context": "service_selection",
    "ui_theme": "default"
  }
}
```

#### 列表选择响应示例 (ListSelectionResponseMessageDto)

```json
{
  "message_id": "list_response_001",
  "timestamp": 1703001605000,
  "version": "1.0.0",
  "type": "Response",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "list_selection_response",
    "params": {},
    "data": {
      "selected_items": [
        {
          "id": "service_2",
          "value": "technical_support",
          "text": "技术支持",
          "selected_at": 1703001604000
        }
      ],
      "selection_count": 1,
      "total_items": 4,
      "response_time": 5000,
      "selection_method": "click",
      "cancelled": false
    },
    "status": "Success",
    "error": null
  },
  "metadata": {
    "source": "device_client",
    "response_to": "list_001",
    "device_serial": "VP2024001"
  }
}
```

#### 多选列表示例

```json
{
  "message_id": "multi_list_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "Request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "list_selection",
    "params": {
      "timeout": 60000,
      "priority": "normal",
      "require_response": true,
      "selection_mode": "Multiple"
    },
    "data": {
      "title": "选择您感兴趣的服务",
      "description": "您可以选择感兴趣的服务类型（可多选）：",
      "placeholder": "搜索或选择服务...",
      "options": [
        {
          "label": "在线咨询",
          "description": "即时在线客服咨询",
          "value": "online_consultation",
          "icon": "chat",
          "disabled": false,
        },
        {
          "label": "预约服务",
          "description": "预约线下服务时间",
          "value": "appointment",
          "icon": "calendar",
          "disabled": false,
        },
        {
          "label": "产品介绍",
          "description": "了解产品功能和特性",
          "value": "product_intro",
          "icon": "info-circle",
          "disabled": false
        }
      ]
    },
    "status": null,
    "error": null
  },
  "metadata": {
    "source": "interest_survey",
    "priority": "normal",
    "survey_id": "survey_2024_001"
  }
}
```

### 5. 上下文操作消息 (ContextActionMessageDto)

```json
{
  "message_id": "context_action_001",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "context_action",
    "params": {
      "timeout": 30000,
      "context_type": "user_profile"
    },
    "data": {
      "title": "用户操作",
      "description": "请选择要执行的操作：",
      "actions": [
        {
          "action_id": "view_profile_001",
          "type": "Button",
          "label": "查看资料",
          "icon": "user",
          "style": "Primary",
          "params": {
            "user_id": "user_456",
            "view_mode": "readonly"
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 1
        },
        {
          "action_id": "edit_profile_001",
          "type": "Link",
          "label": "编辑资料",
          "icon": "edit",
          "style": "Default",
          "url": "/profile/edit",
          "open_in_new_window": false,
          "params": {
            "user_id": "user_456",
            "edit_mode": "full"
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 2
        },
        {
          "action_id": "delete_account_001",
          "type": "ApiCall",
          "label": "删除账户",
          "icon": "trash",
          "style": "Danger",
          "params": {
            "api_endpoint": "/api/user/delete",
            "method": "DELETE",
            "user_id": "user_456"
          },
          "enabled": true,
          "requires_confirmation": true,
          "confirmation_text": "此操作将永久删除您的账户，确定要继续吗？",
          "sort_order": 3
        }
      ]
    }
  },
  "metadata": {
    "source": "user_management_service"
  }
}
```

#### 多种操作类型示例

```json
{
  "message_id": "context_action_002",
  "timestamp": 1703001600000,
  "version": "1.0.0",
  "type": "request",
  "session_id": "session_123",
  "user_id": "user_456",
  "payload": {
    "action": "context_action",
    "params": {
      "timeout": 45000,
      "context_type": "document_operations"
    },
    "data": {
      "title": "文档操作",
      "description": "请选择对当前文档的操作：",
      "actions": [
        {
          "action_id": "download_doc_001",
          "type": "Download",
          "label": "下载文档",
          "icon": "download",
          "style": "Info",
          "params": {
            "file_id": "doc_123456",
            "format": "pdf",
            "include_watermark": true
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 1
        },
        {
          "action_id": "share_doc_001",
          "type": "Share",
          "label": "分享文档",
          "icon": "share",
          "style": "Primary",
          "params": {
            "file_id": "doc_123456",
            "share_type": "link",
            "expire_days": 7
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 2
        },
        {
          "action_id": "copy_link_001",
          "type": "Copy",
          "label": "复制链接",
          "icon": "copy",
          "style": "Text",
          "params": {
            "content": "https://example.com/doc/123456",
            "copy_type": "url"
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 3
        },
        {
          "action_id": "view_online_001",
          "type": "Link",
          "label": "在线预览",
          "icon": "eye",
          "style": "Success",
          "url": "/preview/doc/123456",
          "open_in_new_window": true,
          "params": {
            "file_id": "doc_123456",
            "preview_mode": "readonly"
          },
          "enabled": true,
          "requires_confirmation": false,
          "sort_order": 4
        },
        {
          "action_id": "delete_doc_001",
          "type": "ApiCall",
          "label": "删除文档",
          "icon": "trash",
          "style": "Danger",
          "params": {
            "api_endpoint": "/api/documents/123456",
            "method": "DELETE",
            "file_id": "doc_123456"
          },
          "enabled": false,
          "requires_confirmation": true,
          "confirmation_text": "删除后无法恢复，确定要删除此文档吗？",
          "sort_order": 5
        }
      ]
    }
  },
  "metadata": {
    "source": "document_service",
    "document_id": "doc_123456",
    "document_type": "report"
  }
}
```

#### 操作类型说明

根据 `ContextActionType` 枚举，支持以下操作类型：

| 类型 | 描述 | 示例用途 |
|------|------|----------|
| `Button` | 按钮操作 | 触发客户端本地操作或事件 |
| `Link` | 链接跳转 | 页面导航、外部链接 |
| `ApiCall` | API调用 | 服务端接口调用操作 |
| `Download` | 下载文件 | 文件下载、资料获取 |
| `Share` | 分享内容 | 社交分享、链接分享 |
| `Copy` | 复制内容 | 复制文本、链接到剪贴板 |

#### 操作样式说明

根据 `ActionStyle` 枚举，支持以下样式：

| 样式 | 描述 | 视觉效果 |
|------|------|----------|
| `Default` | 默认样式 | 普通按钮样式 |
| `Primary` | 主要操作 | 突出显示的主要按钮 |
| `Success` | 成功样式 | 绿色系，表示正面操作 |
| `Warning` | 警告样式 | 黄色系，表示需要注意 |
| `Danger` | 危险样式 | 红色系，表示危险操作 |
| `Info` | 信息样式 | 蓝色系，表示信息展示 |
| `Text` | 文本样式 | 文本链接样式 |