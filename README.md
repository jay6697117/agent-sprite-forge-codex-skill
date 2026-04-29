# Agent Sprite Forge Codex Skills

这是一个给 **Codex Skills** 使用的 2D 游戏美术生成工具包。

它包含两个本地 skill：

- `generate2dsprite`：生成并后处理 2D 精灵、角色、怪物、技能特效、投射物、道具和动画序列帧。
- `generate2dmap`：生成并组织 2D 游戏地图、分层地图、地图道具、碰撞数据、预览图和地图元数据。

核心思路很简单：

1. Codex 根据用户自然语言需求规划资源。
2. Codex 使用内置图片生成能力生成原始图片。
3. Python 脚本只做确定性的后处理：去洋红背景、切帧、对齐、导出透明 PNG/GIF、抽取地图道具、合成预览图。

本仓库不是传统 Python 包，也不是完整游戏工程；它主要提供 `.codex/skills` 目录给 Codex 读取。

## 项目结构

```text
.
├── .codex/
│   └── skills/
│       ├── generate2dsprite/
│       │   ├── SKILL.md
│       │   ├── references/
│       │   │   ├── modes.md
│       │   │   └── prompt-rules.md
│       │   ├── agents/
│       │   │   └── openai.yaml
│       │   └── scripts/
│       │       └── generate2dsprite.py
│       └── generate2dmap/
│           ├── SKILL.md
│           ├── references/
│           │   ├── layered-map-contract.md
│           │   ├── map-strategies.md
│           │   └── prop-pack-contract.md
│           ├── agents/
│           │   └── openai.yaml
│           └── scripts/
│               ├── compose_layered_preview.py
│               └── extract_prop_pack.py
├── requirements.txt
├── LICENSE
└── README.md
```

## 环境要求

需要：

- Python 3.10 或更高版本
- `Pillow`
- `numpy`
- 能读取本项目 `.codex/skills` 的 Codex 环境
- Codex 运行环境中可用的图片生成能力

安装 Python 依赖：

```bash
cd "./agent-sprite-forge-codex-skill"
python3 -m pip install -r requirements.txt
```

依赖文件内容很小：

```text
numpy>=1.26
Pillow>=10.0
```

## 安装 / 加载 Skill

### 方式 1：作为项目级 Codex Skills 使用

进入本项目目录后启动 Codex，让 Codex 读取项目内的 `.codex/skills`：

```bash
cd "./agent-sprite-forge-codex-skill"
codex
```

如果你的 Codex 环境支持项目级 `.codex/skills`，这样就够了。

### 方式 2：链接到全局 Codex Skills 目录

如果你的 Codex 版本只读取全局 skills，可以把两个 skill 链接到全局目录。

示例：

```bash
cd "./agent-sprite-forge-codex-skill"
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
ln -s "$PWD/.codex/skills/generate2dsprite" "${CODEX_HOME:-$HOME/.codex}/skills/generate2dsprite"
ln -s "$PWD/.codex/skills/generate2dmap" "${CODEX_HOME:-$HOME/.codex}/skills/generate2dmap"
```

如果全局目录里已经存在同名 skill，请先确认不要覆盖你自己的版本。

## 使用方式

在 Codex 对话里直接描述你要的资源即可。可以显式点名 skill，也可以让 Codex 自动判断。

### 生成 Sprite 示例

```text
使用 generate2dsprite 生成一个俯视角像素风火焰史莱姆，4 帧 idle 动画，透明背景。
```

```text
使用 generate2dsprite 生成一个 1x4 的蓝色魔法飞弹 projectile 动画，要求循环流畅。
```

```text
使用 generate2dsprite 生成一个 healer NPC，俯视角 16-bit RPG 风格，单张透明角色图。
```

### 生成 Map 示例

```text
使用 generate2dmap 生成一个俯视角森林小镇地图，有道路、草地、房子、路灯、碰撞区域和出生点。
```

```text
使用 generate2dmap 生成一个 clean HD 风格的 RPG shrine map，要求 base map、props、collision JSON 和 layered preview。
```

```text
使用 generate2dmap 生成一个 3x3 森林道具包，包括石头、灌木、木箱、木牌、蘑菇和草丛。
```

## generate2dsprite 做什么

`generate2dsprite` 用于生成独立 2D 游戏资源。

适合：

- 玩家角色
- NPC
- 怪物 / boss / 召唤物
- 技能施法动画
- projectile / impact / explosion 特效
- 地图道具
- 透明序列帧
- 透明 sprite sheet
- GIF 预览

常见输出：

```text
raw-sheet.png
raw-sheet-clean.png
sheet-transparent.png
frames/*.png
animation.gif
prompt-used.txt
pipeline-meta.json
```

对于 4 方向玩家行走表，还可能输出：

```text
direction strips
walk-down.gif
walk-left.gif
walk-right.gif
walk-up.gif
```

### Sprite 后处理脚本

脚本位置：

```text
.codex/skills/generate2dsprite/scripts/generate2dsprite.py
```

查看支持选项：

```bash
python3 .codex/skills/generate2dsprite/scripts/generate2dsprite.py list-options
```

处理一张已经生成好的原始 sprite sheet：

```bash
python3 .codex/skills/generate2dsprite/scripts/generate2dsprite.py process \
  --input raw-sheet.png \
  --target asset \
  --mode idle \
  --rows 2 \
  --cols 2 \
  --output-dir outputs/sprite-demo \
  --fit-scale 0.85 \
  --align center \
  --shared-scale \
  --component-mode all \
  --threshold 100 \
  --edge-threshold 150
```

重要参数：

- `--input`：原始图片路径。
- `--target`：资源类型，可选 `asset`、`creature`、`npc`、`player`。
- `--mode`：动作或模式，例如 `single`、`idle`、`walk`、`projectile`、`impact`。
- `--rows` / `--cols`：序列帧网格行列数。
- `--output-dir`：输出目录。
- `--fit-scale`：主体放入 cell 后的缩放比例。
- `--align`：对齐方式，可选 `center`、`bottom`、`feet`。
- `--shared-scale`：多帧共用缩放比例，避免每帧大小漂移。
- `--component-mode largest`：只保留最大连通主体，适合清理边缘碎片或无意义火花。
- `--reject-edge-touch`：发现主体碰到边缘时直接拒绝，适合严格质检。

## generate2dmap 做什么

`generate2dmap` 用于生成生产导向的 2D 游戏地图资源。

它会按需求选择地图管线：

- `baked_raster`：一张完整背景图，适合战斗背景、菜单背景、固定场景。
- `layered_raster`：地面 base map + 独立 props + 碰撞/区域 JSON + 合成预览。
- `tilemap`：瓦片图 + 地图数据，适合已有 tilemap 编辑器或引擎流程。
- `layered_tilemap`：多层瓦片地图。
- `parallax_layers`：横版或滚动背景分层。

常见输出：

```text
assets/map/<name>-base.png
assets/map/<name>-base.prompt.txt
assets/map/<name>-dressed-reference.png
assets/props/<prop>/prop.png
data/<name>-props.json
data/<name>-collision.json
data/<name>-zones.json
assets/map/<name>-layered-preview.png
```

### 地图道具包抽取脚本

脚本位置：

```text
.codex/skills/generate2dmap/scripts/extract_prop_pack.py
```

从 3x3 洋红背景道具表里抽取透明 prop：

```bash
python3 .codex/skills/generate2dmap/scripts/extract_prop_pack.py \
  --input assets/props/raw/forest-props-sheet.png \
  --rows 3 \
  --cols 3 \
  --labels mossy-rock,shrub,fallen-log,small-lantern,wooden-sign,flower-patch,stump,crate,grass-tuft \
  --output-dir assets/props \
  --manifest assets/props/forest-prop-pack.json \
  --component-mode largest \
  --component-padding 8 \
  --min-component-area 200 \
  --reject-edge-touch
```

输出示例：

```text
assets/props/mossy-rock/prop.png
assets/props/shrub/prop.png
assets/props/forest-prop-pack.json
```

### 分层地图预览合成脚本

脚本位置：

```text
.codex/skills/generate2dmap/scripts/compose_layered_preview.py
```

把 base map 和 props placement JSON 合成一张 QA 预览图：

```bash
python3 .codex/skills/generate2dmap/scripts/compose_layered_preview.py \
  --base assets/map/forest-town-base.png \
  --placements data/forest-town-props.json \
  --output assets/map/forest-town-layered-preview.png \
  --report outputs/forest-town-preview-report.json \
  --project-root .
```

placement JSON 示例：

```json
{
  "props": [
    {
      "id": "mossy-rock-1",
      "image": "assets/props/mossy-rock/prop.png",
      "x": 420,
      "y": 512,
      "w": 96,
      "h": 72,
      "sortY": 512,
      "layer": "props"
    }
  ]
}
```

## 关键规则

### 1. 原始图建议使用纯洋红背景

后处理脚本主要依赖这个背景色：

```text
#FF00FF
```

生成 sprite sheet、prop pack、单个透明道具时，都应该要求：

- 背景必须是纯 `#FF00FF`。
- 不要渐变。
- 不要阴影地面。
- 不要 UI。
- 不要文字。
- 不要边框。
- 物体不要碰到 cell 边缘。

### 2. 脚本只做后处理，不负责创意生成

正确流程：

```text
Codex 写图像生成 prompt
→ 图片生成工具生成 raw PNG
→ Python 脚本清理、切割、对齐、导出
```

不要把 Python 脚本当成最终美术生成器。脚本不会凭空画出高质量美术。

### 3. 分层地图不要把所有东西烤进一张图

如果地图需要：

- 角色走到树后面
- 道具可复用
- 道具有碰撞
- 道具有交互
- 需要 y-sort 遮挡

优先使用：

```text
base map + transparent props + placement JSON + collision JSON + preview
```

不要只交付一张完全烤死的背景图。

### 4. 道具包只适合小型静态物体

适合 prop pack：

- 石头
- 草丛
- 花
- 木箱
- 木桶
- 小灯
- 小木牌
- 地面装饰

不适合 prop pack：

- 建筑
- 大树
- 大门
- 桥
- 关键剧情物品
- 需要精确轮廓的大型物体
- 动画物体

大型或重要道具应该一张一张生成。

## 常见问题

### Codex 识别不到 skill 怎么办？

先确认目录存在：

```text
.codex/skills/generate2dsprite/SKILL.md
.codex/skills/generate2dmap/SKILL.md
```

然后尝试：

1. 退出 Codex。
2. 进入本项目根目录。
3. 重新启动 Codex。
4. 如果仍然识别不到，把两个 skill 链接到全局 Codex skills 目录。

### 生成结果背景没有变透明怎么办？

通常原因是原始图背景不是纯洋红，或者存在抗锯齿边缘。

处理方法：

- 重新生成图片，并明确要求 `100% solid flat #FF00FF magenta background`。
- 提高或调整 `--threshold` 和 `--edge-threshold`。
- 对道具包使用更严格的 prompt：物体必须在 cell 中央，不能碰边。

### 切出来的动画每帧大小不一致怎么办？

使用：

```bash
--shared-scale
```

并确保原始 prompt 要求：

```text
same bounding box and same pixel scale in every frame
```

### 道具或角色碰到格子边缘怎么办？

不要强行放行。推荐重新生成原图，并在 prompt 中加入：

```text
The entire subject must fit inside the central 50% to 60% of its cell with generous magenta margin on all four sides. Nothing may touch or cross a cell edge.
```

处理时可以加：

```bash
--reject-edge-touch
```

## Git 忽略规则

当前 `.gitignore` 会忽略：

```text
__pycache__/
*.pyc
.DS_Store
outputs/
.venv/
```

所以临时生成物建议放在 `outputs/` 里。

## License

MIT License. See [LICENSE](LICENSE).
