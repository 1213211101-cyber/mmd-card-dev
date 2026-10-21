# mmd-card-dev — 魅魔岛（MMD）AI 文字角色卡开发总纲

一个 Kimi skill：让 Kimi 成为魅魔岛（MMD）文字角色卡的开发助手。涵盖三代平台分层（新页沙盒 / 当前版旧页 / 已退役旧版）、硬上限红线、路线选择、构建交付纪律、关系网与文风纪律、社区成品样本与 GC 组件库。

## 内容结构

```
SKILL.md            入口与速查（frontmatter 内含触发场景）
references/
  魅魔岛状态栏开发通用安全规则_v2.4.md   唯一权威总文档（1100 行，写卡前必读）
  workflow.md                            构建链/文风/关系网/交付纪律、DIS 项目教训
  sandbox-redlines.md                    新页沙盒红线详解与排错速查表
  same-layer-card.md                     同层全屏卡制作规范（新页实战流派）
  option-statusbar-lite.md               轻量选项栏/配角状态栏协议（旧页正则派）
  opening-protocol.md                    开场白生成协议
  muv-3d-stage.md                        MUV/3D 卡与舞台能力台（新页专用）
  gc-component-library/                  GC 组件库（社区开源教程六份，旧页专用）
assets/
  community-samples/                     三份社区成品卡样本（鲨鱼卡/补梦卡/3D能力台）
```

## 安装（二选一）

**方式一（推荐）：把本仓库的 GitHub 链接直接发给 Kimi**，说一句"安装这个 skill"，Kimi 会自动转换、注册并在插件页一键安装。

**方式二（手动）**：

```bash
git clone <本仓库地址>
```

然后把克隆下来的 `mmd-card-dev` 目录整个放进 Kimi 的 skills 目录（Windows：`%APPDATA%\kimi-desktop\daimon-share\daimon\skills\`），重开会话即可。

## 配套工具链

静态审核（validate）、三面板预览、SBK 基座、沙盒仿真器与平台权威规范位于配套开源仓库 [tavern-mmd](https://github.com/yofengi/tavern-mmd)（`skills/tavern-mmd/` 下），建议一并克隆。SKILL.md 的"资源指针"表列出了各工具在其中的具体路径。

## 使用

安装后直接对 Kimi 提需求即可，例如"帮我写一张新页沙盒的角色卡""给这张卡加个状态栏"，skill 会按代际与路线自动加载对应纪律。

## License

MIT（见 LICENSE 文件）。`assets/community-samples/` 内的社区成品样本版权归原作者所有，已获开源分享，仅作学习参考。
