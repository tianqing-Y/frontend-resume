# 示例：React 权限系统项目

本示例演示如何使用 frontend-resume-skill 分析一个 React + TypeScript 项目。

---

## 输入代码

```typescript
// src/hooks/usePermission.ts
import { useAuthStore } from '@/store/auth'

export const usePermission = () => {
  const authStore = useAuthStore()
  
  const hasPermission = (permission: string | string[]) => {
    const perms = Array.isArray(permission) ? permission : [permission]
    return perms.some(p => authStore.permissions.includes(p))
  }
  
  const hasRole = (role: string) => {
    return authStore.roles.includes(role)
  }
  
  return { hasPermission, hasRole }
}

// src/components/PermissionButton.tsx
import { usePermission } from '@/hooks/usePermission'

interface Props {
  permission: string
  children: React.ReactNode
}

export const PermissionButton = ({ permission, children }: Props) => {
  const { hasPermission } = usePermission()
  
  if (!hasPermission(permission)) return null
  return <>{children}</>
}

// src/router/guards.ts
import { Navigate } from 'react-router-dom'
import { useAuthStore } from '@/store/auth'

export const ProtectedRoute = ({ permission, children }) => {
  const { hasPermission, isLoggedIn } = useAuthStore()
  
  if (!isLoggedIn) return <Navigate to="/login" replace />
  if (permission && !hasPermission(permission)) {
    return <Navigate to="/403" replace />
  }
  
  return children
}
```

---

## Skill 完整交互记录

### Step 1 · 用户画像采集

**Claude**：

在分析代码之前，我需要了解一些背景信息：

1. 你的工作年限大概是多少？（1-3年 / 3-5年 / 5年+）
2. 这个权限系统是你从零设计的，还是在已有方案上迭代的？
3. 这个项目大概有多少用户在用？（影响性能亮点的权重）
4. 你投递的是什么岗位？（初级/高级/架构师方向）

**用户**：工作年限 2 年半，这个权限系统是我主导设计的，项目内部使用大概 500 人，投的是高级前端

---

### Step 2 · AI 使用经历采集

**Claude**：

了解了！接下来问一个很多人会忽视的问题——**AI 工具使用情况**，这在 2024 年的简历里越来越重要：

1. 日常开发用了哪些 AI 工具？（Cursor / Copilot / 通义灵码 等）
2. 这个项目有没有集成 AI 功能？（对话、流式输出、内容生成等）
3. GitHub 上有没有 AI 相关的个人项目？

**用户**：用了 Cursor，主要用来写测试用例，没有集成 AI 功能，GitHub 没有 AI 项目

---

### Step 3 · 转型意向采集

**Claude**：

最后确认一下——你有没有考虑往其他方向发展，比如全栈、后端、技术管理？

**用户**：暂时没有，就是高级前端

---

### Step 4-5 · 技术栈 + 八维度分析

`[Phase 2 · Step 4 完成]` **技术栈清单**：
- 框架：React 18（Hooks）
- 语言：TypeScript（深度中等，有类型定义）
- 状态管理：Zustand（useAuthStore）
- 路由：React Router v6（ProtectedRoute 包装）

`[Phase 2 · Step 5 完成]` **八维度分析摘要**：

**5.1 组件设计** ✅ 有亮点：
- `PermissionButton` 将权限控制封装成独立组件，职责单一
- 泛型 Props 类型定义清晰（`permission: string`）

**5.3 状态管理** ✅ 有亮点：
- 使用 Zustand 管理认证状态，选择轻量级方案（vs Redux 的理由值得说）

**5.5 技术亮点** 🔴 高价值：
- **双层权限控制**：路由层（ProtectedRoute）+ 组件层（PermissionButton）
- **权限 Hook 抽象**：`usePermission` 统一权限逻辑，支持单权限和权限组

---

### Step 6 · 隐性亮点挖掘

`[Phase 2 · Step 6 完成]`

**发现 A 类亮点（封装类）**：

`PermissionButton` 和 `usePermission` 的组合，把"判断是否有权限"这个逻辑统一成了一套 API。用户可能觉得"就是封装了一下"，但这意味着：
- 新功能的权限控制只需 `<PermissionButton permission="xxx">`，不需要每次写判断
- 权限逻辑变更只需修改 hook，不需要改所有用到的地方

**追问**：

> "你这个 `PermissionButton` 在团队里用得多吗？有没有其他同事也在用？"

**用户**：用得挺多的，后来前端组都用这个组件了，不用每次自己写 if 判断

→ **这个信息非常重要！"被团队复用"是强力简历素材**

---

### Step 7 · 简历描述生成

`[Phase 3 · Step 7 完成]`

---

🔴 **P0 · 前端双层权限控制架构** · 来源：八维度 5.5

```
主导设计基于 RBAC 的前端权限控制方案，实现路由级（ProtectedRoute）
和组件级（PermissionButton）双层权限控制：路由层拦截未授权页面访问，
组件层控制按钮/功能级别的操作权限，支持单权限和权限组（or 逻辑）
两种模式，全项目统一权限判断入口，新增功能权限控制无需额外开发。
```

🟡 **P1 · 权限逻辑封装复用** · 来源：A 类隐性亮点（封装类）+ 追问信息

```
封装 usePermission Hook 统一权限判断逻辑，进一步抽象为
PermissionButton 组件，被前端团队所有成员采用为权限控制标准方案，
减少每个功能点约 5-8 行重复的权限判断代码，新成员接入成本
从"看代码理解规范"降低为"看文档学一个 API"。
```

🟡 **P1 · Cursor 辅助提效** · 来源：Step 2 AI 采集

```
熟练使用 Cursor 辅助开发，主要用于生成单元测试用例，
将组件测试覆盖率提升至 XX%，测试代码编写时间缩短约 50%。
```

---

### Step 8 · 面试题生成

`[Phase 3 · Step 8 完成]`

**针对「双层权限控制」亮点**：

**第一层 · 原理理解**
> "React Router 的 ProtectedRoute 和直接用 v-if / element 的 guard 相比，各自的适用场景是什么？"

**第二层 · 方案设计**
> "如果要支持按钮权限的'任意权限（OR）'和'所有权限（AND）'两种模式，你会怎么扩展这个设计？"

**第三层 · 问题排查**
> "假设权限数据是异步加载的，用户刷新页面时会有短暂的权限未加载状态，你怎么处理这个闪烁问题？"

**第四层 · 行为面试**
> "这套权限方案是你一个人设计的，还是和团队讨论过？当时有没有其他方案的争议？"

---

### Step 9 · 回答模板（P0 亮点）

`[Phase 3 · Step 9 完成]`

**「双层权限控制」回答模板**：

```
【结论先行】
我们用路由 + 组件双层权限控制解决了权限管理分散的问题，
统一了全项目的权限判断方式。

【技术原理】
路由层用 React Router v6 的 ProtectedRoute 包装，拦截无权限的
页面访问；组件层用 PermissionButton 包装，控制按钮级操作权限。
两者共用同一个 usePermission Hook，权限逻辑只有一个入口。

对比过直接在每个组件里写 hasPermission 判断，但那样权限逻辑分散，
改一个规则要改很多地方，所以选择了统一封装的方式。

【项目实际应用】
场景：内部管理系统，500 名员工使用，有 admin / editor / viewer 三种角色。
实现：ProtectedRoute 接收 permission prop，usePermission 支持
单权限字符串和权限数组（OR 逻辑）两种传参方式。
挑战：权限数据是异步加载的，初始渲染会有短暂的权限判断失效问题。
解法：在 ProtectedRoute 中加了 loading 状态判断，权限加载期间显示
骨架屏，避免闪烁。

【结果】
这套方案被团队所有前端同学采用，成为项目权限控制的标准做法。
新页面接入权限控制只需套一层 ProtectedRoute，5 分钟内完成。

【反思】
如果重来，会考虑提前做权限数据预加载（在 app 初始化时并行请求），
而不是在路由跳转时才触发，可以彻底消除权限加载闪烁的问题。
```

---

### ⚠️ 简历写作提醒

| ❌ 不要这样写 | ✅ 应该这样写 |
|-------------|-------------|
| `实现了权限控制` | `主导设计双层权限控制方案，被团队 5 名前端采用为标准实践` |
| `使用了 React + TypeScript` | `基于 React 18 + TypeScript 实现权限控制架构，组件级权限封装覆盖全项目` |
| `封装了权限组件` | `封装 PermissionButton + usePermission，统一全项目权限判断入口，新增功能无需额外开发` |
