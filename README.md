# video-factory-pipeline

视频内容工厂 · 一条龙出片 skill。给智能体（ZCode / Codex / Claude Code 等）装上之后，丢一条抖音链接（或本地视频）就能产出原创 9:16 成片：来源分析 → 原创脚本 → 情感化配音 → 内容驱动动效（深圳地图选区 / 地铁线路 / 三步回顾）→ S1/S2/S3 门禁 → 发布包（final.mp4 + SRT + manifest）。

## 安装（命令行一条）

```bash
git clone https://github.com/Elvispku/video-factory-pipeline.git ~/.zcode/skills/video-factory-pipeline
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

## 前置条件（重要：本仓库只是操作手册，产线本体在私库）

本 skill 驱动的是**私有仓库 [video-factory](https://github.com/Elvispku/video-factory)**（完整产线：引擎/脚本/合同/数据资产）。首次安装把它一起克隆：

```bash
git clone https://github.com/Elvispku/video-factory.git   # 私库，需要访问权
```

- 本机（Elvispku 的 Windows 机器）已存凭据，克隆零配置
- 其他机器/账号：请仓库所有者添加协作者，或提供访问令牌（Settings → Collaborators）

其余依赖：Node.js ≥ 20、Python ≥ 3.10、ffmpeg、Git；Python 依赖 `pip install -r requirements.txt`（在项目内）；Remotion 引擎 `cd remotion-engine && npm install`；字幕 skill [generate-timestamped-subtitles](https://github.com/pensongchs/shengcheng-shijianma-zimu-skill)；分镜模板 [storyboard-visual-style-template-3.0](https://github.com/pensongchs/storyboard-visual-style-template-3.0)。逐条命令见项目内 `SETUP.md`。装好后 `npm run doctor` 全绿即产线就绪。

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
