---
name: medical-guide-ppt-planner
description: 将医学诊疗指南PDF文件转换为NotebookLM生成ppt时使用的PPT规划Markdown文件的专业工作流。当用户需要从医学指南PDF制作PPT规划文件时使用此技能，包括：提取指南内容、生成PPT的文字部分（页数由用户指定）、优化文字内容、检查错别字、优化视觉效果描述等完整流程。
author: Hu Song
version: 1.0.0
---

# 医学指南 PPT 规划生成器

# 使用此技能的场景

当用户提供一份医学诊疗指南 PDF 文件，并要求生成 NotebookLM PPT 规划 Markdown 文件时使用。

# 工作流程

** 严格分步执行规则**：本技能的六个步骤（第一步至第六步）**必须**在不同的对话回合中分步执行。
- **严禁**在一次回复中连续执行多个步骤。每完成一个步骤，必须停止并询问用户是否继续下一步。

# 前置条件

用户提供：一份医学指南 PDF 文件

# 第一步：询问用户需求

在开始执行任务前，使用 `AskUserQuestion` 工具询问用户以下问题：

```
问题1：需要制作一份多少页的 PPT？（说明：NotebookLM 默认每次生成 20 页，可分多次生成）
问题2：制作一份什么风格的 PPT？（例如：蓝绿色医疗风、简约商务风、现代科技风等）
```

获取用户输入后，将获得的参数用于后续步骤。

# 第二步：从pdf文件提取文本

必须使用 python 工具 PyMuPDF 读取 PDF 文件内容。
**重要**：读取后必须对提取的文本进行清理，去除异常换行符、页眉页脚及乱码，同时严格保留标题和段落结构（如"一、"、"1."、"摘要"等）。
参考清理逻辑：
1. 识别并移除页眉、页脚、页码（如"--- Page X ---", "· X ·"）。
2. 识别标题行（如章节序号、"摘要"、"关键词"、"表X"），确保其独立成段。
3. 智能合并正文段落：仅当上一行未以句末标点（。！？）结尾时，才将其与下一行合并，修复 PDF 意外断行。
清理后的文本，将获得的参数用于后续步骤。

# 第三步：生成 PPT 框架
**注意** 第一页为封面，最后一页为结束语页，第一页和最后一页文字要少。

根据清理后的文本内容生成 NotebookLM prompt 模板：

```
你需要为 NotebookLM 撰写一套生成 {页数} 页 PPT 的专属 prompt。

1. PPT 主题是"{主题名称}"。引用文献"{PDF文件名}"
2. 风格为蓝绿色医疗风（极简素雅）。
3. 请首先分析并理解我提供给你的 pdf 文献中的内容，每页 ppt 聚焦一个关键知识点。
4. 重点讲解临床表现、诊断、治疗、特殊人群管理，若一个部分内容较多，可以将一个部分拆分成多页 PPT。
5. 忽略指南编写背景、指南编写的方法和流程、局限性和展望、参考文献、编写人员等非知识性的内容。
6. 每页 PPT 的各个级别的标题、文字的字号和字体应该统一。请具体写出每一页中的文字内容，文字级别。需要规划的图形、图片。
7. 框架说明用英文（适配 NotebookLM 识别），PPT 内容为中文，最终整合为完整的 .md 文件

输出文件：notebooklm_prompt.md
```

# 第四步：扩展 PPT 文字内容

对中间页面（除第一页和最后一页之外的页）的 Body 部分增加更多文字内容：

```
请对除第一页和最后一页之外的页 ppt 的文字内容 Body 部分适当增加文字，要有条理，文字来源于"{PDF文件名}"
```

更新 notebooklm_prompt.md 文件。

# 第五步：检查和完善文字内容

检查并修正 PPT 文字内容：

```
请检查"notebooklm_prompt.md"这个文件中所规划的每张 ppt 的文字内容：
1. 确保所有 ppt 没有错别字
2. 语句通顺、有条理
3. ppt 中所有知识点都必须来源于"{PDF文件名}"
```

更新 notebooklm_prompt.md 文件。

# 第六步：优化视觉效果描述

对每页 PPT 的 Visual 部分进行改进：

```
文件"notebooklm_prompt.md"中所规划的各个 ppt 的**Visual:**部分是每张图片所配的视觉效果。请对每张 ppt 的**Visual:**部分进行改进：
1. 对**Visual:**部分的描述更详细一些
2. 每张 ppt 所搭配的视觉效果应该与文字内容的主题相配
3. 色调、风格应该符合 ppt 整体的要求
```

更新 notebooklm_prompt.md 文件。

# 输出文件

最终生成 `notebooklm_prompt.md` 文件，包含完整的 PPT 规划，每页包含：
- Slide number
- Title
- Body（文字内容）
- Visual（视觉效果描述）

# 输出格式示例

```markdown
**Design & Style**

- **Visual Theme**: Blue-Green Medical Style (Minimalist, Elegant, Professional).
- **Format**: Every slide must have a unified hierarchy of titles and varying font sizes for emphasis.

**Detailed Content Breakdown per Slide (Use this EXACT content for text generation):**


- **Slide 1**
  - **Title:** PHN 治疗：二线药物 (1)
  - **Key Concept:** 三环类抗抑郁药 (TCAs)
  - **Body:**
    - 1. **代表药物**：阿米替林 (Amitriptyline)。
    - 2. **作用机制**：抑制 5-HT 和 NE 再摄取，增强下行抑制通路，具有独特的镇痛和抗抑郁双重作用。
    - 3. **使用警惕**：抗胆碱能副作用强（口干、便秘、尿潴留）。
    - 4. **禁忌**：严重心脏病（心律失常）、青光眼、前列腺肥大及高龄患者慎用。
  - **Visual:** **Layout:** Chemical & Caution. **Left:** A clean 2D chemical structure of Amitriptyline. **Right:** A "Warning Triangle" icon containing symbols for: Heart (Arrhythmia), Eye (Glaucoma), and an Elderly Silhouette (High Risk). Use cautionary amber accent color.

- **Slide 2**
  - **Title:** PHN 治疗：二线药物 (2)
  - **Key Concept:** 5-HT/NE 再摄取抑制剂 (SNRIs)
  - **Body:**
    - 1. **代表药物**：度洛西汀 (Duloxetine)、文拉法辛 (Venlafaxine)。
    - 2. **优势**：心血管及抗胆碱能副作用较 TCAs 明显减少，安全性更高。
    - 3. **临床适用**：特别适合合并有抑郁或焦虑症状的 PHN 患者，可改善情绪障碍。
    - 4. **起效**：通常需 1-2 周起效，需足疗程使用。
  - **Visual:** **Layout:** Balance & Mood. **Left:** Chemical structure of Duloxetine. **Right:** A "Brain" icon that looks calm/happy, surrounded by "Serotonin/NE" particles. **Comparison:** A small scale showing "Lower Side Effects" compared to TCAs.
```

# 注意事项
1. 必须使用 python 工具 PyMuPDF 读取 PDF 文件内容。
2. 确保所有内容都来源于提供的 PDF 文件
3. 保持文字专业性和医学准确性
4. 忽略非知识性内容（背景、编写流程、参考文献等）
5. 视觉效果描述应与内容主题匹配
6. 保持整体风格统一（蓝绿色医疗风、极简素雅）
7. **分步执行**：严禁一次性完成所有步骤，
8. **严禁**在一次回复中连续执行多个步骤。每完成一个步骤，必须停止并询问用户是否继续下一步。
9. 最终生成 NotebookLM PPT 所有页面规划必须存为一个本地 Markdown 文件。
