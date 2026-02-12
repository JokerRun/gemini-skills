---
name: gemini-image-generation
description: 使用此技能进行基于Gemini API的图像生成任务，包括文本生成图像、图像编辑、多轮编辑等。涵盖Nano Banana模型（gemini-3-pro-image-preview和gemini-2.5-flash-image）的使用、配置参数、最佳实践和完整的API参考。
---

# Gemini 图像生成技能

## 概述

Gemini API 提供了强大的原生图像生成能力，通过 **Nano Banana** 系列模型，可以实现：
- **文本生成图像（Text-to-Image）** - 根据文本描述生成高质量图像
- **图像编辑** - 使用自然语言指令对图像进行编辑和转换
- **多轮编辑** - 通过连续对话实现迭代式图像优化
- **参考图像支持** - 使用参考图像进行精确合成
- **多模态推理** - 理解并处理详细的视觉和文本输入

## 可用模型

### gemini-2.5-flash-image ("Nano Banana")
- **特点**: 经济实惠、高质量、快速生成
- **最大输出分辨率**: 2K
- **适用场景**: 日常图像生成、快速原型制作

### gemini-3-pro-image-preview ("Nano Banana Pro")
- **特点**: 增强推理能力、4K输出、更强的编辑和搜索定位能力
- **最大输出分辨率**: 4K
- **参考图像**: 支持最多14张参考图像
- **适用场景**: 高质量图像生成、复杂编辑任务、专业创作

<details>
<summary>模型对比详情</summary>

| 特性 | gemini-2.5-flash-image | gemini-3-pro-image-preview |
|------|------------------------|----------------------------|
| 最大分辨率 | 2K | 4K |
| 参考图像数量 | 有限 | 最多14张 |
| 推理能力 | 标准 | 增强 |
| 生成速度 | 快速 | 较快 |
| 成本 | 经济 | 较高 |

</details>

## 快速开始

### Python 示例

#### 基础图像生成
```python
from google import genai

client = genai.Client()
response = client.models.generate_content(
    model="gemini-2.5-flash-image",
    contents="一座未来主义风格的城市，日落时分，等距视角，鲜艳的色彩和精细的建筑细节",
    config={
        "responseModalities": ["IMAGE"],
        "imageConfig": {
            "aspectRatio": "16:9",
            "imageSize": "2K"
        }
    }
)

# 获取Base64编码的PNG图像
image_data = response.candidates[0].content.parts[0].inlineData.data
```

#### 使用Pro模型生成4K图像
```python
from google import genai

client = genai.Client()
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="中国山水画风格的风景，水墨画效果，层峦叠嶂，云雾缭绕",
    config={
        "responseModalities": ["IMAGE"],
        "imageConfig": {
            "aspectRatio": "3:4",
            "imageSize": "4K"  # Pro模型支持4K
        }
    }
)

image_data = response.candidates[0].content.parts[0].inlineData.data
```

<details>
<summary>保存图像到文件</summary>

```python
import base64

# 将Base64数据解码并保存
with open("generated_image.png", "wb") as f:
    f.write(base64.b64decode(image_data))
```

</details>

### JavaScript/TypeScript 示例

```typescript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({});
const response = await ai.models.generateContent({
  model: "gemini-2.5-flash-image",
  contents: "一只可爱的卡通猫咪，赛博朋克风格，霓虹灯效果",
  config: {
    responseModalities: ["IMAGE"],
    imageConfig: {
      aspectRatio: "1:1",
      imageSize: "2K"
    }
  }
});

const imageData = response.candidates[0].content.parts[0].inlineData.data;
```

### Go 示例

```go
package main

import (
	"context"
	"encoding/base64"
	"fmt"
	"log"
	"os"
	"google.golang.org/genai"
)

func main() {
	ctx := context.Background()
	client, err := genai.NewClient(ctx, nil)
	if err != nil {
		log.Fatal(err)
	}

	config := &genai.GenerateContentConfig{
		ResponseModalities: []string{"IMAGE"},
		ImageConfig: &genai.ImageConfig{
			AspectRatio: "16:9",
			ImageSize:   "2K",
		},
	}

	resp, err := client.Models.GenerateContent(
		ctx,
		"gemini-2.5-flash-image",
		genai.Text("一片宁静的竹林，晨光透过竹叶"),
		config,
	)
	if err != nil {
		log.Fatal(err)
	}

	imageData := resp.Candidates[0].Content.Parts[0].InlineData.Data

	// 保存图像
	decoded, _ := base64.StdEncoding.DecodeString(imageData)
	os.WriteFile("output.png", decoded, 0644)
}
```

## 高级用法

### 图像编辑

使用自然语言指令对现有图像进行编辑：

```python
from google import genai
import base64

client = genai.Client()

# 读取原始图像
with open("original_image.png", "rb") as f:
    image_bytes = f.read()
    image_b64 = base64.b64encode(image_bytes).decode()

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        {
            "parts": [
                {"inline_data": {"mime_type": "image/png", "data": image_b64}},
                {"text": "将图片中的天空改成日落效果，增加橙红色调"}
            ]
        }
    ],
    config={
        "responseModalities": ["IMAGE"],
        "imageConfig": {
            "aspectRatio": "16:9",
            "imageSize": "4K"
        }
    }
)

edited_image = response.candidates[0].content.parts[0].inlineData.data
```

<details>
<summary>多轮迭代编辑</summary>

通过多次对话实现渐进式图像优化：

```python
from google import genai

client = genai.Client()

# 第一轮：生成初始图像
response1 = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="一座现代办公楼"
)

# 第二轮：基于上一轮结果继续编辑
response2 = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        {"parts": [
            {"inline_data": {
                "mime_type": "image/png",
                "data": response1.candidates[0].content.parts[0].inlineData.data
            }},
            {"text": "在建筑前添加一些绿化植物"}
        ]}
    ],
    config={
        "responseModalities": ["IMAGE"]
    }
)

# 第三轮：继续细化
response3 = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[
        {"parts": [
            {"inline_data": {
                "mime_type": "image/png",
                "data": response2.candidates[0].content.parts[0].inlineData.data
            }},
            {"text": "调整光线，使其看起来像黄昏时分"}
        ]}
    ],
    config={
        "responseModalities": ["IMAGE"]
    }
)
```

</details>

### 使用参考图像

Nano Banana Pro支持使用参考图像进行精确合成（最多14张）：

```python
from google import genai
import base64

client = genai.Client()

# 准备参考图像
reference_images = []
for i in range(1, 4):  # 使用3张参考图像
    with open(f"reference_{i}.png", "rb") as f:
        img_data = base64.b64encode(f.read()).decode()
        reference_images.append({
            "inline_data": {"mime_type": "image/png", "data": img_data}
        })

# 生成时使用参考图像
parts = reference_images + [
    {"text": "结合这些参考图像的风格，创作一幅新的艺术作品"}
]

response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents=[{"parts": parts}],
    config={
        "responseModalities": ["IMAGE"],
        "imageConfig": {
            "imageSize": "4K"
        }
    }
)
```

### 多模态输出（文本+图像）

同时生成图像和文本说明：

```python
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="创作一幅抽象艺术作品，并解释其艺术理念",
    config={
        "responseModalities": ["TEXT", "IMAGE"],
        "imageConfig": {
            "aspectRatio": "1:1",
            "imageSize": "2K"
        }
    }
)

# 解析响应
for part in response.candidates[0].content.parts:
    if hasattr(part, 'text'):
        print("文本说明:", part.text)
    elif hasattr(part, 'inline_data'):
        print("收到图像数据")
        image_data = part.inline_data.data
```

## 配置参数详解

### responseModalities（响应模态）
指定API返回的内容类型：
- `["IMAGE"]` - 仅返回图像
- `["TEXT"]` - 仅返回文本
- `["TEXT", "IMAGE"]` - 返回文本和图像

<details>
<summary>使用建议</summary>

- **纯图像生成**: 使用 `["IMAGE"]`
- **需要解释说明**: 使用 `["TEXT", "IMAGE"]`
- **图像分析**: 使用 `["TEXT"]`

</details>

### imageConfig（图像配置）

#### aspectRatio（宽高比）
支持的宽高比选项：
- `"1:1"` - 正方形，适合头像、图标
- `"16:9"` - 宽屏，适合横版海报、电脑壁纸
- `"9:16"` - 竖屏，适合手机壁纸、竖版海报
- `"3:4"` - 标准竖版，适合打印、肖像
- `"4:3"` - 标准横版，适合演示文稿

#### imageSize（图像尺寸）
- `"1K"` - 约1024像素
- `"2K"` - 约2048像素（gemini-2.5-flash-image最大支持）
- `"4K"` - 约4096像素（仅gemini-3-pro-image-preview支持）

<details>
<summary>尺寸选择建议</summary>

| 用途 | 推荐尺寸 | 模型选择 |
|------|---------|---------|
| 社交媒体 | 2K | flash-image |
| 网站素材 | 2K | flash-image |
| 打印输出 | 4K | pro-image-preview |
| 专业创作 | 4K | pro-image-preview |
| 快速预览 | 1K | flash-image |

</details>

## 最佳实践

### 提示词（Prompt）编写技巧

<details>
<summary>1. 具体且详细的描述</summary>

**好的示例**:
```
"一只橙色的虎斑猫，坐在木制窗台上，阳光从左侧照射进来，
背景是模糊的绿色植物，柔和的景深效果，自然光线，高清细节"
```

**不好的示例**:
```
"一只猫"
```

</details>

<details>
<summary>2. 明确艺术风格</summary>

常用风格关键词：
- 写实风格: "照片级真实感"、"超现实主义"
- 艺术风格: "水彩画"、"油画"、"水墨画"、"素描"
- 现代风格: "扁平设计"、"低多边形"、"赛博朋克"
- 传统风格: "中国山水画"、"浮世绘"、"印象派"

示例：
```python
contents="一座古桥，中国传统水墨画风格，留白构图，层次分明，墨色浓淡变化"
```

</details>

<details>
<summary>3. 控制光线和氛围</summary>

光线描述：
- 时间: "清晨柔和的阳光"、"正午强烈的光线"、"黄昏金色光芒"
- 方向: "侧光"、"逆光"、"顶光"
- 质感: "柔和散射光"、"硬质阴影"、"戏剧性光线"

氛围描述：
- "宁静祥和"、"充满活力"、"神秘莫测"、"温馨舒适"

</details>

<details>
<summary>4. 技术参数说明</summary>

可以在提示词中加入技术要求：
- "高清细节"、"4K分辨率"
- "景深效果"、"虚化背景"
- "电影级色彩"、"高对比度"
- "柔和色调"、"鲜艳色彩"

</details>

### 性能优化

<details>
<summary>选择合适的模型</summary>

**使用 gemini-2.5-flash-image 当:**
- 需要快速生成
- 预算有限
- 2K分辨率已满足需求
- 批量生成场景

**使用 gemini-3-pro-image-preview 当:**
- 需要4K高分辨率
- 进行复杂图像编辑
- 使用多个参考图像
- 需要更精确的理解和推理

</details>

<details>
<summary>批量处理建议</summary>

```python
from google import genai
import asyncio

client = genai.Client()

async def generate_image(prompt):
    return await client.models.generate_content_async(
        model="gemini-2.5-flash-image",
        contents=prompt,
        config={"responseModalities": ["IMAGE"]}
    )

# 并发生成多张图像
prompts = [
    "一座山",
    "一条河",
    "一片森林"
]

results = await asyncio.gather(*[generate_image(p) for p in prompts])
```

</details>

### 错误处理

<details>
<summary>常见错误和解决方案</summary>

```python
from google import genai
from google.genai import errors

client = genai.Client()

try:
    response = client.models.generate_content(
        model="gemini-3-pro-image-preview",
        contents="生成图像的提示词",
        config={
            "responseModalities": ["IMAGE"],
            "imageConfig": {"imageSize": "4K"}
        }
    )
except errors.InvalidArgument as e:
    print(f"参数错误: {e}")
    # 可能原因：不支持的imageSize、aspectRatio等
except errors.ResourceExhausted as e:
    print(f"配额已用尽: {e}")
    # 解决：检查API配额或切换到flash模型
except errors.PermissionDenied as e:
    print(f"权限错误: {e}")
    # 解决：检查API密钥设置
except Exception as e:
    print(f"其他错误: {e}")
```

</details>

## 身份验证

### 设置API密钥

<details>
<summary>环境变量方式</summary>

```bash
# Linux/Mac
export GEMINI_API_KEY="your-api-key-here"

# Windows (PowerShell)
$env:GEMINI_API_KEY="your-api-key-here"
```

</details>

<details>
<summary>代码中直接设置</summary>

```python
from google import genai

client = genai.Client(api_key="your-api-key-here")
```

**注意**: 不建议在生产代码中硬编码API密钥

</details>

<details>
<summary>使用服务账号（生产环境推荐）</summary>

```python
from google import genai
from google.auth import default

credentials, project = default()
client = genai.Client(credentials=credentials)
```

</details>

## CLI 工具扩展

社区维护的 Gemini CLI 扩展提供了便捷的命令行工具：

### 安装
```bash
gemini extensions install https://github.com/gemini-cli-extensions/nanobanana
```

### 使用

<details>
<summary>命令示例</summary>

```bash
# 生成图像
/generate 一片星空，梵高风格

# 编辑图像
/edit image.png 将天空改成夜晚

# 恢复图像
/restore old_photo.jpg

# 生成图标
/icon 一个音乐播放器应用的图标
```

</details>

<details>
<summary>环境变量配置</summary>

```bash
# 设置API密钥
export NANOBANANA_GEMINI_API_KEY="your-api-key"

# 选择模型
export NANOBANANA_MODEL="gemini-3-pro-image-preview"
```

</details>

## API规范

### REST API 端点

使用最新的REST API发现规范作为真实来源：

- **v1beta** (推荐): `https://generativelanguage.googleapis.com/$discovery/rest?version=v1beta`
- **v1**: `https://generativelanguage.googleapis.com/$discovery/rest?version=v1`

官方SDK（google-genai、@google/genai、google.golang.org/genai）默认使用v1beta。

### 响应格式

成功响应结构：
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "<base64-encoded-png>"
            }
          }
        ]
      }
    }
  ]
}
```

包含文本的响应：
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "这是图像的描述..."
          },
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "<base64-encoded-png>"
            }
          }
        ]
      }
    }
  ]
}
```

## 参考资源

### 官方文档
- [Nano Banana 图像生成官方文档](https://ai.google.dev/gemini-api/docs/image-generation.md.txt)
- [Gemini API 文档索引 (llms.txt)](https://ai.google.dev/gemini-api/docs/llms.txt)
- [模型规格说明](https://ai.google.dev/gemini-api/docs/models.md.txt)
- [Google AI Studio - Nano Banana](https://aistudio.google.com/models/gemini-2-5-flash-image)

### 学习资源
- [DeepMind Gemini Image 模型介绍](https://deepmind.google/models/gemini-image/)
- [Cookbook: 使用Nano-Banana进行原生图像生成](https://deepwiki.com/google-gemini/cookbook/5.1-native-image-generation-with-nano-banana)
- [Nano Banana API集成完整指南](https://apidog.com/blog/nano-banana-via-api/)

### 社区工具
- [Gemini CLI Extension - nanobanana](https://github.com/gemini-cli-extensions/nanobanana)

### SDK文档
- **Python SDK**: `google-genai` - [迁移指南](https://ai.google.dev/gemini-api/docs/migrate.md.txt)
- **JavaScript SDK**: `@google/genai` - [迁移指南](https://ai.google.dev/gemini-api/docs/migrate.md.txt)
- **Go SDK**: `google.golang.org/genai`

## 常见问题

<details>
<summary>Q: 如何选择Flash和Pro模型？</summary>

A:
- **日常使用、快速迭代**: 选择 `gemini-2.5-flash-image`，性价比高
- **高质量输出、专业创作**: 选择 `gemini-3-pro-image-preview`，支持4K和更多参考图像
- **成本考虑**: Flash模型更经济
- **分辨率需求**: 需要4K必须使用Pro模型

</details>

<details>
<summary>Q: 生成的图像质量不理想怎么办？</summary>

A:
1. **优化提示词**: 增加更多细节描述
2. **指定风格**: 明确说明艺术风格和技术要求
3. **调整参数**: 尝试更高的imageSize
4. **多轮编辑**: 使用迭代式编辑逐步优化
5. **使用参考图像**: 提供参考图像帮助模型理解

</details>

<details>
<summary>Q: 支持哪些图像格式？</summary>

A:
- **输入**: 支持常见格式（PNG、JPEG等），通过Base64编码传递
- **输出**: 统一为PNG格式的Base64编码数据
- **保存**: 解码后可保存为任意格式

</details>

<details>
<summary>Q: 有图像生成的数量限制吗？</summary>

A:
受API配额限制，具体限制取决于：
- API密钥的配额设置
- 所选模型（Pro模型通常配额更严格）
- 时间窗口内的请求频率

建议：
- 监控配额使用情况
- 实现错误重试机制
- 合理选择模型

</details>

<details>
<summary>Q: 可以生成特定尺寸的图像吗？</summary>

A:
API提供三个档位（1K、2K、4K），具体像素值由模型根据宽高比自动计算。

如需精确尺寸：
1. 生成较大尺寸（如4K）
2. 后期使用图像处理库裁剪/缩放

```python
from PIL import Image
import base64
import io

# 解码生成的图像
img_data = base64.b64decode(image_data)
img = Image.open(io.BytesIO(img_data))

# 调整到精确尺寸
img_resized = img.resize((1920, 1080), Image.LANCZOS)
img_resized.save("output_1920x1080.png")
```

</details>

<details>
<summary>Q: 如何提高生成速度？</summary>

A:
1. **使用Flash模型**: 比Pro模型更快
2. **降低分辨率**: 使用1K或2K而非4K
3. **并发请求**: 批量生成时使用异步并发
4. **优化提示词**: 避免过于复杂的描述

</details>

## 使用限制

- 遵守 [Google AI 使用政策](https://policies.google.com/terms/generative-ai/use-policy)
- 不得生成违法、有害、暴力、色情等内容
- 尊重版权，不要复制受版权保护的艺术作品
- 注意个人隐私保护

## 版本说明

> [!IMPORTANT]
> 本文档基于以下版本：
> - API版本: v1beta
> - 模型: gemini-2.5-flash-image, gemini-3-pro-image-preview
> - SDK: google-genai (Python), @google/genai (JavaScript), google.golang.org/genai (Go)

如遇到与文档不符的情况，请参考官方API规范和最新文档。

## 技术支持

遇到问题时的诊断步骤：

1. 检查API密钥是否正确设置
2. 确认使用的模型名称正确
3. 验证参数配置符合规范
4. 查看完整错误信息和堆栈跟踪
5. 参考官方文档的最新变更
6. 在 [GitHub Issues](https://github.com/google-gemini) 搜索类似问题

---

*最后更新: 2026-02-12*
