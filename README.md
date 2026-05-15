# 🎯 frontend-resume-skill

> **一个帮助前端开发者从项目代码中提炼简历亮点的 Claude Skill**

把你的项目代码丢给它，它会帮你：
- 🔍 识别你自己都没意识到的技术亮点
- 📝 生成符合 STAR 法则的简历描述
- ❓ 生成分层面试题 + 结构化回答模板

---

## 📖 目录

- [这是什么](#这是什么)
- [快速开始](#快速开始)
- [使用示例](#使用示例)
- [Skill 设计思路](#skill-设计思路)
- [适用人群](#适用人群)
- [常见问题](#常见问题)
- [贡献指南](#贡献指南)

---

## 这是什么

这是一个 **Claude Skill**（也叫 System Prompt / 提示词工程文件），可以让 Claude 按照结构化流程分析你的前端项目代码，并输出完整的求职材料包。

**它不是**：

- ❌ 一个网页工具或 App
- ❌ 一个简历模板
- ❌ 一个需要填写表格的生成器

**它是**：

- ✅ 一套对话式工作流，Claude 主动引导你完成信息采集
- ✅ 一个会主动挖掘"你自己觉得没什么、其实很有价值"的隐性亮点的分析器
- ✅ 根据你的工作年限自动调整策略（1-3年 / 3-5年 / 5年+）

---

## 快速开始

### 方式一：在 Claude.ai 中使用（推荐）

1. 打开 [claude.ai](https://claude.ai)
2. 进入 **Projects**，创建一个新项目
3. 在项目的 **Custom Instructions（自定义指令）** 中，粘贴 `SKILL.md` 的全部内容
4. 上传你的项目代码（或直接粘贴关键文件）
5. 发送任意触发词开始，例如：
   - `帮我看看这段代码能写什么简历`
   - `这个项目怎么写在简历上`
   - `帮我提炼一下项目亮点`

### 方式二：在 API 中使用

将 `SKILL.md` 内容作为 `system` prompt 传入：

```python
import anthropic

with open("SKILL.md", "r") as f:
    skill_content = f.read()

client = anthropic.Anthropic()
message = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=4096,
    system=skill_content,
    messages=[
        {"role": "user", "content": "这是我的项目代码：[粘贴你的代码]"}
    ]
)
print(message.content[0].text)
```

### 方式三：在 Cursor / Continue 等 IDE 插件中使用

在插件的 system prompt 或 `.cursorrules` 中引入 `SKILL.md` 内容。

---

## 使用示例

### 示例一：权限系统

**输入代码片段**（简化示意）：
```javascript
// router/index.js
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore()
  if (!userStore.permissions) await userStore.fetchPermissions()
  if (to.meta.permission && !userStore.hasPermission(to.meta.permission)) {
    next('/403')
  } else {
    next()
  }
})

// directives/permission.js
export const vPermission = {
  mounted(el, binding) {
    if (!useUserStore().hasPermission(binding.value)) {
      el.parentNode?.removeChild(el)
    }
  }
}
```

**Skill 输出（简化）**：

```
[Phase 1 · Step 1 完成] 用户画像采集

在分析代码之前，我需要了解以下信息：
1. 你的工作年限大概是多少？（1-3年 / 3-5年 / 5年+）
2. 这个权限系统是你主导设计的，还是在已有基础上迭代的？
3. 这个项目的用户规模大概是多少？
...

[Phase 2 · Step 5 完成] 八维度分析发现：

🔴 P0 亮点 · 路由 + 指令双层权限控制
来源：5.5 技术亮点（权限系统）

简历描述建议：
"主导设计前端双层权限控制方案，路由级守卫控制页面访问权限，
指令级（v-permission）控制按钮级操作权限，支持动态权限刷新，
权限变更无需重新部署，相比硬编码方案维护成本降低约 60%。"

面试题（四层）：
第一层：路由守卫和 v-if 控制权限有什么区别？分别适用什么场景？
第二层：如果让你设计一个支持多角色的 RBAC 系统，你会怎么做？
...
```

---

### 示例二：隐性亮点挖掘（D 类 · 踩坑类）

**用户描述**："我就是发现接口会重复请求，加了个 loading 锁"

**代码**：
```javascript
const useRequest = (apiFn) => {
  const loading = ref(false)
  const execute = async (...args) => {
    if (loading.value) return  // 防重复提交
    loading.value = true
    try {
      return await apiFn(...args)
    } finally {
      loading.value = false
    }
  }
  return { loading, execute }
}
```

**Skill 挖掘结果**：

```
💎 发现 D 类隐性亮点（踩坑解决类）

你可能觉得"这只是加了个 loading 锁，没什么好说的"，但这恰恰是值得写的：

✅ 简历描述建议：
"发现高频操作页面存在接口重复提交问题，封装 useRequest hook
统一实现防重复提交逻辑，通过 loading 状态锁避免并发请求，
改造后覆盖全项目 XX 个表单提交场景，彻底消除重复提交引发的
数据异常问题。"

💡 为什么这值得写？
- 体现了"发现问题 → 系统性解决" 的工程思维
- 封装成 hook 说明有抽象复用能力
- 解决的是真实业务痛点，而非技术练习
```

---

## Skill 设计思路

### 三阶段 10 步流程

```
Phase 1 · 信息采集（Step 1-3）
  Step 1 · 用户画像：工作年限 + 项目背景        ← 决定亮点优先级策略
  Step 2 · AI 使用经历                          ← 2024+ 重要加分项
  Step 3 · 转型意向                             ← 影响简历定位

Phase 2 · 代码分析（Step 4-6）
  Step 4 · 技术栈识别
  Step 5 · 八维度深度分析                       ← 组件/请求/状态/CSS/亮点/性能/工程化/监控
  Step 6 · 隐性亮点挖掘（A-F 六类）             ← 核心创新点

Phase 3 · 输出生成（Step 7-10）
  Step 7 · 简历描述（P0/P1/P2 优先级）
  Step 8 · 四层面试题
  Step 9 · 结构化回答模板
  Step 10 · 最终汇总
```

### 核心创新：六类隐性亮点挖掘

| 类型 | 典型场景 | 用户的认知误区 |
|------|---------|-------------|
| A 类 | 封装了请求 hook | "就是简单封装，没啥好说的" |
| B 类 | 实现了表单联动 | "这是产品要求的，我只是实现了一下" |
| C 类 | 配了 ESLint + Husky | "大家都这么做，不算亮点" |
| D 类 | 修复了内存泄漏 | "说出来显得自己之前很菜" |
| E 类 | 推动引入了某工具 | "不是我独立做的，不好意思写" |
| F 类 | 实现了可配置系统 | "那只是业务需求，不是技术亮点" |

---

## 适用人群

| 人群 | 适用场景 |
|------|---------|
| 🟢 1-3 年初中级前端 | 不知道自己的项目有什么可以写的 |
| 🟡 3-5 年高级前端 | 有内容但不知道怎么包装得更有深度 |
| 🔴 5年+ 资深前端 | 需要把技术价值翻译成业务语言 |
| 🔄 准备跳槽的前端 | 系统整理过去 1-2 个项目的核心价值 |
| 🤖 有 AI 开发经历的前端 | 不知道如何在简历中体现 AI 相关能力 |

---

## 常见问题

**Q：代码要上传多少才够？**

A：建议提供以下文件（按优先级）：
1. 你认为最能体现技术深度的文件（router、store、核心组件）
2. 项目入口文件（main.ts、App.vue）
3. 配置文件（vite.config.ts、package.json）

不需要上传全部代码，关键文件即可。

**Q：代码涉及公司机密怎么办？**

A：可以做以下处理后再上传：
- 删除业务相关的文本内容，保留代码结构
- 将公司名、产品名替换为占位符（如 `CompanyName`、`ProjectName`）
- 只提供核心逻辑部分，不包含数据库连接信息等敏感内容

**Q：没有代码，只有项目描述，可以用吗？**

A：可以，直接描述项目情况，Skill 会在 Step 6 通过追问挖掘亮点。

**Q：Skill 适配哪些 Claude 版本？**

A：推荐 Claude Sonnet 或 Opus 系列，效果更好。Claude Haiku 也可用，但输出深度略有差异。

---

## 贡献指南

欢迎提 Issue 和 PR！

**可以贡献的方向**：
- 🐛 Bug：某类代码识别错误 / 输出格式问题
- ✨ 新亮点类型：发现 A-F 之外的隐性亮点类型
- 🌐 示例补充：补充更多语言/框架的示例（React / Angular / Svelte）
- 📖 文档改进：示例不够清晰、说明不够详细

**PR 格式**：
```
类型: 简短描述

- 改动点 1
- 改动点 2
```

类型：`fix` / `feat` / `docs` / `example`

---

## License

MIT © 2024

---

> 如果这个 Skill 帮你找到了新工作，欢迎来 Issues 区分享你的故事 🎉
