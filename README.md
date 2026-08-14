
当前 UI 存在以下问题：
- **文字过多**：每个屏幕都有大段描述文字。用户是扫描式阅读，不会逐字阅读。
- **视觉层级弱**：按钮太小，看起来不像可点击元素。主要操作按钮与背景融为一体。
- **缺乏步骤引导**：Jira 配置页使用纯数字标题 "1) 2)"，没有可视化的进度指示器。
- **信息密度不合理**：配置页的 "Preview" 区域永久占据 40% 的屏幕空间，尽管它是次要信息。
- **登录错误提示不友好**："Session is no longer valid. Please sign in again." 看起来像系统崩溃，而不是友好的提醒。
- **模式选择卡片不明确**：首页的两个模式卡片按钮看起来不像主要操作。

## 设计方向
将其重构成**高密度 SaaS 仪表盘**风格（参考：Linear、Vercel Dashboard、GitHub Settings）。
规则：
- 背景使用纯白或 slate-50。**禁止渐变**。**禁止营销页风格**。
- 间距紧凑。使用 `gap-4`、`p-6`、`rounded-md`（不要用 3xl 超大圆角）。
- 字体：Inter。正文用 `text-sm`，区块标签用 `text-xs uppercase tracking-wide`。
- 按钮：主要操作使用 `bg-slate-900` 或品牌蓝色。仅在必要时全宽；否则使用紧凑的 `px-5 py-2`。
- 表单：`border-slate-300`，`focus:ring-blue-500/20 focus:border-blue-500`，`rounded-md`。

## 逐页需求

### 1. 登录视图
- 居中卡片，最大宽度 `sm`（约 380px）。
- 将红色错误横幅替换为卡片顶部的内联柔和警告框："Session expired. Please sign in again."
- 输入框：`staffId` 和 `Password`。添加显示/隐藏密码的切换图标。
- 按钮："Sign In" — 全宽，`bg-slate-900`，`rounded-md`，`font-medium`。
- 删除所有不必要的文字。不要 "Enterprise Estimation Cockpit" 大段描述。只保留 logo/名称和表单。

### 2. 首页 / 入口视图
- 顶部栏：左侧 Logo，右侧用户头像 "SY" + 名称 "Shalom Yao"。高度 `h-14`。底部边框。
- 标题区：小写大写标签 "Workflow Entry Point"，然后 `h1` "Choose analysis mode"，一行副标题。
- 两个模式卡片并排（使用 `st.columns(2)`）：
  - **Jira Link Analysis**（标签："Single"）：图标 + 标题 + 一行描述 + 蓝色 "Start Jira Analysis" 按钮。
  - **Excel/CSV Analysis**（标签："Batch"）：图标 + 标题 + 一行描述 + 翠绿色 "Start Batch Workflow" 按钮。
  - 卡片样式：`border border-slate-200`，`rounded-lg`，`p-6`。悬停：`hover:border-blue-400` / `hover:border-emerald-400`，微弱阴影。
  - 每张卡片右上角或左上角应有小标签 pill 标明 "Single" 或 "Batch"。
- 页脚：微小元信息 "Output: CSV + Workbook • Master & QA Agents"。

### 3. Jira 配置视图
- 顶部栏保持。
- 面包屑：Home > Jira Link Analysis。
- 标题："Configure Jira Analysis"。
- **步骤条**：水平步骤指示器，共 3 步："Reference"（当前/蓝色）、"Confidence"、"Run"。使用小圆圈 + 连接线。当前步骤为填充蓝色圆圈 + 粗体文字。未激活步骤为空心边框 + 灰色文字。
- **卡片容器**（`border rounded-lg divide-y`）：
  - **区块 1 — Jira Reference**：
    - 标签："1. Enter Jira Reference"（大写、小号、粗体）+ 帮助图标。
    - 输入框占位符："https://jira.example.com/browse/ABC-123 or ABC-123"。
    - 辅助文字："Accepts full Jira URLs or ticket keys only."
  - **区块 2 — Confidence Mode**：
    - 标签："2. Choose Confidence Mode"。
    - 单选选项使用**紧凑列表行**（不要用卡片，不要用横向胶囊）：
      - Auto（默认选中）："Follow workbook indicator or model judgement"
      - Force T-shirt (T)："Override with T-shirt sizing estimate"
      - Force L0："Override with Level 0 estimate"
    - 选中行：`bg-blue-50/30 border-blue-200`。未选中：`border-slate-200 hover:bg-slate-50`。
  - **区块 3 — Preview**：
    - 将其做成**可折叠区块**（`st.expander` 或自定义切换）。默认状态：**折叠**。
    - 标签："Preview Setup" + 眼睛图标。
    - 展开后内容：设置摘要的 bullet list + 一条小号琥珀色提示："Jira backend execution is not wired yet. This screen is the first UI slice only."
- **底部操作**：左侧 "Cancel"，右侧 "Continue"（主按钮，深色）。

## 技术实现说明
- 尽可能使用 Streamlit 原生组件（`st.columns`、`st.container`、`st.expander`、`st.markdown` 带 HTML）。
- 对于自定义样式，使用 `st.markdown(..., unsafe_allow_html=True)` 配合 `<style>` 块，或在顶部通过 `st.markdown` 注入 CSS。
- 所有 CSS 应做作用域隔离，避免污染。使用特定类名如 `.vsip-login`、`.vsip-stepper`。
- 确保布局响应式，但针对**桌面浏览器**优化（最小宽度 1024px）。
- **不要**使用 `st.sidebar` 做此流程。仅使用顶部导航。

## 交付物
提供完整、可直接运行的 Streamlit Python 代码（`app.py` 或拆分为 `pages/`），实现所有 3 个视图的重设计 UI。包含 CSS 块和任何辅助函数。代码应可直接复制粘贴，通过 `streamlit run app.py` 运行。
