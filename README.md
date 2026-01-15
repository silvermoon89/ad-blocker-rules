# AdBlocker 规则库

本仓库包含 Windows 广告弹窗拦截器的规则配置文件。

## 文件结构

```
ad-blocker-rules/
└── rules.json    # 规则配置文件
```

## rules.json 文件格式

### 顶层结构

| 字段 | 类型 | 说明 |
|------|------|------|
| `version` | string | 规则库版本号，格式为语义化版本（如 `1.0.0`） |
| `updated_at` | string | 最后更新时间，ISO 8601 格式（如 `2026-01-12T00:00:00Z`） |
| `description` | string | 规则库描述信息 |
| `software` | array | 软件规则列表 |

### software 对象

每个 software 对象代表一个软件的广告拦截规则集合：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | string | 是 | 软件唯一标识符，使用 kebab-case 格式（如 `sogou-pinyin`） |
| `name` | string | 是 | 软件显示名称（如 `搜狗输入法`） |
| `category` | string | 是 | 软件分类（见下方分类列表） |
| `detect` | object | 否 | 软件检测配置（用于自动检测是否安装） |
| `rules` | array | 是 | 该软件的拦截规则列表 |

**软件分类 (category) 可选值：**

- `输入法` - 输入法软件
- `安全软件` - 杀毒/安全卫士类
- `工具软件` - 通用工具类
- `办公软件` - 办公类软件
- `系统工具` - 系统优化/检测类
- `驱动管理` - 驱动安装/更新类
- `即时通讯` - 聊天/通讯类
- `下载工具` - 下载器类
- `视频播放` - 视频播放器类
- `音乐播放` - 音乐播放器类
- `云存储` - 网盘类
- `系统组件` - 系统组件类
- `浏览器` - 浏览器类
- `直播平台` - 直播软件类
- `语音通讯` - 语音软件类
- `压缩工具` - 压缩软件类

### detect 对象

用于自动检测软件是否安装：

| 字段 | 类型 | 说明 |
|------|------|------|
| `registry` | array | 注册表路径列表，存在任一路径则表示已安装 |

### rules 数组

每个规则对象定义一条具体的拦截规则：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | 是 | 规则名称，格式建议：`软件名 - 弹窗类型` |
| `conditions` | array | 是 | 匹配条件列表 |
| `logic` | string | 是 | 条件逻辑运算符 |
| `action` | string | 是 | 匹配后执行的动作 |
| `priority` | number | 是 | 优先级（数值越大优先级越高） |

### conditions 数组

每个条件对象包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | string | 条件类型 |
| `value` | string/object | 匹配值（根据条件类型不同而不同） |

**条件类型 (type) 可选值：**

| 类型 | 值类型 | 说明 | 示例 |
|------|--------|------|------|
| `TitleContains` | string | 窗口标题包含指定文本（不区分大小写） | `{"type": "TitleContains", "value": "推荐"}` |
| `TitleEquals` | string | 窗口标题完全等于指定文本（不区分大小写） | `{"type": "TitleEquals", "value": "广告"}` |
| `TitleRegex` | string | 窗口标题匹配正则表达式 | `{"type": "TitleRegex", "value": ".*广告.*"}` |
| `ClassNameContains` | string | 窗口类名包含指定文本（不区分大小写） | `{"type": "ClassNameContains", "value": "Popup"}` |
| `ClassNameEquals` | string | 窗口类名完全等于指定文本（不区分大小写） | `{"type": "ClassNameEquals", "value": "TipWnd"}` |
| `ClassNameRegex` | string | 窗口类名匹配正则表达式 | `{"type": "ClassNameRegex", "value": "^Ad.*Wnd$"}` |
| `ProcessNameEquals` | string | 进程名完全等于指定文本（不区分大小写） | `{"type": "ProcessNameEquals", "value": "popup.exe"}` |
| `ProcessPathContains` | string | 进程路径包含指定文本（不区分大小写） | `{"type": "ProcessPathContains", "value": "360"}` |
| `WindowSizeEquals` | object | 窗口尺寸完全等于指定值 | `{"type": "WindowSizeEquals", "value": {"width": 300, "height": 200}}` |
| `WindowSizeRange` | object | 窗口尺寸在指定范围内 | `{"type": "WindowSizeRange", "value": {"min_w": 200, "max_w": 500, "min_h": 200, "max_h": 400}}` |

### logic 字段

条件之间的逻辑关系：

| 值 | 说明 |
|----|------|
| `And` | 所有条件都必须满足 |
| `Or` | 任一条件满足即可 |

### action 字段

匹配规则后执行的动作：

| 值 | 说明 | 适用场景 |
|----|------|----------|
| `Close` | 发送关闭消息（WM_CLOSE） | 大多数弹窗，温和关闭 |
| `Destroy` | 强制销毁窗口（WM_DESTROY） | 顽固弹窗，无法正常关闭时使用 |
| `Minimize` | 最小化窗口 | 不想关闭但希望隐藏的窗口 |
| `Hide` | 隐藏窗口 | 隐藏而不关闭 |
| `KillProcess` | 终止进程 | 最强力的方式，慎用 |

### priority 字段

规则优先级（整数），数值越大优先级越高。建议范围：

| 范围 | 适用场景 |
|------|----------|
| 100 | 高优先级 - 专用广告进程（如 `popwnd.exe`） |
| 80-90 | 中高优先级 - 明确的广告弹窗 |
| 60-79 | 中优先级 - 一般推广弹窗 |
| 40-59 | 中低优先级 - 可能误判的弹窗 |
| 1-39 | 低优先级 - 测试性规则 |

---

## 如何添加新规则

### 方法一：直接编辑 rules.json

1. 找到对应软件的 `software` 对象（如果不存在则新建）
2. 在该软件的 `rules` 数组中添加新规则

**示例：为搜狗输入法添加新规则**

```json
{
  "name": "搜狗输入法 - 新发现的弹窗",
  "conditions": [
    {"type": "ProcessNameEquals", "value": "SogouNewPopup.exe"}
  ],
  "logic": "And",
  "action": "Close",
  "priority": 90
}
```

### 方法二：添加新软件

在 `software` 数组中添加新的软件对象：

```json
{
  "id": "new-software",
  "name": "新软件名称",
  "category": "工具软件",
  "detect": {
    "registry": [
      "SOFTWARE\\NewSoftware",
      "SOFTWARE\\WOW6432Node\\NewSoftware"
    ]
  },
  "rules": [
    {
      "name": "新软件 - 广告弹窗",
      "conditions": [
        {"type": "ProcessNameEquals", "value": "newpopup.exe"}
      ],
      "logic": "And",
      "action": "Close",
      "priority": 80
    }
  ]
}
```

---

## 规则编写技巧

### 1. 使用组合条件提高准确性

单一条件可能导致误判，建议组合多个条件：

```json
{
  "name": "精确匹配示例",
  "conditions": [
    {"type": "ProcessPathContains", "value": "SomeSoftware"},
    {"type": "ClassNameContains", "value": "PopupWnd"},
    {"type": "TitleContains", "value": "推荐"}
  ],
  "logic": "And",
  "action": "Close",
  "priority": 80
}
```

### 2. 针对专用广告进程使用高优先级

如果确定某个 exe 是纯广告进程，可以只匹配进程名：

```json
{
  "name": "纯广告进程",
  "conditions": [
    {"type": "ProcessNameEquals", "value": "adpopup.exe"}
  ],
  "logic": "And",
  "action": "Close",
  "priority": 100
}
```

### 3. 使用窗口尺寸辅助判断

广告弹窗通常有固定或特定范围的尺寸：

```json
{
  "name": "小尺寸广告弹窗",
  "conditions": [
    {"type": "ProcessPathContains", "value": "SomeSoftware"},
    {"type": "WindowSizeRange", "value": {"min_w": 200, "max_w": 400, "min_h": 150, "max_h": 300}}
  ],
  "logic": "And",
  "action": "Close",
  "priority": 70
}
```

### 4. 常见广告关键词

标题中常见的广告关键词：

- 推荐、热门、精选
- 特惠、优惠、限时
- 会员、VIP、升级
- 新闻、资讯、热点
- 直播、游戏

---

## 如何获取窗口信息

在 AdBlocker 应用中，可以通过以下方式获取窗口信息用于编写规则：

1. 打开 AdBlocker 主界面
2. 进入「仪表盘」页面
3. 查看「当前窗口」列表，可以看到所有窗口的：
   - 窗口标题
   - 窗口类名
   - 进程名
   - 进程路径
   - 窗口尺寸

或者使用 Windows 自带的 Spy++ 工具获取窗口信息。

---

## 注意事项

1. **测试规则**：添加新规则后，建议先测试确保不会误拦截正常窗口
2. **备份配置**：修改前建议备份原文件
3. **JSON 格式**：确保 JSON 格式正确，可使用 JSON 校验工具检查
4. **谨慎使用 KillProcess**：终止进程可能导致数据丢失，慎用
5. **优先级设置**：合理设置优先级，避免低优先级规则被高优先级规则覆盖

---

## 贡献规则

欢迎提交新的规则！请确保：

1. 规则经过测试，确认可以正确拦截目标弹窗
2. 规则不会误拦截正常窗口
3. 遵循本文档的格式规范
4. 提供规则的测试环境说明（软件版本等）

## License

MIT
