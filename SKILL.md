---
name: video-to-markdown
description: 视频转Markdown技能。当用户提供视频链接（B站/YouTube/抖音等）并要求转录、提取文字稿、转文字时自动触发。一键完成：下载音频 → Whisper转录 → 生成Markdown思维导图格式，输出三个同名文件（m4a音频、txt原始转录、md思维导图）。触发场景：转录视频、视频转文字、提取视频内容、视频笔记、视频文字稿、把视频变成文字等。
---

# 视频转文字稿技能

一键将视频转换为三个成果文件：音频 + 原始转录 + Markdown思维导图。

---

## 触发场景

用户提供视频链接后，使用以下指令触发：
- "转录这个视频"
- "把视频转成文字"
- "提取视频内容"
- "视频笔记"
- "视频文字稿"
- "解析此视频"

---

## 工具路径

| 工具 | 路径 |
|------|------|
| yt-dlp | `/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.yt-dlp_Microsoft.Winget.Source_8wekyb3d8bbwe/yt-dlp.exe` |
| Python | `/c/Users/ldy/AppData/Local/Programs/Python/Python312/python.exe` |
| ffmpeg | `/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe/ffmpeg-N-123074-g4e32fb4c2a-win64-gpl/bin` |

---

## 执行流程

```
视频链接 → 下载音频 → Whisper转录 → 生成Markdown → 重命名文件
```

---

## Step 1：下载音频 + 获取标题（并行执行）

### Bash命令

```bash
# 下载音频（合并export和yt-dlp命令）
export PATH="/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe/ffmpeg-N-123074-g4e32fb4c2a-win64-gpl/bin:$PATH" && /c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.yt-dlp_Microsoft.Winget.Source_8wekyb3d8bbwe/yt-dlp.exe -x --audio-format m4a --audio-quality 0 --ffmpeg-location "/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe/ffmpeg-N-123074-g4e32fb4c2a-win64-gpl/bin" -o "C:/Users/ldy/Downloads/_temp_audio.%(ext)s" "视频链接"

# 获取视频标题（并行执行）
/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.yt-dlp_Microsoft.Winget.Source_8wekyb3d8bbwe/yt-dlp.exe --get-filename -o "%(title)s" "视频链接"
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `-x` | 仅提取音频 |
| `--audio-format m4a` | 输出m4a格式 |
| `--audio-quality 0` | 最佳音频质量 |
| `--ffmpeg-location` | ffmpeg路径 |
| `-o` | 输出路径，使用临时文件名 |
| `--get-filename` | 获取视频标题用于重命名 |

### 输出

| 文件 | 路径 |
|------|------|
| 临时音频 | `C:/Users/ldy/Downloads/_temp_audio.m4a` |
| 视频标题 | 用于最终文件命名 |

---

## Step 2：Whisper转录音频

### Bash命令

```bash
# Whisper转录（合并export和whisper命令）
export PATH="/c/Users/ldy/AppData/Local/Microsoft/WinGet/Packages/yt-dlp.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe/ffmpeg-N-123074-g4e32fb4c2a-win64-gpl/bin:$PATH" && /c/Users/ldy/AppData/Local/Programs/Python/Python312/python.exe -m whisper "C:/Users/ldy/Downloads/_temp_audio.m4a" --model base --language zh --output_format txt --output_dir "C:/Users/ldy/Downloads/"
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--model base` | 使用base模型（推荐） |
| `--language zh` | 中文转录 |
| `--output_format txt` | 输出txt格式 |
| `--output_dir` | 输出目录 |
| `timeout` | 建议600000ms（10分钟） |

### 输出

| 文件 | 路径 |
|------|------|
| 转录文件 | `C:/Users/ldy/Downloads/_temp_audio.txt` |

---

## Step 3：生成Markdown思维导图

### 流程

1. **读取转录文件**：`Read _temp_audio.txt`
2. **LLM生成结构化md**：根据转录内容生成思维导图格式
3. **写入md文件**：`Write [视频标题].md`

### Markdown输出模板

```markdown
# [视频标题] - 思维导图

## 元信息
- **来源**：B站视频转录总结
- **原始链接**：[视频URL]
- **处理时间**：[YYYY-MM-DD]
- **视频时长**：约X分钟
- **处理方式**：Whisper 转录 + LLM 结构化总结

---

## 思维导图结构

[结构化内容]

---

## 内容详解

[详细内容]

---

## 关键要点速记

[要点列表]
```

---

## Step 4：重命名文件

### Bash命令

```bash
# 重命名临时文件为最终文件名
mv "C:/Users/ldy/Downloads/_temp_audio.m4a" "C:/Users/ldy/Downloads/[视频标题].m4a" && mv "C:/Users/ldy/Downloads/_temp_audio.txt" "C:/Users/ldy/Downloads/[视频标题].txt"
```

---

## 成果文件表

完成后输出：

| 步骤 | 文件名 | 格式 | 状态 |
|------|--------|------|------|
| Step 1 | [视频标题].m4a | 音频 | ✅ 已生成 |
| Step 2 | [视频标题].txt | 转录 | ✅ 已生成 |
| Step 3 | [视频标题].md | 思维导图 | ✅ 已生成 |

**保存位置**：`C:\Users\ldy\Downloads\`

---

## 权限配置（settings.json）

为避免每次运行询问权限，在 `~/.claude/settings.json` 中添加：

```json
{
  "permissions": {
    "allow": [
      "Bash(*)",
      "Read(**)",
      "Write(**)"
    ]
  }
}
```

| 配置 | 说明 |
|------|------|
| `Bash(*)` | 允许所有Bash命令 |
| `Read(**)` | 允许读取所有文件 |
| `Write(**)` | 允许写入所有文件 |

---

## 支持的视频平台

- Bilibili (B站)
- YouTube
- 抖音 / TikTok
- 小红书
- 其他主流视频平台

---

## Whisper模型选择

| 模型 | 速度 | 准确率 | 适用场景 |
|------|------|--------|----------|
| tiny | 最快 | 较低 | 快速预览 |
| base | 快 | 良好 | **推荐** |
| small | 中等 | 较高 | 高质量需求 |
| medium | 较慢 | 高 | 专业场景 |

---

## 注意事项

1. **临时文件命名**：使用 `_temp_audio` 避免中文编码问题
2. **音频质量**：清晰的录音效果更好
3. **转录时间**：取决于视频时长和模型大小
4. **隐私安全**：本地转录，数据不上传
5. **权限配置**：使用通配符避免频繁询问权限

---

## 完整执行示例

**用户输入**：
```
https://www.bilibili.com/video/BV1xxx 解析此视频
```

**执行过程**：
```
🎬 开始视频转录...

📥 [步骤1/3] 下载音频
   → 下载视频音频... (并行获取标题)
   → 保存为：_temp_audio.m4a (X MB)

🎤 [步骤2/3] Whisper 转录
   → 加载 base 模型...
   → 处理音频文件...
   → 生成原始转录稿...

📝 [步骤3/3] 生成 Markdown
   → 分析内容结构...
   → 提取关键要点...
   → 转换思维导图格式...

📦 [步骤4/4] 重命名文件
   → 重命名为最终文件名

✅ 完成！输出文件：
   📁 [视频标题].m4a
   📁 [视频标题].txt
   📁 [视频标题].md
```