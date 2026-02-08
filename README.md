# Eval Dataset Manager

> AIGC 多模态生成模型评估数据集管理平台

一个纯前端的评估数据集管理工具，用于系统化管理文生图（T2I）、文生视频（T2V）、图生视频（I2V）、图生图（I2I）、拼豆、I2LoRA 等 AI 生成任务的评估数据集。

## 特性

- **零部署**：单 HTML 文件，双击浏览器打开即用，无需安装任何依赖
- **6 种任务类型**：T2I / T2V / I2V / I2I / 拼豆 / I2LoRA，每种类型有独立的参数模板
- **多维度标签**：风格（10类）、能力维度（7类）、场景（8类）、难度（3级）+ 自定义标签
- **媒体预览**：支持本地路径和远程 URL 的图片/视频预览
- **多维筛选**：按标签、难度、风格、能力维度交叉筛选，实时计数
- **导入导出**：JSON 导入（含 Legacy I2V/I2LoRA 格式适配）、CSV 导出（对接 Batch Task 推理平台）
- **自动保存**：localStorage 持久化，500ms 防抖，数据不丢失
- **扩展字段**：每个任务支持自定义 key-value 元数据

## 快速开始

```bash
# 方式 1：直接打开
open index.html

# 方式 2：本地服务器（推荐，避免本地文件跨域问题）
python3 -m http.server 8080 --directory .
# 然后访问 http://localhost:8080
```

## 架构定位

```
[本平台: 数据集管理] → [Batch Task: 批量推理] → [Argilla: 人工标注] → 统计分析
```

本平台作为评估流水线的上游，负责：
1. 收集和管理评估任务的输入数据（Prompt、媒体文件、参数）
2. 对评估任务进行多维度打标分类
3. 导出 CSV 供下游 Batch Task 推理平台消费

## 数据模型

### 任务类型及其参数

| 类型 | 核心字段 | 媒体字段 |
|------|---------|---------|
| T2I | prompt(双语), negative_prompt, width, height, seed, steps, cfg_scale | reference_images |
| T2V | prompt(双语), width, height, duration, fps, seed, motion_type | - |
| I2V | initial_prompt, intent_cn, width, height, duration, seed | first_frame, last_frame |
| I2I | prompt(双语), negative_prompt, strength, controlnet_type | input_image |
| 拼豆 | prompt, palette, size, style | input_image |
| I2LoRA | style_description, test_prompts[] | training_images[] |

### 标签维度

- **风格** (10): Anime, Realistic, Sci-Fi, Illustration, Character, Design, Fantasy, Photography, 3D, Abstract
- **能力** (7): Prompt遵循度, 美学质量, 一致性, 物理合理性, 创意表现, 技术质量, 可控性
- **场景** (8): 真实-人物/动物/自然/静物/城市, 动漫, 科幻, 奇幻
- **难度** (3): 低, 中, 高

## 导入格式

### 平台原生格式

```json
{
  "version": "1.0",
  "datasets": [
    {
      "name": "数据集名称",
      "taskType": "i2v",
      "tasks": [...]
    }
  ]
}
```

### Legacy I2V 格式（自动适配）

```json
{
  "tasks": [
    {
      "id": 1,
      "difficulty": "低",
      "category": "真实-人物",
      "intent_cn": "...",
      "initial_prompt": "...",
      "first_frame": { "en": "...", "cn": "..." },
      "last_frame": { "en": "...", "cn": "..." }
    }
  ]
}
```

### Legacy I2LoRA 格式（自动适配）

```json
{
  "style_id": "01_90s_anime",
  "style_description": "...",
  "content_variants": [...],
  "test_prompts": { "no_style": "..." }
}
```

## CSV 导出字段

导出 CSV 包含以下列：

```
eval_id, task_type, difficulty, style_categories, capability_focus, scene_type,
custom_tags, prompt_en, prompt_zh, negative_prompt, first_frame_url, last_frame_url,
input_image_url, initial_prompt, intent_cn, width, height, duration, fps, seed,
steps, cfg_scale, strength, model, style, motion_type, controlnet_type, notes
```

## 技术栈

- **Vue 3** (CDN) - 响应式框架
- **Tailwind CSS** (CDN) - 样式系统
- **localStorage** - 数据持久化

## License

MIT
