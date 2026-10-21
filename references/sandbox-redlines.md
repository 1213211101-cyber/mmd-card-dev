# 新页沙盒红线与硬上限（v2.4 提炼）

权威来源：`references/魅魔岛状态栏开发通用安全规则_v2.4.md`（本仓库）+ tavern-mmd 仓库（[github.com/yofengi/tavern-mmd](https://github.com/yofengi/tavern-mmd)）内 `skills/tavern-mmd/references/platforms/mmd-sandbox.md`（源码逆向+三轮真机探针，含全部官方手册勘误）。

## 官方手册已被证伪的点（冲突时以 tavern-mmd 为准）

- ready 事件**不补发**（官方手册说补发，错）
- stage.el() 可能为 null，el() 恒有 DIV 别判空
- z-index 分段：3500-7999 可用，一千段位压顶栏
- beginning 真上限 4000（官方校验写 10240，错）
- 官方手册的 4 空格代码块、外链白名单、"只有 onclick" 等描述均有勘误
- 宿主页弹窗（模型设置等 10 个）卡片 CSS 打不到；iframe 内浮层能吃 `--chat-modal-*` 变量

## 事件与生命周期

- mount / done 事件**有补发**（订阅晚了也能收到）
- ready 最后到，**不补发**——必须提前订阅
- 脚本只跑一次：顶层代码别碰 DOM，进 DOM 操作必须挂事件回调
- 幂等哨兵：回调入口先检查自己是否已执行过（平台可能重复触发）
- send 没完别再 send，BUSY 不是限频信号
- 舞台开关看 `visible()`，不要自己猜状态

## 存储

- `save.remove` 是坏死的，删除用 `save.set(key, null)`
- `save.get` 必须 try/catch
- 共享状态挂 window，各规则作用域各自独立跑
- 存档用 save，缓存用 cache；舞台常住，气泡暂存

## CSS / DOM

- 自建 `data-*` 属性会被平台净化删除，别拿它当状态载体
- 净化会转义 `]>`，写进属性里的该序列会删属性
- **反引号 ` 才是爆代码真凶**（模板字符串嵌套时），不是引号；旧页才是引号撞层必爆
- 跨源 iframe 改不动父页；写给宿主页弹窗的 CSS 等于白写
- 换肤变量 29 个，选择器叠 `data-theme`；只有换背景才用 `!important`
- （旧页教训）空白条真因是 pre-line：容器 `white-space:normal` 一行治本
- flex 容器滚动区必须 `min-height:0`，否则溢出内容被裁剪

## 匹配式预算（正则替换）

- findRegex ≤1000、replaceString ≤20000 是硬上限
- 空串禁令：replaceString 不能产出空串
- 单条规则体积余量要在加代码前量好（剥注释后算）

## 排错速查（症状 → 首查项）

完整表在 v2.4 文档第三部分。高频项：

| 症状 | 首查 |
|---|---|
| 整次保存无声失败 | 各字段是否超上限（尤其 replaceString 20000） |
| 脚本不执行 | 顶层是否碰了 DOM / 是否缺幂等哨兵 / chatVersion 是否写了 |
| 样式不生效 | data-* 被净化？`]>` 被转义？选择器是否依赖宿主类名（沙盒不成立） |
| 状态丢失 | 是否用了 save.remove（改用 set null）/ save.get 是否裸调 |
| 主题错乱 | 变量名是否 29 变量表内 / 是否误用 !important / 多层 CSS 是否休眠激活 |
| 三次修不好 | 停止局部修补，全面重查（v2.4 顺口溜："三次不好全面查"） |

## 顺口溜（v2.4 附录一，速背）

```
共享状态挂window，各规则作用域各自跑
send没完别再send，BUSY不是限频
save删除用set null（remove是坏的），save.get要try/catch
舞台开关看visible()，el()恒有DIV别判空
z-index三五零零起（3500-7999），一千段位压顶栏
换肤变量二十九，选择器叠data-theme；换背景才用important
跨源iframe改不动父页，宿主弹窗写了白写
选项先填舞台框，确认才发不直连
（旧页）引号撞层必爆码，标记字面别处现交叉瘫
空白条真因pre-line，容器white-space:normal一行治本
存档save缓存cache，舞台常住气泡暂
两万拆链四千开场白，三次不好全面查
```

## 制作工作流

**新页**：定字段 → 写人设（含输出格式，标记用方括号）→ 组规则（CSS/UI/脚本三条起手）→ `validate.py --platform mmdsandbox` 0 错 → 本地仿真 chat + thin-preview 两 profile → 桌面/窄屏/横屏 GUI 截图 →（授权后）实机 `?sdkDebug=1` 验证（改动必须点「保存编辑」预览才看得到）。

**旧页**：需求 → 五级分类 → 选示例改造 → 改 CSS/引擎 → 写模型侧规则 → 脚本序列化打包 → validate.py 0 错 → build-preview 三面板+全景 → 实机验证。
