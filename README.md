# gpt-image2-webpage

基于 OpenAI 兼容 API 的本地个人图片生成站，支持 **GPT Image 2** 和 **Seeddream 5.0 Pro** 双模型，异步任务提交、轮询、预览与下载。

**零依赖、单文件**，双击 `gpt-image2本地工作站_旗舰版.html` 即可在浏览器中使用。

![GPT Image2 个人图片站截图](example.jpg)

---

## 功能概览

| 功能 | 说明 |
|------|------|
| 文生图 | 输入 Prompt 生成图片 |
| 图生图 | 提供参考图 URL + 可选蒙版 URL，进行局部编辑或视觉延展 |
| 双模型支持 | 切换 GPT Image 2 / Seeddream 5.0 Pro，参数面板自动适配 |
| 异步轮询 | 提交任务后自动轮询，最长等待 10 分钟 |
| 手动查询 | 通过 request_id 单独查询任务状态和结果 |
| 图片下载 | 逐张下载生成结果，自动推断文件扩展名 |
| 任务恢复 | 页面加载时自动恢复上次未完成的任务 |
| 参数持久化 | 所有参数（含模型特有参数）保存到 localStorage，刷新不丢失 |

---

## 多模型支持

页面内置两个模型，通过设置面板的 **Model ID** 下拉框切换：

| 模型 | Model ID | 品牌标题 |
|------|----------|----------|
| GPT Image 2 | `openai/gpt-image-2` | GPT Image2 个人图片站 |
| Seeddream 5.0 Pro | `bytedance/doubao-seedream-5-0-pro` | Seeddream 5 Pro Image2 个人图片站 |

切换模型后，左侧参数面板、右侧 Meta 摘要、品牌标题均自动切换，无需手动刷新。

---

## 接口设置

点击 **「接口设置」** 按钮，填写以下三个参数（仅保存在浏览器 localStorage，不会上传到任何服务器）：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| Base URL | 你的 API 端点地址 | 无（需自行填写，例如 `https://api.example.com/v1`） |
| API Key | 你的个人密钥 | 无（需自行填写） |
| Model ID | 模型标识（下拉选择） | `openai/gpt-image-2` |

> 完整请求端点会自动拼接为 `{Base URL}/tasks/generations`，在侧边栏实时预览。

---

## 参数说明

### GPT Image 2 参数

对照 [OpenAI Image Generation API 官方文档](https://developers.openai.com/api/docs/guides/image-generation#customize-image-output) 设计。

#### 核心参数

| 参数 | 类型 | 可选值 | 说明 |
|------|------|--------|------|
| `prompt` | 文本 | — | 必填，描述画面主体、材质、构图、镜头、光线、风格等 |
| `n` | 数字 | 1–10 | 一次生成几张不同的完整图片 |
| `size` | 下拉 | `auto` / `1024x1024` / `1536x1024` / `1024x1536` / `2048x2048` / `2048x1152` / `3840x2160` / `2160x3840` | 输出图片尺寸 |
| `quality` | 下拉 | `auto` / `low` / `medium` / `high` | 渲染质量 |
| `background` | 下拉 | 留空 / `auto` / `opaque` | 背景模式（gpt-image-2 不支持 transparent） |

#### 输出参数

| 参数 | 类型 | 可选值 | 说明 |
|------|------|--------|------|
| `output_format` | 下拉 | 留空 / `png` / `jpeg` / `webp` | 输出文件格式 |
| `output_compression` | 数字 | 0–100 | 仅对 jpeg/webp 有效，压缩级别 |

#### 高级参数

| 参数 | 类型 | 可选值 | 说明 |
|------|------|--------|------|
| `moderation` | 下拉 | 留空 / `auto` / `low` | 审核严格度 |
| `partial_images` | 数字 | 0–3 | 生成过程中的中间预览图数量（流式传输时生效） |

---

### Seeddream 5.0 Pro 参数

Seeddream 模型有独立的参数面板，切换模型时自动显示。

| 参数 | 类型 | 可选值 | 说明 |
|------|------|--------|------|
| `n` | 数字 | 1–10 | 生成张数 |
| `size` | 下拉 | `1K` / `2K` / `自定义` | 输出尺寸，选「自定义」时可手动输入宽x高（如 `1280x720`） |
| `output_format` | 下拉 | `jpeg` / `png` | 输出文件格式 |
| `response_format` | 下拉 | `url` / `b64_json` | 返回格式（URL 链接或 Base64 编码） |
| `watermark` | 下拉 | 开启 / 关闭 | 是否添加水印 |
| `optimize_prompt` | 下拉 | 开启 / 关闭 | 是否自动优化 / 改写提示词（Seeddream 专有参数） |

> `optimize_prompt` 是 Seeddream / 豆包专有参数，GPT Image 2 及其他 OpenAI 兼容模型不支持，请求时只对 Seeddream 模型携带此字段。

---

### 图生图扩展（两模型通用）

| 参数 | 说明 |
|------|------|
| 参考图 URL | 图生图模式必填，支持 HTTP/HTTPS 或 OSS 地址 |
| 蒙版 URL | 可选，用于指定编辑区域 |

---

## 超时与轮询设置

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 轮询间隔 | 5 秒 | 每次查询任务状态的间隔 |
| 总超时 | 10 分钟 | 超过此时间未返回结果则报超时 |
| completed 重试 | 最多 10 次 | 状态为 completed 但图片未落盘时的额外等待 |
| 手动查询 | 无限制 | 通过 request_id 在「请求查询板块」随时查询 |

---

## 错误处理

### 错误展示机制

发生错误时会同时触发两种展示：

1. **Toast 弹窗**（右上角，5 秒自动消失）— 快速感知
2. **右侧错误卡片**（持续展示，直到下次生成）— 包含完整错误信息、Request ID、时间戳，便于截图和排查

### 常见错误

| 错误信息 | 可能原因 | 解决方法 |
|----------|----------|----------|
| 请先在设置里填写 Base URL、API Key 和 Model ID | 接口未配置 | 点击「接口设置」填写参数 |
| 请输入提示词后再开始生成 | Prompt 为空 | 输入画面描述 |
| 图生图模式需要填写参考图 URL | 图生图模式未提供参考图 | 填写参考图 URL |
| Seeddream 自定义尺寸需要填写有效的宽x高 | 自定义尺寸未填写或格式错误 | 填写如 `1280x720` 格式 |
| 异步任务提交失败 | 接口地址/密钥/模型配置错误 | 检查设置中的参数 |
| 图片生成失败 | 模型返回错误状态 | 查看错误详情，调整 Prompt 或参数 |
| 图片生成超时（已等待 10 分钟） | 任务耗时过长 | 稍后重试或简化提示词 |
| 任务已完成但未能提取图片链接 | API 返回格式与预期不一致 | 复制 request_id 手动查询 |

---

## API Key 安全性

| 安全措施 | 说明 |
|----------|------|
| 本地存储 | Key 仅保存在浏览器 localStorage，不同设备互不可见 |
| 传输方式 | 仅通过 HTTPS `Authorization: Bearer` 头发送至用户配置的 Base URL |
| 零第三方依赖 | 纯内联 CSS + 原生 JS，无任何外部统计/追踪脚本 |
| 默认值为空 | HTML 文件中不含任何硬编码 Key |
| 输入掩码 | 使用 `<input type="password">`，输入时不可见 |
| 摘要脱敏 | 侧边栏显示格式为 `sk-xx••••xxxx` |

> 建议通过文件（如邮件附件、网盘）发送 HTML 文件，让收件人在本地浏览器用 `file://` 协议打开。

---

## 技术架构

| 维度 | 详情 |
|------|------|
| 文件 | `gpt-image2本地工作站_旗舰版.html`（单文件） |
| 外部依赖 | 无 |
| CSS | 内嵌 `<style>`，Glass Morphism 风格 |
| JavaScript | 内嵌 `<script>`（IIFE 模式），原生 ES6+ |
| 数据持久化 | `localStorage`（3 个 key：settings / controls / recentTask） |
| API 端点 | `POST /tasks/generations`（提交）<br>`GET /tasks/generations/{request_id}`（查询）<br>端点拼接在 Base URL 之后，例如 `https://your-api.com/v1/tasks/generations` |
| 响应格式 | 兼容多种嵌套结构（URL 图片 + Base64 图片），含 doubao/seeddream 特有路径 |
| 模型适配 | 根据 `isSeedreamModel()` 动态组装请求体，GPT Image 2 和 Seeddream 参数完全隔离 |

---

## 部署说明

1. 将 `gpt-image2本地工作站_旗舰版.html` 发送给用户
2. 双击在浏览器中打开
3. 填写自己的 Base URL、API Key，选择 Model ID
4. 开始使用

---

## 版权

made by [Winston](https://github.com/Winston-Tao1)