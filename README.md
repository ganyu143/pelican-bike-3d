# pelican-bike-3d（鹈鹕骑自行车 3D 页面）

日期：2026-10-11（2026-10-11 二次升级）· 状态：已上线 CDN

## 一句话

单文件 Three.js 场景：一只程序化建模的鹈鹕骑着 3 速自行车在运河堤岸上巡航，受力平衡决定速度，爬坡要换挡，路上有鱼筐和苍鹭要避让。

## 访问

- **页面（GitHub Pages CDN，Fastly 全球节点）**：https://ganyu143.github.io/pelican-bike-3d/
- 源仓库：https://github.com/ganyu143/pelican-bike-3d （public，分支 main，Pages 源 = main /）
- 原始文件（jsDelivr）：https://cdn.jsdelivr.net/gh/ganyu143/pelican-bike-3d@main/index.html — 实测返回 `content-type: text/plain; charset=utf-8` + `nosniff`，浏览器只显示源码不渲染，因此不能当页面链接用（jsDelivr 对 HTML 文件的固定策略）
- 本地：`index.html`（`python3 -m http.server` 打开；ES module + importmap 需要 http，不能直接 file:// 双击）

## 文件

| 文件 | 作用 |
|---|---|
| `index.html` | 全部代码（HTML + CSS + ES module），无外部模型/贴图资源 |
| `README.md` | 本文件 |
| `preview.png` | 线上页面截图（headless Chrome 实拍，2026-10-11 上午版本） |

依赖：`three@0.186.1`（jsDelivr importmap：`three` + `three/addons/`）、Google Fonts（Bricolage Grotesque + IBM Plex Mono）。

## 场景构成（全部代码生成，零素材）

- 鹈鹕：Lathe 身体 + Tube 脖颈 + 锥形喙/喉囊（鸣叫时张开）+ 分层羽毛翅膀 + 两骨 IK 腿（脚踩踏板）
- 自行车：轮圈/辐条/花鼓/碟刹、钻石车架（TubeGeometry）、曲柄+脚踏、链条（InstancedMesh 沿闭合曲线跑动）、车篮里的鱼、车铃、后挡泥板
- 环境：渐变天空球（自定义 shader，含太阳盘/辉光/星空）、沥青路面（canvas 贴图滚动）、运河水面、堤坡、芦苇（InstancedMesh）、路灯（夜间自发光）、树/远山/云（视差 + 循环回收）、浮尘粒子
- 光照：Hemisphere + 主平行光（阴影）+ 补光；四套调色板（黄昏/正午/清晨/夜）平滑插值切换
- 后处理：EffectComposer + UnrealBloomPass（默认关）+ OutputPass
- 音频：WebAudio 合成（风声随速度、轮响、BWAARK 鸣叫、车铃叮、撞击闷响），默认关闭，需用户手势启动

## 骑行动力学（2026-10-11 新增）

- 受力：驱动力 = 骑手功率 × 传动效率 ÷ 速度（受姿态最大功率与 0.28 mg 牵引上限约束）；阻力 = ½ρCdA v² + Crr·mg·cosθ + mg·sinθ；制动 = μmg
- 骑手控制器：前馈（维持当前速度所需功率）+ 比例项追目标速度
- 参数：鹈鹕 5.2 kg + 整车 11.5 kg，CdA 0.42 m²，Crr 0.0055，ρ 1.225 kg/m³，传动效率 95 %，轮径 0.35 m
- 档位：30/39/48 : 16T（1.88 / 2.44 / 3.00）。165 W 巡航在 3× 档约 29 km/h、72 rpm；换 1× 档同样速度要 109 rpm
- 路面：运河堤岸剖面 `1.15·sin(2πx/260) + 0.55·sin(2πx/96+1.1)`，最大坡度约 6 %；路面/地面/堤坡逐帧按剖面成形，车架按前后轮高度差俯仰
- 障碍：鱼筐与苍鹭在前方 200–520 m 生成，进入 14 m 内触发 `chooseEvadeAction()`，可选制动 / 绕开 / 跳起 / 按铃；判错会撞上（速度 −65 %、HUD 亮红灯、闷响）

## 交互

目标速度滑杆 0–45 km/h、姿态（巡航/翘头/冲刺/休息）、档位、光照四档、制动、跳起、辉光/声音/自动环绕开关、四个相机预设、PNG 截图、读数曲线 ↔ 数据表切换。
键盘：`空格`鸣叫、`↑↓`目标速度、`G`换挡、`S`制动、`J`跳、`1-4`视角、`B`辉光、`N`夜、`T`数据表、`P`截图、`R`复位、`H`帮助。
点击模型：点鹈鹕=鸣叫，点车铃=叮，点喉囊=鱼跳出来。

## 读数与图表

- 状态灯（巡航/爬坡/制动/避让/腾空/撞车/踏频建议）+ 速度、踏频、功率、坡度、里程、个人最佳、FPS
- 三联曲线（速度 km/h、踏频 rpm、路面剖面 m），共享时间轴 60 s 窗口，悬停十字线 + 数值提示，末端直接标值
- 数据表视图（最近 10 个采样：t / km/h / rpm / W / 坡度 %）
- 传动示意图（inline SVG）+ 规格表：踏频 × 齿比 × 轮周长 = 速度，实时换算
- 图表配色经 `dataviz` 校验脚本核验（light `#c2601c,#1baf7a` / dark `#d95926,#199e70`，CVD ΔE ≥ 9，浅色 aqua 触发对比度 WARN，已用末端直接标值 + 数据表作为补救）

## 主题与本地存储

- 浅色/深色双主题（`prefers-color-scheme` + `[data-theme]`）；浅色默认配正午场景，深色默认配黄昏场景
- 窄屏（≤900px）：控制面板与读数卡互斥展开，中间留给 3D 画面
- 姿态/光照/档位/个人最佳存 `localStorage`（键 `pelican-velo-v2`），读写失败不阻塞页面
- `prefers-reduced-motion` 关闭车身颠簸与加载动画

## 验证记录（2026-10-11，headless Chrome + chrome-devtools）

- 控制台 0 error；60 fps（headless 软件 WebGL）
- 物理自洽：3× 档 165 W → 28.6 km/h / 72 rpm；示意图换算与 HUD 一致
- 换挡 `G`：3× → 1× 后 26.9 km/h 需 109 rpm（状态灯提示该升档）
- 制动：2 s 内 30 → 1.4 km/h，功率归零，状态灯「制动中」
- 跳起 `J`：状态灯「腾空」约 0.6 s
- 障碍决策实测触发自动制动，避开碰撞；障碍通过后自动解除自动制动
- 数据表切换：10 行真实采样，曲线卡隐藏/恢复正常
- 布局：1440×900 浅色与深色、390×780 深色均截图核验；窄屏卡片不再重叠（修正过一次 900px 断点重叠）

## 待人工决定

`index.html` 内 `chooseEvadeAction()` 上方有 `TODO(human)` 块：当前默认策略是「制动距离够就刹、否则侧向空间够就绕、再不行跳、最后按铃」，可自行替换成更激进的策略（代价：撞上会掉 65 % 速度）。

## 回滚 / 更新

改 `index.html` → `git commit` → `git push` → Pages 约 30–60 秒生效（`gh api repos/ganyu143/pelican-bike-3d/pages` 看 status）；jsDelivr 缓存约 5 分钟刷新。
删除仓库：`gh repo delete ganyu143/pelican-bike-3d`（走确认）。
