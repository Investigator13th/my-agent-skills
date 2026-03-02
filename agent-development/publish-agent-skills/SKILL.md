---
name: publish-agent-skills
description: 指导如何将本地开发的Agent Skills按照SKILL.md规范组织，并通过SkillsMP或find-skills等平台发布，使其能被其他Agent顺利检索与调用。
author: Investigator13th
tags: ["skillsmp", "find-skills", "skill-publishing", "agent-integration"]
---

# 发布 Agent 技能指南 (Publishing Agent Skills)

本技能的目的是为了帮助开发者（或其他 Agent）按照业界标准格式规范并发布本地仓库中的技能资源。当您发布到 [SkillsMP](https://skillsmp.com/zh) 或 `find-skills` 组件并被收录后，您的技能将被整个大模型 Agent 生态顺利检索与复用。

## 适用场景
当你需要将此 `my-agent-skills` 仓库（或其他个人 Agent 技能库）分享到开源社区，让更多基于 Claude、OpenClaw 等框架的 Agent 助手能够自动发现并调用您的专属技能时使用此指导规范。

## 一、核心前提：完成 SKILL.md 文件规范

要让各大平台成功抓取与渲染你的技能，每个项目的技能说明文件都**必须**严格遵循类 Anthropic 定义的 `SKILL.md` 规范。

### 1. 强制声明 YAML 头部
Agent 在抓取技能时，最先分析解析的便是这块结构数据。在文件的最顶部加入如下代码块：

```yaml
---
name: your-skill-name        # 必须是纯小写字母加连字符，全局唯一标识
description: 描述文字... # 建议100字以内，一定要囊括你的核心解决痛点和关键词
author: your-github-username # 你的标识名/Github名
tags: ["主要场景", "关键词", "领域"] # 提供5-8个高品质标签以提升搜索曝光率
---
```

### 2. 补全 Markdown 主体逻辑结构
YAML的下方需要包含以下具体的行动指南框架，以使得未来的 Agent 知道如何拆解与使用这个技能：
- **适用场景**：明确指出在何种上下文下激活该技能。
- **输入要求**：触发功能必须补充的前置数据或参数。
- **工作流程**：指导 Agent 一步步思考与执行检索、分析、或生成的逻辑。
- **输出格式**：给定期望的返回形式（如JSON模板、多列Markdown表格等）。
- **示例调用**：提供一个“User: XXX -> Agent: YYY”的具体调用案例。

---

## 二、自动索引与发现机制落地

### 机制A：让 SkillsMP 自动索引
1. **GitHub 仓库状态**：必须是 Public 公开库，建议仓库名或简介可以包含 `skills` 关键词。
2. **严格的目录匹配**：
   - 技能必须采用双层结构：存放在同名子目录内（例如目录叫 `publish-agent-skills/`）。
   - 目录的名称必须与 `SKILL.md` 内定义的 `name` 完全一样。
   - 所有的说明文件自身名字必须全大写 `SKILL.md` 。
3. **设置 GitHub Topic**：在 GitHub 仓库首页为项目添加诸如 `agent-skills` 这样的 Topics 标签加速爬取，同时也可手动到 SkillsMP 平台提交仓库URL。

### 机制B：面向 find-skills / OpenClaw 工具
1. **借助 CLI 发布到 skills.sh 体系**：
   ```bash
   # 全局安装生态包 (如果是Node环境体系下)
   npm install -g @skills/cli
   
   # 在你的仓库路径内执行发布
   skills publish
   ```
2. **根目录索引清单**：可以由 Agent（比如我）为你一键提取仓库内的各项元数据，并在你的根目录构建一份 `skills.json`。部分定制化插件往往更喜欢直接请求抓取单一的 JSON 清单资源来获得你所有的能力列表。

## 三、避坑事项
- **颗粒度**：不要写成“全能的百宝箱”。一个 `SKILL.md` 最好围绕一个具体的垂直场景（比如：单把“分析公司财报”提炼成一项，而不是“全能金融分析”）。
- **权限及隐私**：切勿将带入个人认证信息的 `.env` 或带有绝对隐私的目录（如 `.agent/` 本地持久日志）随着 Git 发布出去（这点可通过 `.gitignore` 规避）。
