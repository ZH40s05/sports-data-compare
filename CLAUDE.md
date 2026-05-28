# 运动数据对比工具

## 项目概览

单文件 H5 网页（`index.html`，~1.9MB），对比不同运动手表的 FIT 心率数据和 GPX 轨迹数据。所有 CSS/JS 内联，无外部依赖，双击即可使用。

## 技术栈

| 组件 | 方案 | 说明 |
|------|------|------|
| 图表 | ECharts 5.5（内联） | 心率曲线图、dataZoom 缩放 |
| 地图 | OpenLayers 10（内联） | 替代 Leaflet；瓦片从高德拉取，无需 API Key |
| FIT 解析 | 自写解析器 `FitReader.parseFIT()` | 只提取 record message (#20) 的 timestamp + heart_rate，~250 行 |
| GPX 解析 | 浏览器 DOMParser | 提取 `<trkpt>` 的 lat/lon/ele/time |
| 坐标系 | `Utils.wgs84ToGcj02()` | WGS-84→GCJ-02 转换，对齐高德瓦片 |

## 文件结构

```
CustomTools/
  index.html       ← 应用本体，所有代码内联
  .gitignore       ← 排除 参考/ .claude/
  参考/            ← 参考网站文件，不入库
```

## 架构（按 `index.html` 内联顺序）

1. **`<style>` OpenLayers CSS** — 地图控件样式
2. **`<style>` 自定义 CSS** — 整体布局、卡片、图表、按钮
3. **`<script>` ECharts** — 图表库（~1MB minified）
4. **`<script>` OpenLayers** — 地图库（~900KB）
5. **`<script>` Utils** — 颜色、Pearson 相关系数、时间格式化、WGS-84→GCJ-02、HTML 转义
6. **`<script>` FitReader** — FIT 二进制解析，完整 base types (0-16)，压缩时间戳
7. **`<script>` GpxReader** — GPX XML 解析 + 边界计算
8. **`<script>` HRCompare** — 心率对比 UI：上传→解析→清洗→概览卡片→ECharts 曲线→相关系数矩阵
9. **`<script>` GPSCompare** — GPS 轨迹 UI：上传→解析→OpenLayers 地图→图层切换
10. **`<script>` 标签切换** — 心率/GPS 面板切换 + resize 处理

## 心率对比功能

- 拖拽/点击上传 `.fit` 文件（最多 10 个）
- 解析后显示每文件概览卡片：平均/最高/最小心率、时长、有效点、原始 0 值、修补点
- 时间轴对齐：相对时间从 0 开始，取所有文件中的最短时长
- 空白值填充：前向填充 + 后向回填，统计修补数量
- ECharts 曲线图：dataZoom 滚轮缩放/拖拽平移、十字准线 tooltip、导出 PNG、重置视图
- Pearson 相关系数矩阵（≥90% 绿色，≥70% 橙色，<70% 红色）

## GPS 轨迹对比功能

- 拖拽/点击上传 `.gpx` 文件
- 高德瓦片 URL（免费，无需 Key）：
  - 道路：`https://webrd01.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}`
  - 卫星：`https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}`
- OpenLayers `ol.source.XYZ` 加载瓦片，`ol.layer.Vector` 叠加轨迹
- WGS-84→GCJ-02 转换后渲染，确保轨迹与道路对齐
- 地图初始化时根据轨迹范围自动适配视野
- 标准图/卫星图切换

## 联网依赖

仅有地图瓦片加载需要网络。ECharts 和 OpenLayers 均已内联，断网时心率功能完全正常，GPS 轨迹线条可显示（地图底图为灰色）。

## 部署

GitHub Pages：`https://zh40s05.github.io/sports-data-compare/`
私有仓库已改为公开以启用 Pages。

## 开发注意事项

- 修改时直接编辑 `index.html`，所有代码内联
- FIT 解析器位于 `FitReader` 模块（搜索 `FitReader` 定位）
- GPS 地图代码位于 `GPSCompare` 模块（搜索 `GPSCompare` 定位）
- 提交前用 `rm -rf .git && git init && git add .gitignore index.html && git commit -m "..." && git remote add origin ... && git push --force origin master` 保持单 commit
- 不要引入 Leaflet（已替换为 OpenLayers）
