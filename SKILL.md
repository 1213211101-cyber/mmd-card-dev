---
name: mmd-card-dev
description: 魅魔岛（MMD）AI 文字角色卡开发总纲。当任务涉及魅魔岛/MMD/写卡/角色卡/状态栏/选项栏/开场白/正则美化/regex_scripts/世界书/舞台模式/SBK 基座/沙盒技能/开屏动画/音乐侧边栏/人设创建面板/全局美化/地图/时间轴/图鉴/MUV卡/3D卡/能力台等卡内组件时使用。涵盖三代平台分层、硬上限红线、路线选择、构建交付纪律、关系网与文风纪律、社区成品样本、GC 组件库。
---

# 魅魔岛（MMD）角色卡开发总纲

所有知识的唯一权威总文档：`references/魅魔岛状态栏开发通用安全规则_v2.4.md`（本仓库，1100 行，v2.4 为最新版，旧版 v1.0~v2.3 仅考古）。**开始任何写卡任务前先读它**，本文件只是入口与速查。

## 第一步：定代际，再选路线

平台三代分层，先确认用户说的是哪一代，路线完全不同：

1. **新页沙盒**（chatVersion:1，路线 S/S+，SBK 基座）——最新测试版
2. **当前版旧页**（路线 0/A/B/C/D/E/T）——现行主流
3. **旧版**（2026-08-25 已退役）——不碰

新页与旧页的 CSS 变量、DOM 类名、SDK、红线**全部不通用**，拿旧页经验写新页卡必爆。代际判定细节见 v2.4 第一部分。

## 硬上限速查（超限=静默拒存/整卡报废）

| 字段 | 上限 | 备注 |
|---|---|---|
| scriptName | ≤20 字 | |
| findRegex | ≤1000 字 | |
| replaceString | ≤20000 字 | 编辑器超限=整次静默拒存 |
| statusbar | ≤200 字 | |
| beginning | **真上限 4000** | 官方校验的 10240 是错的 |
| personality | ≤10000 字 | |
| regex 规则 | ≤130 条 | |
| 世界书标题 | ≤20 字 | |

- 导入 JSON 只有 6 键白名单：chatVersion / pageDepth / statusbar / beginning / personality / regex_scripts
- 交付两路线：A=v2 整卡 PNG，B=分离式三件套；共同铁律=必须新建卡+首次保存前选"使用新版"

## 新页沙盒七条红线（路线 S 必背）

1. chatVersion 必写
2. 脚本只跑一次 + 顶层别碰 DOM + 幂等哨兵
3. mount 别读 body
4. 匹配式预算与空串禁令
5. 自建 data-* 属性会被平台删除
6. **反引号才是爆代码真凶**（不是引号）
7. 净化转义 `]>` 会删属性

事件时序：mount/done 有补发、ready 最后到且不补发。`save.remove` 是坏的，用 `save.set(key, null)` 代替，save.get 要 try/catch。平台事实冲突时以 tavern-mmd 的 `mmd-sandbox.md`（源码逆向+实机探针）为准，不信官方手册。

## 资源指针

> 规则总纲与社区样本随本仓库发布；工具链脚本在配套开源仓库 [tavern-mmd](https://github.com/yofengi/tavern-mmd) 中（下表路径为该仓库内路径，克隆到本地即可用）。

| 资源 | 路径 | 用途 |
|---|---|---|
| 规则总纲 v2.4 | `references/魅魔岛状态栏开发通用安全规则_v2.4.md`（本仓库） | 先读这个 |
| 静态审核 | tavern-mmd 仓库 `skills/tavern-mmd/scripts/validate.py <文件> --platform <mmd\|mmdsandbox\|st>` | 交付前 0 错 |
| 三面板预览 | tavern-mmd 仓库 `skills/tavern-mmd/scripts/build-preview.py` | 空白条/变量失效预览期拦下 |
| SBK 基座 | tavern-mmd 仓库 `skills/tavern-mmd/assets/sandbox-kit/build_sbk.py` | 新页路线 S+ 改 config 即得 6 键 JSON |
| 沙盒仿真器 | tavern-mmd 仓库 `skills/tavern-mmd/scripts/mmdsandbox-sim.js` | 本地回归，不默认登真站 |
| 平台权威规范 | tavern-mmd 仓库 `skills/tavern-mmd/references/platforms/mmd-sandbox.md` / `mmd.md` | 新页/旧页各一份 |
| 官方技能包 | 本机资产，未随仓库发布（generating-role-card：authoring 四章+fixtures） | 可抄代码 |
| 社区加固版 | [mmd-generating-role-card](https://github.com/czjainiaini/mmd-generating-role-card) 仓库 `generating-role-card/`（若无法访问可向作者索取） | 工作流/验收契约/舞台游戏简报 |
| 黑洞猫 8.30.4 美化 | 本机资产，未随仓库发布（可按需联系作者索取） | 旧页日夜双版成品，css1 只剩 300 字余量 |
| DIS 实战项目 | 本机项目，未随仓库发布（可按需联系作者索取） | 唯一完整实战案例，构建脚本+128 项测试 |
| 鲨鱼卡样本 | `assets\community-samples\鲨鱼卡-角色卡正则数据.json` | 旧页轻量派成品：配角状态栏+三选项栏+日夜切换+角色图鉴+微调面板 |
| 补梦卡样本 | `assets\community-samples\补梦卡-正则数据.json` | 功能型卡完整样本：29 条正则，委托单/HUD/状态解析/选项全链路 |
| 能力台样本 | `assets\community-samples\舞台3D能力台_v0.6.2.json` | 新页 3D/MUV 探测骨架卡：8 段正则+__SW 总线，validate 0 错 |

工具链在 Windows 下运行时加 `PYTHONUTF8=1`。

## 详细参考

- 沙盒红线详解、排错速查表（症状→首查项）、顺口溜 → [references/sandbox-redlines.md](references/sandbox-redlines.md)
- 同层全屏卡制作规范（新页实战流派：六硬机制、写法八条、BUG 库、官方作者区框架） → [references/same-layer-card.md](references/same-layer-card.md)
- 构建链纪律、文风纪律、关系网纪律、交付纪律、DIS 项目教训 → [references/workflow.md](references/workflow.md)
- 轻量选项栏/配角状态栏协议（旧页正则派：AI 输出格式+正则实现+日夜切换，与同层卡的分工） → [references/option-statusbar-lite.md](references/option-statusbar-lite.md)
- 开场白生成协议（社区分享版原文+嵌入工程注意：字数预算、触发词、捏人冲突） → [references/opening-protocol.md](references/opening-protocol.md)
- GC 组件库（社区开源教程六份，**旧页专用**：状态栏引擎/全局美化/人设创建面板/音乐侧边栏/开屏动画/进度条·时间轴·地图·图鉴·粒子·小游戏，全部围绕 `[#键=值#]`→`window.__GC_DATA` 总线） → [references/gc-component-library/README.md](references/gc-component-library/README.md)
- MUV/3D 卡与舞台能力台（**新页专用**：真机实测外链通道、模型 .js 包装唯一通道、体积/性能预算、能力台骨架卡架构、与同层卡流派的差异取舍） → [references/muv-3d-stage.md](references/muv-3d-stage.md)

## 演进原则：多路线并存，择优吸收

MMD 写卡不是一条路走到黑——同层卡、能力台、GC 组件、官方作者区等流派同时存在、各自演进。吸收外来成果（别人的卡、教程、实测结论）时：

1. **先验证再吸收**：validate 跑一遍、红线关键词 grep 一遍，分清"真机实测事实"和"作者推测"，只收前者；事实冲突时以 tavern-mmd 源码逆向+探针为最终裁判。
2. **对比不替换**：新流派与现有流派冲突时，写清对照表（哪个场景用哪套、为什么），不轻易废掉旧路线——工具卡和内容卡的纪律本来就不一样。
3. **沉淀三件套**：每次吸收 = 新参考文档（或旧文档增量）+ 样本入 `assets\community-samples\` + SKILL.md 指针更新，缺一不可；版本号、来源、实测机型环境必须留痕。

## 一句话原则

规则文档是底线，实测优先于手册，交付前必过 validate + 预览 + 关键词 grep 验证。
