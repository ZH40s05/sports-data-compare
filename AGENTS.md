# AGENTS.md — CustomTools

本目录是社区工具子模块。当前项目为 `sports-data-compare`，单文件 H5 网页，用于对比 FIT 心率数据和 GPX 轨迹数据。

历史 `CLAUDE.md` 保留，不删除；Codex 在本目录内优先遵循本文件。

## 项目概览

- 应用本体是 `index.html`，所有 CSS/JS 内联，无构建步骤。
- 图表使用内联 ECharts 5.5。
- 地图使用内联 OpenLayers 10，瓦片从高德拉取，不需要 API Key。
- FIT 解析由 `FitReader.parseFIT()` 完成，只提取 record message (#20) 的 `timestamp` 和 `heart_rate`。
- GPX 解析使用浏览器 `DOMParser`，提取 `<trkpt>` 的 `lat/lon/ele/time`。
- 坐标转换使用 `Utils.wgs84ToGcj02()`，用于对齐高德 GCJ-02 瓦片。

## 文件结构

```text
CustomTools/
  index.html       # 应用本体，所有代码内联
  .gitignore       # 排除参考资料和本地配置
  参考/            # 参考网站文件，不入库
```

## 开发规则

- 修改功能时直接编辑 `index.html`。
- 定位 FIT 逻辑搜索 `FitReader`。
- 定位 GPS 地图逻辑搜索 `GPSCompare`。
- 不要引入 Leaflet，本项目已切换到 OpenLayers。
- 不要拆分为多文件构建系统，除非用户明确要求。
- 地图瓦片需要网络；断网时心率功能应保持可用，GPS 轨迹线条仍应显示。

## 验证与部署

- 本地验证可直接用浏览器打开 `index.html`。
- 部署依赖 GitHub Pages，推送到 `master` 后自动更新。
- 本目录是 Git Submodule；提交或推送需要在 `CustomTools/` 子模块内单独处理。

## Git 注意事项

旧 Claude 说明中包含重建仓库并 force push 的单 commit 流程。Codex 不应自动执行这类破坏性 Git 操作；只有用户明确要求时才执行，并且必须先说明影响范围。
