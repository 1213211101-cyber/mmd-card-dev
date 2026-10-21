# GC 组件库（社区开源教程，旧页专用）

来源：社区开源"完全指南"系列六份，围绕同一套 **GC 引擎协议**。全部为**当前版旧页（路线 0/A/B/C/D/E/T）**技术，DOM 选择器（`.chat .item .touch-scope .content.left`、`.chatMsgTextarea textarea`）、localStorage、fixed 悬浮层**在新页沙盒（路线 S/S+）全部不适用**，新页写卡请走 `references/same-layer-card.md` 与 SBK 基座。

## GC 引擎协议（六份教程的共同地基）

```
AI 输出 [#键=值#] → 正则1 包成隐藏 span.z-live-data → <GC_UI> 触发挂载 <gc-live-widget>
→ Web Components 引擎（Shadow DOM）读数渲染 → renderUI 末尾 window.__GC_DATA = kv（全局数据总线）
→ 派发的 gc-dataupdate 事件供图鉴等组件监听
```

- 所有联动组件（顶部进度条 / 音乐切歌 / 时间轴 / 图鉴）都靠轮询 `window.__GC_DATA` 或监听 `gc-dataupdate`。**引擎不挂全局数据 = 全部联动组件失灵**。
- 单条 replaceString ≤20000 字，超了按"正则1/2/3 跨段拆分"，HTML 标签跨段闭合，用数 `<div>` 配对验证。

## 文件索引

| 文件 | 内容 | 何时用 |
|---|---|---|
| `01-状态栏引擎.md` | 三段正则完整引擎（Web Components + Shadow DOM）、AI 输出规则模板、第一句话初始数据、extractData 兜底解析、字段渲染模板（文本/进度条/标签组/选项按钮）、12 项检查清单 | 任何要带状态栏的旧页卡，先抄这套 |
| `02-全局美化.md` | 全界面 CSS 模板（气泡/导航/输入框/滚动条/指令条）、悬浮球拖拽+日夜/字体开关+偏好记忆、配色替换表、6 大常见错误 | 整卡美化、可开关美化 |
| `03-人设创建面板.md` | 超 2 万字的三段拆分范式、单选/多选标签+自定义输入联动、映射预览、确认追加输入框、7 大 BUG 修复 | 进卡先捏人的初始化表单 |
| `04-音乐侧边栏.md` | BGM 播放器（列表/进度/音量/播放模式）、按状态栏数值阶段自动切歌、自动播放限制处理 | 给卡加 BGM 联动 |
| `05-开屏动画.md` | 全屏开屏覆盖层（粒子+逐行文字动画）、localStorage 只显示一次、开屏期间锁聊天区 | 卡的入场仪式 |
| `06-其他组件合集.md` | 顶部进度条 / 时间轴（阶段突破自动存档）/ 地图 / 状态图鉴（雷达图+收录逻辑）/ 粒子背景 / 小游戏（Canvas 重建坑） | 按需取单个组件 |

## 跨组件通用纪律（六份教程反复出现的硬规矩）

1. 所有可点击元素 `event.stopPropagation()`，否则触发聊天气泡编辑
2. 写输入框后必须 `dispatchEvent(new Event('input',{bubbles:true}))`，否则框架读不到值
3. onclick 调用的函数必须挂 `window.`，标签回调必须传 `event` 参数（要用 `ev.currentTarget`）
4. 防重复初始化哨兵：`if(window.__xxInstalled)return; window.__xxInstalled=true;`
5. 类名统一前缀（gc-/jh-/bgm-/mf-/pd-/evt-/map-），颜色边框背景一律 `!important`，选择器写完整路径，禁 body / `*` 通配
6. localStorage 一律 try/catch；音频 `.play().catch()`（浏览器自动播放限制）
7. z-index 层级规范：粒子 9990 → 侧栏按钮 9998 → 遮罩 100000 → 面板 100001 → 悬浮球 100002 → 开屏 999999
8. 数据渲染用 `textContent`，禁 innerHTML 注入 AI 数据（图鉴/地图渲染自建可信字符串除外）
