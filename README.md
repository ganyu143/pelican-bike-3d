# pelican-bike-3d（鹈鹕骑自行车 3D 页面）

日期：2026-10-11 · 状态：已上线 CDN

## 一句话

单文件 Three.js 场景：一只程序化建模的鹈鹕骑着自行车在黄昏运河边巡航，可交互、可换光照/姿态、可截图。

## 访问

- **页面（GitHub Pages CDN，Fastly 全球节点）**：https://ganyu143.github.io/pelican-bike-3d/
- 源仓库：https://github.com/ganyu143/pelican-bike-3d （public，分支 main，Pages 源 = main /）
- 原始文件（jsDelivr）：https://cdn.jsdelivr.net/gh/ganyu143/pelican-bike-3d@main/index.html — 实测返回 `content-type: text/plain; charset=utf-8` + `nosniff`，浏览器只显示源码不渲染，因此不能当页面链接用（jsDelivr 对 HTML 文件的固定策略）
- 本地：`index.html`（直接双击或 `python3 -m http.server` 打开均可）

## 文件

| 文件 | 作用 |
|---|---|
| `index.html` | 全部代码（HTML + CSS + ES module），无外部模型/贴图资源 |
| `README.md` | 本文件 |
| `preview.png` | 线上页面截图（headless Chrome 实拍） |

依赖：`three@0.186.1`（jsDelivr importmap：`three` + `three/addons/`）。

## 场景构成（全部代码生成，零素材）

- 鹈鹕：Lathe 身体 + Tube 脖颈 + 锥形喙/喉囊（会随鸣叫张开）+ 分层羽毛翅膀 + 两骨 IK 腿（脚踩踏板）
- 自行车：轮圈/辐条/花鼓/碟刹、钻石车架（TubeGeometry）、曲柄+脚踏、链条（InstancedMesh 沿闭合曲线跑动）、车篮里的鱼、车铃、后挡泥板
- 环境：渐变天空球（自定义 shader，含太阳盘/辉光/星空）、沥青路面（canvas 贴图滚动）、运河水面、芦苇（InstancedMesh）、路灯（夜间自发光）、树/远山/云（视差 + 循环回收）、浮尘粒子
- 光照：Hemisphere + 主平行光（阴影）+ 补光；四套调色板（黄昏/正午/清晨/夜）平滑插值切换
- 后处理：EffectComposer + UnrealBloomPass（默认关）+ OutputPass
- 音频：WebAudio 合成（风声随速度、轮响、BWAARK 鸣叫、车铃叮），默认关闭，需用户手势启动

## 交互

速度滑杆 0–60 km/h、姿态（巡航/翘头/冲刺/休息）、光照四档、辉光/声音/自动环绕开关、四个相机预设、PNG 截图。
键盘：`空格` 鸣叫、`↑↓` 速度、`1-4` 视角、`B` 辉光、`N` 夜、`P` 截图、`R` 复位、`H` 帮助。
点击模型：点鹈鹕=鸣叫，点车铃=叮，点喉囊=鱼跳出来。

## 验证记录（2026-10-11，headless Chrome + chrome-devtools）

- 控制台：0 error（仅 headless 软件 WebGL 环境警告）
- 截图确认：黄昏侧视、夜+翘头+辉光 两种状态均正常渲染，HUD 数值与物理量一致（30 km/h → 踏频 72 rpm）
- 像素采样确认夜空为深蓝（调色板插值收敛）
- 决策依据（laya classifier，answer_confidence）：黄昏暖色调 0.975 / 默认关辉光 0.880 / 默认关声音 0.957 / HUD 数值有用 0.961

## 回滚 / 更新

改 `index.html` → `git commit` → `git push` → Pages 约 30–60 秒生效（`gh api repos/ganyu143/pelican-bike-3d/pages` 看 status）；jsDelivr 缓存约 5 分钟刷新。
删除仓库：`gh repo delete ganyu143/pelican-bike-3d`（走确认）。
