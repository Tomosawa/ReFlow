# Plan: Starting Number Setting & Existing Designator Handling

## Context
用户希望在 ReFlow 扩展的重新分配位号功能中，增加两个新配置项：
1. **起始编号设置**：自动（默认）或自定义起始数字
2. **已有位号处理**：清除全部（默认）、覆盖已有、跳过已有（仅在"当前页面"模式下可选）

当前 `reannotatePage()` 使用 `prefixCounters = {}` 每个前缀从 1 开始，没有跨页感知能力。新增功能需要扫描其他页面来获取最大编号和已用位号信息。

## Files to Modify
- `iframe/index.html` — HTML 结构、CSS、JavaScript 逻辑
- `locales/zh-Hans.json` — 中文 i18n
- `locales/en.json` — 英文 i18n

## Implementation

### 1. HTML: 两个新 Section

在"编号模式"和"锁定元器件"之间插入：

**起始编号 Section:**
- Radio: Auto（默认）/ Custom
- Custom 选中时显示 `<input type="number" min="1" value="1">`，Auto 时禁用

**已有位号处理 Section:**
- Radio: 全部清除（默认）/ 覆盖已有 / 跳过已有
- 选择"所有页面"时整个 section 灰掉（`.section.disabled`）

### 2. 新增 `scanOtherPages(currentPageUuid)` 函数

当 scope=current 且 (Auto起始编号 或 处理模式≠clear) 时调用。一次遍历收集三个数据结构：
- `maxByPrefix`: 其他页面每个前缀的最大编号 `{R:30, C:15}`
- `otherUsedSet`: 其他页面所有已用位号 `Set<"R30","C15">`
- `conflictDetails`: 冲突详情 `[{designator, primitiveId, pageUuid, pageName}]`

### 3. 修改 `reannotatePage()` — 增加 options 参数

```js
async function reannotatePage(prefixCounters, pageName, lockedDesignators, pagePrefix, options)
```

- `options.defaultStartNum`: 前缀首次出现时的起始编号（默认1）
- `options.existingHandling`: 'clear' | 'overwrite' | 'skip'
- `options.otherUsedSet`: 其他页面已用位号集合

关键改动：
- `if (!(prefix in prefixCounters)) prefixCounters[prefix] = options.defaultStartNum || 1;`
- Skip 模式：分配前检查 `otherUsedSet`，跳过已用编号
- `while (otherUsedSet.has(...)) prefixCounters[prefix]++;`

### 4. 修改 `startReannotate()` — 整合新逻辑

1. 读取新 UI 状态（startingNumberMode, customStartNum, existingHandling）
2. 需要扫描时调用 `scanOtherPages()`
3. Auto+当前页：用 `scanResult.maxByPrefix` 预填充 `prefixCounters`（max+1）
4. Custom：`defaultStartNum = customStartNum`
5. 传 options 给 `reannotatePage()`
6. Overwrite 模式：分配后调用 `handleOverwriteConflicts()`

### 5. 新增 `handleOverwriteConflicts(changes, scanResult, undoPages)` 函数

1. 从 changes 中收集当前页新分配的位号
2. 在 `scanResult.conflictDetails` 中查找冲突
3. 按页分组，逐页打开并设置冲突位号为 `prefix + "?"`
4. 将修改记录加入 `undoPages`（无需修改 performUndo 逻辑）

### 6. UI 交互

- scope=All Pages → 已有位号处理 section 灰掉
- startingNumber=Auto → 自定义数字输入禁用
- Event listeners 在初始化 IIFE 中注册

### 7. i18n 新增约 17 条键值对

涉及：Starting Number, Auto, Custom, Existing Designators, Clear All, Overwrite Existing, Skip Existing, 扫描日志, 冲突处理日志等。

## Verification
1. Current Page + Auto + Clear All → 位号从其他页最大值+1开始
2. Current Page + Custom:5 + Clear All → 所有前缀从5开始
3. Current Page + Custom:1 + Skip → 跳过其他页已用编号
4. Current Page + Custom:1 + Overwrite → 当前页强制分配，其他页冲突位号变R?
5. All Pages + Auto → 所有前缀从1开始（与之前行为一致）
6. 撤销功能对 Overwrite 模式正确恢复其他页面的位号
7. 构建通过 `npm run build`
