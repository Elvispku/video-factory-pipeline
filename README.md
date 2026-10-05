# video-factory-pipeline

视频内容工厂 · 一条龙出片 skill。给智能体（ZCode / Codex / Claude Code 等）装上之后，丢一条抖音链接（或本地视频）就能产出原创 9:16 成片：来源分析 → 原创脚本 → 情感化配音 → 内容驱动动效（深圳地图选区 / 地铁线路 / 三步回顾）→ S1/S2/S3 门禁 → 发布包（final.mp4 + SRT + manifest）。

## 安装（命令行一条）

```bash
git clone https://github.com/pensongchs/video-factory-pipeline.git ~/.zcode/skills/video-factory-pipeline
```

其他智能体的技能目录：

| 智能体 | 安装位置 |
| --- | --- |
| ZCode | `~/.zcode/skills/video-factory-pipeline` |
| Codex | `~/.codex/skills/video-factory-pipeline`（或在项目 AGENTS.md 引用 SKILL.md） |
| Claude Code | `~/.claude/skills/video-factory-pipeline` |
| 任意能读文件+执行命令的智能体 | 把 SKILL.md 全文放进它的系统提示 / 项目说明 |

## 首次配置（一次性，必做）

```bash
cd ~/.zcode/skills/video-factory-pipeline
cp config.example.json config.json
# 编辑 config.json：projectRoot 指向视频工厂项目根目录，deliverDir 指向成片交付目录
```

## 前置条件（本 skill 是"操作手冊"，产线本体必须已就位）

本 skill 驱动的是本地视频工厂项目，运行机器上需要：

1. **视频工厂项目**（含 `factory.config.json`、`scripts/`、`remotion-engine/`、数据合同三件套），见根目录 `factory.config.json` 结构
2. Node.js ≥ 20、Python ≥ 3.10、ffmpeg + ffprobe、Git
3. Python 依赖：`pip install edge-tts faster-whisper rapidocr-onnxruntime pillow`
4. Remotion 引擎：项目内 `remotion-engine/` 执行 `npm install`
5. 字幕 skill（可选但推荐）：[generate-timestamped-subtitles](https://github.com/pensongchs/shengcheng-shijianma-zimu-skill)（whisper.cpp + small 模型 ~466MB）
6. 分镜模板：[storyboard-visual-style-template-3.0](https://github.com/pensongchs/storyboard-visual-style-template-3.0)

装好后跑 `npm run doctor`，全绿即产线就绪。

## 用法

对智能体说（触发词）：

- 「跑一条视频：`<抖音分享链接>`」
- 「用这条参考视频出一片，房产口吻，60 秒左右」

流程：来源分析（新来源 ~8 分钟，同来源永久复用）→ Agent 写原创洞察 → Agent 写脚本分镜 → 情感化配音 + 内容动效合成 → 三道门禁 → 交付 `final.mp4 + manifest.json + captions.srt`。

**耗时基准（实测）**：新来源 ~30 分钟端到端，复用来源 ~20 分钟；纯机器 ~14 分钟。

## 硬规则（skill 内已写死，摘要）

- 章节只提炼洞察不复制原句（≥12 字连续重合即拒绝）
- 素材零重复：一照一镜；字幕/胶囊必须在核心画面安全区内
- 动效与口播语义一一对应（地图选区/地铁线路/三步回顾/排序条带），禁止装饰性动效
- 门禁 FAIL 必须修输入重跑；ON_HOLD 如实报告，不假成功
