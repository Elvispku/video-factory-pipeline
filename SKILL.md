---
name: video-factory-pipeline
description: 从抖音链接或本地视频产出原创 9:16 抖音成片：来源分析→原创脚本→情感化配音→地图/地铁等内容驱动动效→S1/S2/S3 门禁→发布包。当用户发来视频链接要求做成片、要求"跑一条视频 / 出片 / 视频工厂跑一条"，或要求统计/优化产线耗时。新来源约 30 分钟，复用来源约 20 分钟。
---

# 视频工厂 · 一条龙出片

本 skill 是**执行器**：把一条链接（或本地视频）变成一支 READY_TO_PUBLISH 的原创 9:16 成片。
所有结构性规则的唯一来源是项目合同（`factory.config.json` / `video-factory-layout-contract.json` / `data-contracts.schema.json`），本文件只做流程编排与创作纪律，**不得替代合同**。

项目根目录读 `config.json` 的 `projectRoot`（下称 `$R`）。所有命令在 `$R` 下执行。

## 首次安装（一次性）

1. `cp config.example.json config.json`，填入本机 `projectRoot`（视频工厂项目根）与 `deliverDir`（成片交付目录）
2. 产线前置：项目内 `npm run doctor` 全绿（缺 Python 依赖装 `edge-tts faster-whisper rapidocr-onnxruntime pillow`；缺 Remotion 依赖在 `remotion-engine/` 里 `npm install`）
3. 授权检查：`factory.config.json → services.tts.rights.status` 必须为 `authorized` 才可能到 READY_TO_PUBLISH（未确认时项目只会到 ON_HOLD）

## 耗时基准（实测，用于向用户交代预期）

| 阶段 | 耗时 |
| --- | --- |
| 来源分析链（新来源一次性） | ~8 分钟（ASR 占 6 分钟；同来源永久复用） |
| plan（TTS 试算镜头边界） | ~1 分钟 |
| 生产链 compose→run×2（TTS/生图/S1/渲染/S2/S3/发布包） | ~14 分钟 |
| Agent 创作输入（review + plan.json） | ~10-15 分钟 |
| **合计** | **新来源 ~30 分钟；复用来源 ~20 分钟** |

## 第 0 步：前置检查（每期必做，失败即停）

```bash
cd "$R"
npm run doctor        # 必须全绿；ENVIRONMENT_BLOCKED 不得绕过
npm run factory -- status   # 确认无残留未完成任务
```

## 第 1 步：来源登记与分析（新来源才做；复用来源直接跳到第 2 步）

1. 写来源清单 `data/tmp/sources-<slug>.json`：

```json
{ "sources": [{
    "id": "douyin-<awemeId>", "url": "<分享链接>",
    "localPath": "data/media/raw/<id>.mp4",
    "title": "<视频标题>", "sourceType": "video",
    "rights": { "status": "unknown", "basis": "抖音公开分享链接，用户授权作参考素材；仅分析，不进入发布素材库" } }] }
```

2. `npm run factory -- intake data/tmp/sources-<slug>.json`
3. `npm run factory -- run`（反复执行直到"没有依赖已满足的可执行任务"）
4. 验收：`data/chapters/` 出现该来源的章节，`analysisState=completed`。

**下载失败处理**：download 任务 blocked/failed 说明抖音适配器再次失效。回退路径：受控浏览器打开分享页 → 从播放器取 CDN 直链 → 签名过期前下载到 `data/media/raw/<id>.mp4` → 来源改用 `localPath` 重新 intake。全程如实登记，不得伪造下载成功。

## 第 2 步：审核关卡（Agent 执笔，"章节只提炼洞察、不复制原句"的强制关卡）

1. 读该来源的章节（`data/chapters/chp_*.json` 中 `sourceId` 匹配、`contentState=restricted` 的）。
2. 逐章写**原创观点摘要**：≥8 字；与 `rawTranscript` 的最长连续重合必须 <12 字（管线自动查重，超了拒绝）。
3. 写 `data/tmp/review-<slug>.json`：

```json
{ "reviewedBy": "<可追溯的执行者标识>",
  "chapters": [{ "id": "chp_...", "originalSummary": "<原创洞察>",
    "contentState": "insight-only", "factStatus": "opinion|unverified",
    "chapterType": "hook|framework|evidence|risk|conclusion", "topicTags": [], "audience": [] }] }
```

4. `npm run factory -- review data/tmp/review-<slug>.json`，验收：回填数=章节数、0 rejected。
5. 只挑实质章节审核即可（碎片章节保持 restricted，不用于创作）。

## 第 3 步：创作立案（Agent 执笔，全流程最重的创作步）

写 `data/tmp/plan-<slug>.json`，结构：`narration`（完整口播）+ `shots[]`（按画面拆镜）+ `topic` + `script` + `voices`。

**脚本纪律（合同级，不得绕过）：**
- 主题 `grounding="chapters"`，`sourceChapterIds` 只准引用 `insight-only` 章节
- 市场/政策/价格判断一律 `factClaims.verification = "opinion" | "unverified"`；禁"必涨/保值"类承诺
- 不复用参考视频原句、镜头、角色、品牌

**narration 与分镜（管线的硬校验）：**
- narration 300-340 字 ≈ 70-82 秒；`shots[].narrationSpan` 按顺序拼接必须与 narration **逐字一致**（无缝隙无重叠），否则 plan 拒绝
- 每镜必须有 `visual/intent/sourceState/targetState/carrier/actionVerb/handoff/continuityLink`；相邻镜共享叙事连续性理由

**mgKind 动效语义指派（动效必须与口播内容对应，禁止装饰性动效）：**

| 口播讲什么 | mgKind | 画面 |
| --- | --- | --- |
| 各区/板块选择 | `map-gold` | 深圳行政区地图，目标区金色高亮脉冲+"重点板块"标记 |
| 风险/排除区域 | `map-red` | 同图红色状态，风险区红色脉冲+"边缘盘"标记 |
| 交通/通勤/地铁 | `metro` | 14/16 号线线路生长、车站点亮、列车光点 |
| 排序/标准/维度 | `bars-right` | 条带靠右升序，左侧承载排序文字卡 |
| 结论/回顾/步骤 | `recap` | 三步节点连线依次点亮 |
| 有匹配实拍照片 | 照片场景 | `photo` 字段指派（见下表） |

**照片指派（`remotion-engine/public/photos/`，同期零重复，一照一镜）：**
`hook-community.jpg`（城市小区，高清可上核心）、`fantasy-livingroom.jpg`（室内，高清可上核心）、`district-env.jpg`（社区环境，中清可上核心）、`commute.jpg`（通勤，**低清只做 ambientPhoto**）、`renovated.jpg`（装修，**低清只做 ambientPhoto**）。

**mg 卡片几何（硬校验）：**
- 显式 `left/top/width/height`（管线拒绝缺省）
- 全部落在核心安全区 **x∈[108,864]、y∈[690,1230]**
- 字幕占核心画面底部（y≈1050-1214，居中），卡片避开该区域
- 胶囊/文字卡字号 30-38（要大，突出重点）
- 每镜字段：`{kind:'note'|'text'|'info'|'choice'|'disc'|'line', label, sub?, tone:'gold|red|green|ivory', motion:'slide|rise|zoom|reveal', left, top, width, height, size?, start, end?}`（start/end 为镜内局部帧）

写好后：`npm run factory -- plan data/tmp/plan-<slug>.json` → 产出 `*.creation.json`。
失败最常见原因：narrationSpan 拼接不一致（输出会指出拼接字数差）。

## 第 4 步：生产（约 14 分钟机器时间）

```bash
npm run factory -- compose data/tmp/plan-<slug>.creation.json
npm run factory -- run    # 跑两遍：第一遍到 gate-s3，第二遍收尾 publish-package
npm run factory -- run
```

渲染引擎默认 Remotion（`factory.config.json → render.engine`）。**等待渲染时不要并行启动第二个 run**（会竞态污染任务状态）。

## 第 5 步：验收与交付（不验收不算完成）

1. 读发布包 manifest：`projectStatus` 必须为 `READY_TO_PUBLISH` 且 `publishable=true`；`ON_HOLD` 时如实向用户报告 blockedReasons，不得宣称可发布。
2. 读三个门禁报告（`data/gates/<id>.json`）确认 S1/S2/S3 全 PASS；有 FAIL 读明细修输入后重跑对应任务（重置任务：把 job JSON 的 `status` 改回 `queued` 并清空 `error/finishedAt`）。
3. `ffmpeg` 抽 3-5 帧目检：核心画面居中、字幕/卡片在画面内且不重叠、地图/地铁文字未出界。
4. 交付三件套到 `deliverDir`：`final.mp4` + `manifest.json` + `captions.srt`，向用户报告门禁结果与抽帧结论。

**常见失败与修法：**

| 报错 | 原因 | 修法 |
| --- | --- | --- |
| S1「越出安全区」 | mg 卡片坐标出界 | 改 plan 输入坐标，重跑 plan 起的链 |
| S3「未命中任何已注册组件矩形」 | 文字出安全区，或图形场景漏登记图形文字区 | 同上；图形场景由管线自动登记 |
| S3 黑帧 | 场景交叉溶解不重叠 / 画面过暗 / 片尾黑尾 | 溶解必须重叠（FADE 内外各 10 帧）；总帧数与配音精确对齐 |
| render `spawn EINVAL` | Windows 下 .cmd 直启 | 已改为 node 直调 `@remotion/cli/remotion-cli.js`，勿改回 npx |
| Remotion 渲染 2 秒就结束 | props 未包裹 `{props: ...}` 或路径含空格被截断 | props 必须写 `{props}` 包裹结构并拷到无空格临时路径 |
| tts NoAudioReceived | edge-tts 限流 | 适配器已带退避重试；仍失败等 1 分钟重跑 |

## 硬规则速查（违反即返工）

1. 环境门禁不绿，不启动任何任务；服务未配置不得暗中下载。
2. 章节只有 `insight-only` 能进创作；摘要与原文 ≥12 字连续重合即拒绝。
3. 素材零重复：一张照片/一段实拍同期只允许出现在一个连续语义段。
4. 字幕与卡片必须在核心安全区内；核心层素材本身不得烘焙文字（S3 以注册矩形匹配把关）。
5. 动效必须与口播语义对应（见 mgKind 表）；禁止装饰性无关联动效。
6. 门禁 FAIL 必须修输入重跑，禁止绕过；`ON_HOLD` 必须如实报告原因。
7. edge-tts 授权已由用户确认为 authorized（config `services.tts.rights`）；改授权状态必须经用户确认。

## 产物规格（对照验收）

1080×1920 / 9:16 / 30fps / h264+aac；核心画面 960×540 居中 (60,690) `contain`；
安全区 x=108-864、y=288-1540；音画同长；发布包含 manifest（门禁 ID + 授权追溯）+ SRT + Caption JSON。
