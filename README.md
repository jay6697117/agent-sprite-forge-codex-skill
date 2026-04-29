# Agent Sprite Forge Codex Skills

## 依赖

Python 依赖：

```bash
python3 -m pip install -r requirements.txt
```

图片生成依赖：

```text
.claude/skills/codex-gateway-imagegen
```

也就是说，Claude Code 中必须能使用 `codex-gateway-imagegen` skill。该 skill 负责调用 gateway 生成图片，并把图片保存为本地文件。

## 使用方式：Claude Code 本地 skill 模式

进入当前目录：

```bash
cd "/Users/zhangjinhui/Desktop/agent-sprite-forge-claude-plugin"
```

安装依赖：

```bash
python3 -m pip install -r requirements.txt
```

启动 Claude Code：

```bash
claude
```

然后在 Claude Code 里直接使用本地 skill：

```text
/generate2dsprite
/generate2dmap
```

示例：

```text
/generate2dsprite 生成一个俯视角像素风火焰史莱姆，4帧 idle 动画，透明背景
```

```text
/generate2dmap 生成一个俯视角像素风森林小镇地图，带道路、草地、房子和碰撞区域
```

如果 Claude Code 已经打开但识别不到新 skill，最稳妥的做法是退出后在当前目录重新运行 `claude`。

## Sprite 生成流程

1. Claude Code 根据用户需求规划 sprite 类型、动作、帧数、网格。
2. Claude Code 调用 `codex-gateway-imagegen` skill 生成 raw PNG。
3. `codex-gateway-imagegen` 返回或保存本地图片路径。
4. `generate2dsprite` skill 把这个本地图片路径传给：

```bash
python "${CLAUDE_SKILL_DIR}/scripts/generate2dsprite.py" process ...
```

5. Python 脚本做洋红背景清理、分帧、对齐、透明 sheet、GIF 和 QC metadata。

## Map 生成流程

1. Claude Code 选择地图 pipeline：baked raster、layered raster、tilemap 或 parallax。
2. 需要生成图片时，调用 `codex-gateway-imagegen` skill。
3. 对 prop pack，使用：

```bash
python "${CLAUDE_SKILL_DIR}/scripts/extract_prop_pack.py" ...
```

4. 对 layered map preview，使用：

```bash
python "${CLAUDE_SKILL_DIR}/scripts/compose_layered_preview.py" ...
```

## 注意事项

- 本地 skill 入口在 `.claude/skills/`。
- 实际 skill 源文件仍在 `skills/`，`.claude/skills/` 里用符号链接指向它们。
- 不复制原项目的 `agents/openai.yaml`，因为 Claude Code 不需要 Codex agent 配置。
- 不复制原项目的 `src/` showcase 图片，避免让适配目录变重。
- 不复制 `.DS_Store`、`__pycache__`、`outputs/`。
- 生成图片必须先落到本地文件，再交给 Python 后处理脚本。
