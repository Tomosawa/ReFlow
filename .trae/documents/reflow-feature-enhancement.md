# ReFlow 扩展功能增强计划

## Context
当前扩展"快捷位号分配"已实现基于连通性的位号重分配功能，但功能单一、名称缺乏辨识度。需要重命名为"ReFlow"并增加5项特色功能，使扩展更具竞争力。

## 一、重命名
- `extension.json`: `name` → `"reflow"`, `displayName` → `"ReFlow"`, 菜单 `title` → `"ReFlow"`
- `src/index.ts`: iframe title → `"ReFlow"`
- `locales/extensionJson/zh-Hans.json` / `en.json`: 更新菜单标题映射

## 二、5项新功能

### 功能1: 预览模式 (Preview)
- 新增"预览"和"清除预览"按钮
- 预览时执行只读聚类计算，对每个簇用 `generateIndicatorMarkers()` 绘制矩形标记
- 每个簇不同颜色（HSL色轮均匀分布），半透明(alpha=0.3)
- 清除时调用 `removeIndicatorMarkers()`
- 提取 `computeClusters()` 只读函数，供 Preview 和实际执行共用

### 功能2: 差异报告 (Diff Report)
- 在 `reannotatePage` 中每次修改位号时记录 `{ primitiveId, oldDesignator, newDesignator }`
- 执行完成后在日志区域显示变更报告："R5 → R1", "U3 → U1"
- 保存完整报告到 `sys_Storage`

### 功能3: 锁定保护 (Lock & Preserve)
- 新增输入框，用户输入逗号分隔的位号（如 "U1,R3"）
- 锁定组件跳过修改，保留原位号
- prefixCounter 自动跳过锁定位号编号：若 U1 被锁定，counter 从 2 开始

### 功能4: 撤销 (Undo)
- 执行前在 `reannotatePage` 中收集所有组件的 `{ primitiveId: oldDesignator }`
- 保存到 `sys_Storage` 的 `reflow-undo` 键
- "撤销"按钮读取映射，逐个恢复旧位号
- 页面加载时检查是否有可撤销数据，控制按钮启用/禁用

### 功能5: 每页独立编号 (Per-page Mode)
- 新增 radio: "跨页连续编号"（默认）/ "每页独立编号"
- 每页模式：每页开始时重置 prefixCounters
- 可选 checkbox "添加页号前缀"（如 P1-R1, P2-R1）
- 选择每页模式时才启用页号前缀 checkbox

## 三、关键文件修改

| 文件 | 修改内容 |
|------|----------|
| `iframe/index.html` | 完整重写：UI 新增5个 section、JS 新增10+函数、重构 reannotatePage |
| `src/index.ts` | iframe 高度 500→620，title 改为 "ReFlow" |
| `extension.json` | name/displayName/菜单标题 改为 "ReFlow" |
| `locales/extensionJson/zh-Hans.json` | 更新 "快捷位号分配"→"ReFlow" |
| `locales/extensionJson/en.json` | 更新 "快捷位号分配"→"Fast Reassign"→"ReFlow" |

## 四、UI 布局（从上到下）

```
[ReFlow 位号重分配]                    ← header

[分配范围]                             ← 已有
  当前页面 / 所有页面

[编号模式]                             ← 新增(功能5)
  跨页连续编号 / 每页独立编号
  ☐ 添加页号前缀 (如 P1-R1)

[锁定元器件]                           ← 新增(功能3)
  输入框: U1,R3,C5
  提示: 锁定的元器件将保留当前位号

[操作按钮]                             ← 改造
  [预览] [清除预览] [开始分配] [撤销] [关闭]

[进度条 + 统计]                        ← 已有

[日志 / 差异报告]                      ← 已有+功能2
```

## 五、核心函数变更

### 新增函数
- `computeClusters(lockedDesignators)` — 只读聚类计算（从 reannotatePage 提取）
- `previewClusters()` — 预览入口
- `clearPreview()` — 清除预览
- `generateClusterColors(count)` — 生成 N 种颜色
- `parseLockedDesignators(inputText)` — 解析锁定输入
- `displayDiffReport(allChanges)` — 显示变更报告
- `saveUndoSnapshot(undoMapping)` — 保存撤销快照
- `performUndo()` — 执行撤销
- `checkUndoAvailable()` — 检查撤销可用性

### 修改函数
- `reannotatePage(prefixCounters, pageName, lockedDesignators, pagePrefix)` — 新增 lockedDesignators 和 pagePrefix 参数，增加 changes/undoData 返回
- `startReannotate()` — 读取新 UI 状态（编号模式、锁定、页号前缀），汇总 diff/undo 数据

## 六、API 依赖（已确认可用）
- `generateIndicatorMarkers(markers, color?, lineWidth?, zoom?, tabId?)` — 矩形标记，type="rectangle"
- `removeIndicatorMarkers(tabId?)` — 清除标记
- `sys_Storage.setExtensionUserConfig(key, value)` — 持久化存储
- `sys_Storage.getExtensionUserConfig(key)` — 读取存储
- `sys_Storage.deleteExtensionUserConfig(key)` — 删除存储
- `sch_PrimitiveComponent.modify(primitiveId, { designator })` — 修改位号

## 七、验证方式
1. 安装扩展到 EasyEDA Pro
2. 打开多页原理图，测试5项功能
3. 预览 → 看到彩色矩形标记 → 清除 → 标记消失
4. 锁定 U1 → 执行 → U1 不变，其他重新编号
5. 执行 → 看到 diff 报告 → 撤销 → 位号恢复
6. 每页独立编号 → 执行 → 每页从1开始 → 页号前缀 → P1-R1 格式
