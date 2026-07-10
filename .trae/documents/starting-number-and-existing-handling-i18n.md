# Plan: 更新 i18n 文件以支持起始编号与已有位号处理功能

## Summary

iframe/index.html 已实现"起始编号设置"（Auto/Custom）和"已有位号处理"（Clear All/Overwrite/Skip）功能，但 locales 文件缺少这些功能使用的 i18n 键。需补全中英文 locale 文件并验证构建。

## Current State Analysis

- `iframe/index.html`: 已包含完整的 Starting Number 和 Existing Designators UI 及逻辑
- `locales/zh-Hans.json`: 缺少 18 个新 i18n 键
- `locales/en.json`: 缺少 18 个新 i18n 键
- `src/index.ts`: 无需修改

## Missing i18n Keys

HTML 中使用 `t()` 调用但 locale 文件中不存在的键：

1. `Starting Number` → UI 标签
2. `Auto` → 单选按钮
3. `Custom` → 单选按钮
4. `Existing Designators` → UI 标签
5. `Clear All` → 单选按钮
6. `Overwrite Existing` → 单选按钮
7. `Skip Existing` → 单选按钮
8. `Scanning other pages...` → 日志
9. `Max designators on other pages: ${1}` → 日志
10. `Starting number: Auto` → 日志
11. `Starting number: ${1}` → 日志
12. `Existing handling: Clear All` → 日志
13. `Existing handling: Overwrite Existing` → 日志
14. `Existing handling: Skip Existing` → 日志
15. `No designator conflicts with other pages` → 日志
16. `${1} designator conflicts found on other pages` → 日志
17. `Unassigned ${1} conflicts on page ${2}` → 日志
18. `Resolving designator conflicts...` → 状态

## Proposed Changes

### 1. 更新 `locales/zh-Hans.json`

追加 18 个键值对：

| Key | 中文翻译 |
|-----|---------|
| Starting Number | 起始编号 |
| Auto | 自动 |
| Custom | 自定义 |
| Existing Designators | 已有位号处理 |
| Clear All | 全部清除 |
| Overwrite Existing | 覆盖已有 |
| Skip Existing | 跳过已有 |
| Scanning other pages... | 正在扫描其他页面... |
| Max designators on other pages: ${1} | 其他页面最大位号: ${1} |
| Starting number: Auto | 起始编号: 自动 |
| Starting number: ${1} | 起始编号: ${1} |
| Existing handling: Clear All | 已有位号处理: 全部清除 |
| Existing handling: Overwrite Existing | 已有位号处理: 覆盖已有 |
| Existing handling: Skip Existing | 已有位号处理: 跳过已有 |
| No designator conflicts with other pages | 与其他页面无位号冲突 |
| ${1} designator conflicts found on other pages | 在其他页面发现 ${1} 个位号冲突 |
| Unassigned ${1} conflicts on page ${2} | 在页面 ${2} 上取消分配了 ${1} 个冲突位号 |
| Resolving designator conflicts... | 正在解决位号冲突... |

### 2. 更新 `locales/en.json`

追加对应的 18 个英文键值对。

### 3. 运行 `npm run build` 验证构建

确保 i18n 文件格式正确、构建通过。

## Verification

- `npm run build` 成功
- 确认 locale JSON 格式正确（无语法错误）
- 确认所有 HTML 中 `t()` 使用的 key 都在 locale 文件中存在
