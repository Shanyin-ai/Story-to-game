# 分支剧情游戏 JSON 写作规范

本文档说明如何在已有原著小说、短篇小说、剧本或故事大纲的基础上，将其改写成可游玩的分支小说 JSON。重点不是重新发明故事，而是在保留原著气质、人物关系和核心情节的前提下，设计选择、分支、变量、成就和多种结局。

## 1. 基础结构

```json
{
  "meta": {},
  "startNodeId": "start",
  "variables": {},
  "achievements": {},
  "nodes": {}
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `meta` | 是 | 剧本元信息和全局表现设置 |
| `startNodeId` | 是 | 游戏开始节点 ID |
| `variables` | 否 | 自定义变量初始值 |
| `achievements` | 否 | 成就定义 |
| `nodes` | 是 | 所有剧情节点 |

## 2. meta

```json
{
  "meta": {
    "title": "雨夜来信",
    "author": "山音",
    "version": "1.0.0",
    "description": "一封没有署名的信，一场从门缝里涨起的雨。",
    "ambient": "rain",
    "cover": {
      "label": "RAIN TAPE / 01",
      "tagline": "请在午夜前抵达天台，别让雨替你做决定。"
    },
    "variableName": "确信",
    "initialVariable": 35
  }
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `title` | 是 | 作品标题 |
| `author` | 否 | 作者名 |
| `version` | 否 | 剧本版本 |
| `description` | 否 | 作品简介 |
| `ambient` | 否 | 默认环境氛围 |
| `cover.label` | 否 | 封面短标签，例如卷号、类型、章节 |
| `cover.tagline` | 否 | 作品气质短句 |
| `variableName` | 否 | 主状态值名称，例如理性值、燥热值、好感 |
| `initialVariable` | 否 | 主状态值初始值，范围建议 0-100 |

不使用 `subtitle`。

## 3. 明暗模式和氛围

当前启动器不再使用 JSON 的 `theme` 来切换整套 UI 主题色。界面只有浅色和深色两种显示模式，由玩家在启动页或游戏菜单里切换。

作者需要控制的是剧情氛围，也就是 `ambient`。`ambient` 可写在 `meta.ambient` 作为全局默认值，也可以写在节点或结局里临时覆盖。

旧稿如果已经写了 `theme`，启动器会尝试把它当作氛围关键词兼容读取，但新剧本不建议继续写 `theme`。

`ambient` 是节点级环境氛围，适合章节或重要场景，不需要每个节点都写。

| ambient | 效果 |
| --- | --- |
| `none` | 无环境效果 |
| `rain` | 雨线、潮湿感 |
| `wind` | 风、横向流动 |
| `tide` | 海潮、水面呼吸 |
| `sea` | `tide` 的兼容写法 |
| `heat` | 热浪、闷热扭曲 |
| `static` | 电流、信号、档案噪声 |

## 4. 节点 nodes

每个节点是一个剧情单位。节点 ID 必须唯一。

```json
{
  "nodes": {
    "start": {
      "chapterTitle": "第一章：雨夜",
      "title": "门缝里的信",
      "scene": {
        "id": "old_apartment",
        "name": "旧公寓门前",
        "description": "雨水沿着楼道边缘往下淌，门缝里压着一封没有署名的信。",
        "arrival": "你在这里停下脚步。"
      },
      "progress": 5,
      "ambient": "rain",
      "segments": [],
      "choices": []
    }
  }
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `chapterTitle` | 否 | 进入节点时显示的章节幕 |
| `title` | 否 | 当前场景标题 |
| `scene` | 建议 | 当前节点所处场景。场景变化时会触发场景转场 |
| `progress` | 否 | 进度百分比，0-100 |
| `ambient` | 否 | 当前节点环境氛围 |
| `segments` | 否 | 正文段落数组 |
| `choices` | 否 | 选项数组 |
| `next` | 否 | 无选项时自动跳转 |
| `routes` | 否 | 自动条件路由 |
| `isEnding` | 否 | 是否为结局节点 |

### 场景 scene

`scene` 用来告诉玩家“这段对话或叙述发生在哪里”。建议每个有正文或选项的节点都写场景；同一个大场景内连续推进时，保持相同 `scene.id`，播放器不会重复打断。只有从一个大场景切到另一个大场景时，才会播放场景转场动画。

简写形式：

```json
{
  "scene": "旧公寓门前"
}
```

完整形式：

```json
{
  "scene": {
    "id": "old_apartment",
    "name": "旧公寓门前",
    "type": "major",
    "description": "雨水沿着楼道边缘往下淌，门缝里压着一封没有署名的信。",
    "arrival": "你在这里停下脚步。",
    "duration": 1900
  }
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `id` | 建议 | 场景唯一标识。同一场景跨多个节点时必须保持一致 |
| `name` | 是 | 场景名，会作为转场主标题显示 |
| `type` | 否 | 场景类型。`major` 为大场景，会触发转场；`transient` 为过场场景，不触发转场 |
| `description` | 建议 | 场景描述，会显示在场景名下方 |
| `arrival` | 否 | 进入该剧情段时的补充叙述，用于“来到这里”的一两句话 |
| `duration` | 否 | 转场时长，毫秒。不写则使用默认节奏 |

#### 大场景与过场场景

大场景适合连续多个节点共用，例如“厂医办公室”“大烟囱顶”“旧公寓门前”。写法：

```json
{
  "scene": {
    "id": "factory_clinic",
    "name": "厂医办公室",
    "type": "major",
    "description": "药柜贴着墙排开，窗外的厂区广播时断时续。",
    "arrival": "你推门进去，屋里的人同时抬头。"
  }
}
```

过场场景适合一闪而过的地点或瞬间，例如“走廊尽头”“楼梯间”“车窗倒影”。它会更新当前场景名，但不会播放场景转场动画：

```json
{
  "scene": {
    "id": "stairwell_flash",
    "name": "楼梯间",
    "type": "transient",
    "description": "灯管闪了一下。"
  }
}
```

如果省略 `type`，默认按 `major` 处理。也可以用 `kind` 代替 `type`；播放器能识别 `transient`、`passing`、`brief`、`minor` 为过场场景。

写作建议：

- `scene.name` 写地点，不写情绪，例如“厂医办公室”“大烟囱顶”“旧公寓门前”。
- `scene.description` 写稳定环境，例如光线、声音、空间关系。
- `scene.arrival` 写本次进入场景的动作或叙述，例如“你推开门，屋里的人同时回头。”
- 场景不是章节。章节负责作品结构，场景负责当前空间和沉浸感。

### 章节开始节点

如果某个节点是章节开头，应写 `chapterTitle`。

```json
{
  "chapter_02_start": {
    "chapterTitle": "第二章：潮声之前",
    "segments": [
      { "text": "第二天清晨，雨停了。" }
    ],
    "choices": [
      { "text": "走向车站", "next": "station_001" }
    ]
  }
}
```

章节选择只允许回到玩家已经游玩过的章节开始节点。作者需要把真正适合作为“章节入口”的节点标上 `chapterTitle`，不要随意给普通节点加章节标题。

章节入口会记录当时的缓存状态，包括数值、变量、flag、重要决定标签等。玩家之后从章节选择返回时，会回到当时的状态，而不是只跳转到章节文本本身。

## 5. 段落 segments

`segments` 是逐段显示的正文。

```json
{
  "segments": [
    { "speaker": "系统", "text": "雨从傍晚开始下。", "effect": "breath" },
    { "text": "你在旧公寓的门缝里，发现了一封没有署名的信。" }
  ]
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `speaker` | 否 | 说话者。不写则为普通叙述 |
| `text` | 是 | 正文 |
| `effect` | 否 | 单个特效 |
| `effects` | 否 | 多个特效 |

当 `speaker` 是 `系统`、`System`、`旁白`、`Narrator` 时，会显示为旁白样式。

## 6. 互动节奏

不强制规定每个节点只能有多少段，但建议每 1-5 句左右给玩家一次互动。

互动不一定必须是重大分支，也可以是只有一个选项的确认，例如：

```json
{
  "choices": [
    {
      "text": "继续听下去",
      "next": "tape_002"
    }
  ]
}
```

这样做有两个目的：

- 增加玩家代入感，让玩家感觉自己在推进故事。
- 避免长时间纯阅读导致疲劳。

长段落可以存在，但应当用于特殊场景，例如结局、告白、审判、真相揭露、独白高潮。普通推进段落建议保持短促、有节奏。

## 7. 特效 effects

特效适合放在关键句、关键选择或结局上。

```json
{ "text": "脚下忽然一滑。", "effect": "shake" }
```

```json
{ "text": "世界像坏掉的录音一样断续起来。", "effects": ["glitch", "noise"] }
```

| effect | 用途 |
| --- | --- |
| `shake` | 震动、撞击、惊吓 |
| `blur` | 模糊、恍惚、失焦 |
| `noise` | 电流、录音、信号干扰 |
| `flash` | 闪白、瞬间冲击 |
| `blackout` | 黑屏、昏迷、切断 |
| `silence` | 静默、环境被抽空 |
| `heartbeat` | 心跳、压迫、紧张 |
| `pulse` | `heartbeat` 的兼容写法 |
| `breath` | 呼吸、微弱明暗变化 |
| `glitch` | 错位、系统错误、现实断裂 |
| `drown` | 水下、窒息、潮水涌入 |
| `heatwave` | 热浪、眩晕、闷热 |
| `heat` | `heatwave` 的兼容写法 |
| `cold` | 冷光、退色、寒意 |
| `red` | 暗红压迫、危险 |
| `blood` | `red` 的兼容写法 |

## 8. 选项 choices

```json
{
  "choices": [
    {
      "text": "拆开信封",
      "next": "open_letter",
      "changes": {
        "val": 12,
        "set": {
          "letterOpened": true
        },
        "unlockAchievement": "first_choice"
      }
    }
  ]
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `text` | 是 | 选项显示文本 |
| `next` | 是 | 选择后跳转的节点 ID |
| `condition` | 否 | 显示条件 |
| `changes` | 否 | 选择后修改变量 |
| `effect` | 否 | 选择后触发单个特效 |
| `effects` | 否 | 选择后触发多个特效 |

`changes` 中和 flag 相关的常用字段：

| 字段 | 说明 |
| --- | --- |
| `addFlag` | 静默记录一个普通 flag |
| `addFlags` | 静默记录多个普通 flag |
| `importantFlag` | 记录一个重要 flag，并浮动提示“重要决定已记录” |
| `importantFlags` | 记录多个重要 flag，并浮动提示 |
| `important` | 配合 `addFlag` / `addFlags` 使用，设为 `true` 时按重要 flag 处理 |
| `removeFlag` | 移除一个 flag |

## 9. 主状态值 val

内置主状态值 `val`，范围 0-100。适合表示理性、燥热、好感、污染、勇气等核心心理或剧情状态。

```json
{
  "changes": {
    "val": -10
  }
}
```

```json
{
  "changes": {
    "valSet": 50
  }
}
```

`val` 会自动限制在 0-100。

### val 的定位

`val` 不建议作为结局的主要判定条件。它更适合做“体验质感调节器”，影响同一条故事线里的态度、细节和可见信息，例如：

- NPC 对玩家更友善或更冷淡。
- 某些旁支信息、心理描写、场景细节是否出现。
- 同一结局里的 `description`、`closing` 语气略有不同。
- 个别选项是否可见，或某段对话是否更尖锐。

不推荐写成“`val < 30` 直接坏结局 / `val >= 60` 直接好结局”。这样容易形成低数值陷阱，让玩家为了选低 `val` 方向而提前被杀死或提前结束。

结局判定应主要由 flag / 路径组合决定；`val` 可以作为辅助条件，而不是主轴。

## 10. 自定义变量 variables

适合记录剧情状态，例如是否打开信、是否拿到钥匙、角色是否信任玩家。

```json
{
  "variables": {
    "letterOpened": false,
    "trust": 0,
    "route": "normal"
  }
}
```

选项中赋值：

```json
{
  "changes": {
    "set": {
      "letterOpened": true,
      "trust": 2
    }
  }
}
```

当前 `set` 是直接赋值，不做加减运算。数值增减优先使用 `val`。

## 11. flags 标记

`flags` 适合记录一次性事件。

```json
{
  "changes": {
    "addFlag": "checked_envelope"
  }
}
```

重要 flag 可以触发一个轻量浮动提示：“重要决定已记录”。适合用在会影响长期路线、人物关系或结局判断的关键选择上。

```json
{
  "changes": {
    "importantFlag": "saved_lin"
  }
}
```

也可以给重要决定加一个作者可读标签，播放器会显示在提示后：

```json
{
  "changes": {
    "importantFlag": {
      "id": "found_truth",
      "label": "发现真相"
    }
  }
}
```

`label` 会被写入自动存档、手动存档和章节检查点。存档槽与章节选择会显示这些关键选择标签，方便玩家知道这个缓存状态已经走过哪些重要路线。建议 `label` 写成玩家能读懂的短句，例如“救下林”“拆开信封”“承认真相”，不要只写内部变量名。

一次记录多个重要 flag：

```json
{
  "changes": {
    "importantFlags": ["saved_lin", "found_truth"]
  }
}
```

如果想继续使用 `addFlag`，也可以加 `important: true`：

```json
{
  "changes": {
    "addFlag": "chose_dark_path",
    "important": true
  }
}
```

```json
{
  "changes": {
    "removeFlag": "checked_envelope"
  }
}
```

判断：

```json
{ "condition": "hasFlag 'checked_envelope'" }
```

反向判断：

```json
{ "condition": "!hasFlag 'checked_envelope'" }
```

## 12. 条件 condition

条件可以用于选项显示，也可以用于自动路由。

字符串条件：

```json
{ "condition": "val >= 60" }
```

支持：

```text
val >= 60
val <= 30
val > 50
val < 20
val == 80
val != 0
trust >= 2
route == 'true'
hasFlag 'flag_name'
!hasFlag 'flag_name'
default
```

对象条件：

```json
{
  "condition": {
    "all": [
      { "var": "val", "op": ">=", "value": 55 },
      { "flag": "checked_envelope" }
    ]
  }
}
```

```json
{
  "condition": {
    "any": [
      { "var": "val", "op": "<", "value": 20 },
      { "flag": "panic" }
    ]
  }
}
```

```json
{
  "condition": {
    "not": { "flag": "safe" }
  }
}
```

## 13. 自动路由 routes

`routes` 用于不显示选项，直接根据条件跳转。

```json
{
  "routes": [
    {
      "condition": {
        "all": [
          { "flag": "saved_lin" },
          { "flag": "found_truth" }
        ]
      },
      "next": "ending_true"
    },
    { "condition": { "flag": "chose_dark_path" }, "next": "ending_dark" },
    { "condition": "default", "next": "ending_quiet" }
  ]
}
```

规则：

- 按数组顺序判断。
- 第一个满足条件的路由生效。
- 建议最后放一个 `default`，避免无路可走。
- 结局路由优先使用关键 flag / 路径组合；`val` 只建议作为辅助条件，用于区分同一路线中的细节、语气或少量隐藏选项。

## 14. 自动跳转 next

如果一个节点没有选项，但有 `next`，正文播放完后会自动进入下一个节点。

```json
{
  "segments": [
    { "text": "你走进雨里。" }
  ],
  "next": "street"
}
```

普通剧情推进中，建议优先用单选项代替大量自动跳转，让玩家保持参与感。

## 15. 成就 achievements

成就数量应当多于结局数量。结局是故事终点，成就是玩家行动痕迹。

成就适合在这些时刻解锁：

- 做出关键选择。
- 让 `val` 达到关键变化或关键阈值。
- 拿到重要线索、道具、flag。
- 进入隐藏分支。
- 达成某种结局。
- 走出少见路线或反常路线。

成就定义：

```json
{
  "achievements": {
    "first_choice": {
      "title": "雨夜的第一个决定",
      "description": "在门缝里的信前做出选择。"
    },
    "low_sanity": {
      "title": "理性崩落",
      "description": "理性值第一次跌破 30。"
    }
  }
}
```

选项解锁：

```json
{
  "changes": {
    "unlockAchievement": "first_choice"
  }
}
```

一次解锁多个：

```json
{
  "changes": {
    "unlockAchievements": ["first_choice", "secret_reader"]
  }
}
```

结局解锁：

```json
{
  "isEnding": true,
  "title": "结局：潮声归来",
  "type": "TRUE ENDING",
  "achievement": "true_tide",
  "description": "你喊出了那个早已不该被记得的名字。",
  "closing": "后来你仍会在每个雨夜醒来，但这一次，你知道潮声不是为了带走你，而是为了把你送回岸上。"
}
```

## 16. 结局 endings

结局节点必须写：

```json
{
  "ending_true": {
    "isEnding": true,
    "title": "结局：潮声归来",
    "type": "TRUE ENDING",
    "progress": 100,
  "ambient": "tide",
    "achievement": "true_tide",
    "description": "你喊出了那个早已不该被记得的名字。雨停了。",
    "closing": "后来你仍会在每个雨夜醒来，但这一次，你知道潮声不是为了带走你，而是为了把你送回岸上。"
  }
}
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `isEnding` | 是 | 必须为 `true` |
| `title` | 是 | 结局标题 |
| `type` | 否 | 结局类型 |
| `description` | 是 | 结局正文 |
| `closing` | 是 | 结局最后的收束语。用于给玩家一个明确的结束余韵 |
| `progress` | 否 | 通常写 100 |
| `ambient` | 否 | 结局氛围 |
| `effect` / `effects` | 否 | 结局特效 |
| `achievement` | 否 | 抵达结局时解锁的成就 |

每个结局最后都必须有一段话，不要让剧情莫名其妙停住。建议把结局写成两层：

- `description`：说明这个结局实际发生了什么，交代人物、行动、代价或结果。
- `closing`：最后留给玩家的一段收束语，可以是余韵、反讽、遗憾、确认、回声或一句非常短的落幕。

播放器会把 `description` 和 `closing` 连接显示。旧写法中如果使用 `finalText` 或 `epilogue`，播放器也能读取，但新剧本统一推荐使用 `closing`。

结局必须尽量不重样。不同结局不应只是换一句总结，而应对应不同的行动、代价、真相理解、人物关系或世界状态。

推荐结局层级：

| 类型 | 建议 type | 说明 |
| --- | --- | --- |
| 草率结局 | `RASH ENDING` / `BAD ENDING` | 玩家明显选了某种错误选项后，直接进入较短结局线 |
| 分支结局 | `BRANCH ENDING` | 关键抉择后还有较长剧情，甚至继续分叉，最终抵达的结局 |
| 普通结局 | `NORMAL ENDING` | 合理完成故事，但未抵达核心真相或最佳状态 |
| 真结局 | `TRUE ENDING` | 原著内容或作者认定的核心结局 |
| 隐藏结局 | `HIDDEN ENDING` | 需要特殊变量、flag、路线或多次选择才能进入 |
| 特殊结局 | `SPECIAL ENDING` | 可以是玩笑、元叙事、极端路线、彩蛋路线等 |

### 结局判定建议

结局不建议主要依赖 `val` 区间。`val` 可以影响同一故事线中人物态度、细节、可见信息和结尾语气，但不应成为“低 `val` 直接提前结束 / 高 `val` 直接好结局”的主要开关。

更成熟的结局判定方式是 flag / 路径驱动：

```json
{
  "routes": [
    {
      "condition": {
        "all": [
          { "flag": "saved_lin" },
          { "flag": "found_truth" }
        ]
      },
      "next": "ending_true"
    },
    { "condition": { "flag": "chose_dark_path" }, "next": "ending_dark" },
    { "condition": "default", "next": "ending_quiet" }
  ]
}
```

可以把触发条件分成三层：

- 路径结局：约 60-70%，由玩家走过哪些关键节点或重要 flag 组合决定。
- 条件结局：约 15-25%，由 flag 加 `val` 辅助条件决定。
- 稀有结局：约 10-15%，由罕见 flag 组合、隐藏路线或反常选择决定。

如果确实想让低 `val` 方向变黑暗，这条路线也应该同样完整，不应该更短或更敷衍。`val` 更适合在结局内部做变体，例如同一结局下根据 `val` 高低显示不同的 `closing` 语气或最后一句话。

一个成熟剧本通常应当同时拥有：

- 若干草率结局，用于回应明显危险或错误选择。
- 多个分支结局，用于承接关键选择后的长期后果。
- 至少一个普通结局。
- 至少一个真结局。
- 可选隐藏结局或特殊结局。

## 17. 完整示例

```json
{
  "meta": {
    "title": "雨夜来信",
    "author": "示例作者",
    "version": "1.0.0",
    "description": "一封没有署名的信，一场从门缝里涨起的雨。",
    "ambient": "rain",
    "cover": {
      "label": "RAIN TAPE / 01",
      "tagline": "请在午夜前抵达天台，别让雨替你做决定。"
    },
    "variableName": "确信",
    "initialVariable": 35
  },
  "startNodeId": "start",
  "variables": {
    "letterOpened": false
  },
  "achievements": {
    "first_choice": {
      "title": "雨夜的第一个决定",
      "description": "在门缝里的信前做出选择。"
    },
    "true_tide": {
      "title": "潮声归来",
      "description": "抵达真正的潮声。"
    }
  },
  "nodes": {
    "start": {
      "chapterTitle": "雨夜",
      "progress": 5,
      "ambient": "rain",
      "segments": [
        { "speaker": "系统", "text": "雨从傍晚开始下。", "effect": "breath" },
        { "text": "你在旧公寓的门缝里，发现了一封没有署名的信。" }
      ],
      "choices": [
        {
          "text": "拆开信封",
          "next": "open_letter",
          "changes": {
            "val": 12,
            "unlockAchievement": "first_choice",
            "set": { "letterOpened": true },
            "importantFlag": {
              "id": "opened_letter",
              "label": "拆开信封"
            }
          }
        },
        {
          "text": "把信放回原处",
          "next": "ending_rash_silence",
          "changes": { "val": -8 }
        }
      ]
    },
    "open_letter": {
      "progress": 40,
      "ambient": "static",
      "segments": [
        { "speaker": "信", "text": "如果你还记得海，请在午夜前到天台来。", "effect": "glitch" }
      ],
      "choices": [
        {
          "text": "去天台",
          "next": "ending_true",
          "condition": { "flag": "opened_letter" },
          "effect": "flash"
        },
        {
          "text": "留在房间",
          "next": "ending_normal_room",
          "condition": "default"
        }
      ]
    },
    "ending_rash_silence": {
      "isEnding": true,
      "title": "结局：无人回信",
      "type": "RASH ENDING",
      "progress": 100,
      "description": "第二天早晨，门缝里什么也没有。",
      "closing": "你再也没有收到回信。可偶尔你会想，也许沉默本身，就是那个人最后留下的答案。"
    },
    "ending_normal_room": {
      "isEnding": true,
      "title": "结局：留在房间的人",
      "type": "NORMAL ENDING",
      "progress": 100,
      "description": "你没有去天台。雨停后，你开始怀疑那封信是否真的存在过。",
      "closing": "多年以后，你仍会记得那个没有走出去的夜晚。门一直在那里，只是你再也没有伸手。"
    },
    "ending_true": {
      "isEnding": true,
      "title": "结局：潮声归来",
      "type": "TRUE ENDING",
      "progress": 100,
      "ambient": "tide",
      "achievement": "true_tide",
      "description": "你喊出了那个早已不该被记得的名字。雨停了。",
      "closing": "后来你仍会在每个雨夜醒来，但这一次，你知道潮声不是为了带走你，而是为了把你送回岸上。"
    }
  }
}
```

## 18. 写作建议

- 保持互动节奏。通常 1-5 句左右就让玩家点一次选项，即使只是“继续”。
- 不要把所有选择都写成真假题。好的选项应该暴露玩家的态度、欲望、恐惧或误判。
- 成就数量应当多于结局数量，用来记录关键行动和关键状态变化。
- 结局之间要有明显差异，不要只改标题或一句总结。
- `ambient` 用于章节或重要场景，不需要每个节点都换。
- `effect` 用于关键瞬间，不要每一句都加。
- `val` 适合承载主心理状态和体验差异，不建议作为结局主判定。
- 结局优先使用 flag / 路径组合决定；关键选择请用 `importantFlag` 或 `importantFlags` 记录。
- 多结局可以用 `routes` 收束，避免结尾处写大量重复选项。
- 节点 ID 建议使用英文、数字、下划线，例如 `chapter1_start`、`ending_true`。
- 正文里可以使用中文标点和换行，但不要写 HTML 标签；正文会按纯文本显示。
