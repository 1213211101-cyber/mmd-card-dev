# MUV / 3D 卡 与 舞台能力台（新页沙盒·真机实测）

来源：`MUV卡_落地指南.md`（真机实测：Android 16 / Chrome 154 WebView / 480×925 / DPR 2.25 / PowerVR SGX）+ 样本卡 `assets\community-samples\舞台3D能力台_v0.6.2.json`（探测骨架卡，8 段正则，validate 0 错）。

## 一、外链通道事实（真机实测，不是推测）

| 通道 | 结果 | 结论 |
|---|---|---|
| `fetch()` | ❌ Failed to fetch | 不能拉模型 |
| `XMLHttpRequest` | ❌ 被拦 | 不能拉模型 |
| 平台自有域名 | ❌ fetch/XHR 都被拦 | 传上去也没用 |
| `<script src>` | ✅ unpkg / jsdelivr / cdnjs 全通 | **唯一模型通道** |
| `<img>` | ✅ 通 | 贴图专用 |
| 内联 base64 → `loader.parse()` | ✅ 真机渲染成功 | 体积极小才行 |

**铁律：模型必须 base64 包装成 `.js`（`window.__XXX="..."`），用 `<script src>` 加载，再 `loader.parse()`。** 这是该平台唯一真模型通道。

## 二、落地流程与体积/性能预算（实测值）

流程：模型（VRM/GLB ≤3MB）→ base64 转 .js（×1.37）→ 托管到可被 `<script src>` 引到的地方（jsdelivr/GitHub 仓库、https 直链 CDN、平台文件托管需给 JS content-type）→ 卡内 loadScript 链式加载 three.js + GLTFLoader + 模型.js → parse 渲染。

| 项 | 数值 |
|---|---|
| 模型单文件 | 建议 ≤3 MB，最多 8 MB |
| 首屏 4 MB js | 3–8 秒（4G），**必须给进度提示** |
| FPS | 立方体 37–47 / 空场景 50–56 → 模型面数 **≤3 万**、骨骼 ≤80 |
| 纹理 | ≤2048（上限 8192），无浮点纹理 → 不做 HDR |
| 物理 | WASM 能跑但保守，刚体 ≤30 |
| 缓存 | localStorage / IndexedDB / Cache / OPFS 全有 → 二次进卡可秒开 |

三种 MUV 卡：A 展示向（VRM+idle，触摸旋转/双指缩放，★★）/ B 演出向（VMD→glTF，事件触发 clipAction().play()，★★★）/ C 交互向（滑杆改 bones 旋转、换装=换 material.map、姿态存 sdk.save，★★★★）。

## 三、MUV 必踩的坑

1. 不能自动播放/自动加载大文件——必须玩家点一下再拉
2. content-type 要小心：托管方带 `nosniff` + `octet-stream` 会被 `<script>` 拒
3. 别把模型内联进卡——单段上限 20000 字符只够 10 KB
4. 一次只加载一个模型，同时拉两个 4 MB js 卡死低端机
5. 失败兜底退回静态立绘，不能白屏
6. MMD 模型动辄 5 万面+200 骨骼，务必减面减骨

## 四、能力台骨架卡架构（v0.6.x 样本）

- 8 段正则各干一件事：SW1 样式（全 CSS 变量驱动换配色）/ SW2 核心（舞台 DOM+主题存 sdk.save+面板+三通道复制）/ SW3 八组探测（图形/计算/外链/输入/基准/音频/存储/宿主）/ SW4 冒烟演示（纯 WebGL 立方体、WASM）/ SW5 模型直链实测（预设 6 条验过 CORS 的样例链接）/ SW6 MUV A/B/C 可跑模板（2D 火柴人骨架引擎演示绑骨）/ SW7 加载通道实测（内联 1664 字节 Box.glb 走 base64→parse）/ SW8 端到端真模型渲染。
- 共享总线 `window.__SW`：`__SW.stage` 舞台根、`__SW.add(组,名,值,状态,提示)` 注册探测、`__SW.render()` 重绘、`__SW.demo(标题,标签,描述)` 领演示位——后续段只认总线不认 DOM，段间解耦。
- 每段幂等哨兵（`if(G.demos7) return; G.demos7=1`），全部 try/catch 包裹，复制功能三通道兜底（Clipboard API → execCommand → 长按文本框）。
- 「结论」页把探测结果翻译成本机路线（能直链拉 or 只能 base64+.js），并生成一段可直接丢给 AI 的机况描述——这是"探测卡"最值得抄的设计。

## 五、与同层卡流派（same-layer-card.md）的差异——用哪套要看场景

能力台是**工具/骨架卡**，同层八条是**内容卡**纪律，两者冲突点与取舍：

| 点 | 同层卡纪律 | 能力台做法 | 取舍 |
|---|---|---|---|
| 启动 | 轮询+巡检重挂，事件 document 捕获委托 | 顶层 IIFE 直接建 DOM、直绑元素 | 能力台 DOM 全由脚本自建（无平台注入 HTML），JS 不跑=什么都没有，"默认可见全屏层"风险天然不存在；内容卡有平台注入的 src 块，必须守八条 |
| 顶层碰 DOM | 禁（红线 2/3） | 顶层 `document.body.appendChild` | 真机可跑（脚本抽出时 body 已就绪），但**保守写法仍是挂 mount 回调**；新写卡建议守红线，能力台属于已验证的特例 |
| data-* | 禁（红线 5） | 运行时用 setAttribute 设 data-k | 净化只扫平台注入内容，运行时自建节点的 data-* 存活；仍建议只当临时查找键，别当状态载体 |
| 标签 | 只用 div/span（aside 会脱壳） | 曾用 `<aside>`，v0.6.2 已改 div | 脱壳只作用于注入 HTML，运行时 innerHTML 不受影响；但统一用 div/span 无害，已对齐 |
| z-index | 3500–7999 可用带 | 1500/2000（一千段压顶栏） | 能力台刻意压顶栏做全屏舞台；普通卡别学 |

写 MUV/3D 内容卡时：舞台骨架抄能力台（总线+哨兵+配色变量），生命周期与事件纪律守同层八条。

## 六、交付纪律补充

- validate 0 错是底线：regex id 必须负数、一条规则恰好四字段（无 trimStrings）、findRegex 写 slash 形态。
- innerHTML/cssText 警告在"脚本运行时自建 DOM"场景可接受（平台净化管不到），平台注入 HTML 场景不可接受。
- 版本号三处对齐：文件名、卡内 chip、copyJson 的 ver。
