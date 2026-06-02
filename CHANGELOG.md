# Changelog

## 2026-06-01 — 趋势图重构 & 桌面图标修复

### 趋势线 → 平滑面积堆叠图

| 变更 | 之前 | 之后 |
|---|---|---|
| 图表类型 | 柱状图 (`type: 'bar'`) | 平滑面积曲线 (`type: 'line'` + `fill: true`) |
| 趋势算法 | 线性回归 (直线) | 居中移动平均 (自然跟随数据起伏) |
| 曲线平滑 | `tension: 0.4` | `tension: 0.5` + `cubicInterpolationMode: 'monotone'` |
| 视觉效果 | 柱状 + 单一趋势线渐变 | 双层渐变面积叠加：Streak 暖红 + Trend 冷蓝 |
| 高亮 | 柱状金色标识最高 | 最高点金色圆点 (radius 6 vs 4) |

**面积堆叠设计**：
- **Streak 面积层** (下层 `order: 2`)：暖红渐变 `rgba(196,30,58,0.42)` → 透明
- **Trend 面积层** (上层 `order: 1`)：冷蓝渐变 `rgba(107,155,210,0.32)` → 透明
- 两层透明度叠加在交汇处产生紫色过渡，视觉上自然圆滑
- `monotone` 插值模式保持曲线单调性，避免贝塞尔曲线过冲变形

### 桌面图标数字更新修复

**根因**：`updateAppIcon()` 调用了 `lastStreakDays()`（已结束的 streak），而非 `getCurrentStreak()`（当前进行中的 streak）。

**修复**：
- 图标数字改为当前坚持天数；reset 后回退显示上次记录
- 新增 `<link rel="icon">` 动态 favicon，浏览器标签页实时反映天数
- 接入 **Badging API**（`navigator.setAppBadge`），Windows 任务栏角标同步更新
- favicon / apple-touch-icon / manifest 三路同步刷新

### 云同步配置增强

- 新增 **Save** 按钮：仅保存 URL / Anon Key / Sync Key 到 localStorage，不触发网络请求
- 新增 **Clear** 按钮：一键清空三个字段 + 清除已保存配置 + 重置同步状态灯

### 项目精简

- 删除独立文件 `streak-tracker.html`，仅保留 `streak-repo/index.html` 作为唯一迭代目标

## 2026-06-02 — 图标静态化

### 诊断结论

动态天数图标在已安装的 PWA 桌面图标上**无法更新**，这是 OS 级别的硬限制：
- **iOS**：添加到桌面时截屏固化，无 API 可更改
- **Android**：同 iOS，Badging API 仅支持 Chrome 且需 Service Worker，只显示小角标而非图标本身

### 改动

- 移除 `generateIconSVG()` / `lastStreakDays()` / Badging API 等全部动态图标逻辑
- `favicon` / `apple-touch-icon` / `manifest` 统一使用静态 `streak-icon.png` (512×512)
- `updateAppIcon()` 简化为仅刷新 manifest
