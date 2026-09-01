---
name: image-resize-crop-webp
description: 将图片按固定宽度缩放后，从左上角裁剪为 1080×1440 像素，并输出为 WebP。适用于批量图片预处理、统一尺寸和 WebP 转换。
---

# Image Resize → Crop → WebP

这个 Skill 用于复现以下图像处理流程：

1. 调整大小
   - 模式：填充（等价于保持原始宽高比的缩放）
   - 宽度：1080 px
   - 高度：按原图比例自动计算
   - 保持比例：开启
   - Gamma 校正：关闭
   - 跟随方向：关闭
   - 扩大/缩小：允许永久执行
   - 重采样：Lanczos

2. 裁剪
   - 裁剪方式：从“开始位置”
   - X：0 px
   - Y：0 px
   - 宽度：1080 px
   - 高度：1440 px
   - 即从缩放后的图片左上角开始，截取 1080×1440 px。

3. 输出
   - 将最终图片转换为 `.webp`
   - 保留透明通道（如果源图片具有透明度）
   - 默认使用高质量 WebP 编码
   - 输出文件名沿用原文件名，仅将扩展名改为 `.webp`

## 处理顺序

必须严格按照：

原图
→ 按宽度 1080 px 等比例缩放
→ 从左上角裁剪 1080×1440 px
→ 转换为 WebP

不要先裁剪再缩放，否则最终构图和尺寸会发生变化。

## Python / Pillow 实现参考

```python
from pathlib import Path
from PIL import Image, ImageOps

TARGET_WIDTH = 1080
CROP_HEIGHT = 1440

def process_image(input_path, output_path=None):
    input_path = Path(input_path)

    if output_path is None:
        output_path = input_path.with_suffix(".webp")
    else:
        output_path = Path(output_path)

    with Image.open(input_path) as img:
        # 如果图片包含 EXIF 方向信息，按实际方向展开。
        # 若需要完全复现“跟随方向关闭”的行为，可删除这一行。
        img = ImageOps.exif_transpose(img)

        # 等比例缩放到宽度 1080 px，重采样使用 Lanczos。
        ratio = TARGET_WIDTH / img.width
        new_height = round(img.height * ratio)
        img = img.resize(
            (TARGET_WIDTH, new_height),
            Image.Resampling.LANCZOS
        )

        # 从左上角裁剪 1080 × 1440。
        # 如果缩放后的高度不足 1440，使用黑色/透明填充前应根据项目需求处理。
        # 默认要求最终严格为 1080 × 1440，因此这里采用透明填充。
        if img.height < CROP_HEIGHT:
            mode = "RGBA" if "A" in img.getbands() else "RGB"
            background = (0, 0, 0, 0) if mode == "RGBA" else (0, 0, 0)
            canvas = Image.new(mode, (TARGET_WIDTH, CROP_HEIGHT), background)
            canvas.paste(img, (0, 0))
            img = canvas
        else:
            img = img.crop((0, 0, TARGET_WIDTH, CROP_HEIGHT))

        # WebP 支持 RGB/RGBA。
        if img.mode not in ("RGB", "RGBA"):
            img = img.convert("RGBA" if "A" in img.getbands() else "RGB")

        img.save(
            output_path,
            "WEBP",
            quality=95,
            method=6
        )

    return output_path

# 示例：
# process_image("input.jpg", "output.webp")
```

## 批量处理

处理文件夹时，对每个常见图片文件执行同样流程：

- JPG / JPEG
- PNG
- TIFF
- BMP
- GIF（如为动画 GIF，默认只处理第一帧；若项目要求保留动画，应另行处理）
- WebP

输出统一放到指定输出目录，并避免覆盖原文件。

## 重要行为

- 最终尺寸必须是 `1080 × 1440 px`。
- 缩放阶段只固定宽度 1080，高度按比例计算。
- 裁剪位置固定为左上角 `(0, 0)`。
- 裁剪尺寸固定为 `1080 × 1440`。
- 缩放算法使用 Lanczos。
- 最终格式必须是 WebP。
- 不应在缩放和裁剪之间加入其他改变构图的操作。
