---
name: everything-claude-code-zh
description: 为 AI 智能体框架（Claude Code, Cursor, Codex）提供性能优化系统：智能体、技能、钩子、命令、规则与 MCP 配置的完整中文集合
triggers:
  - "如何使用 everything-claude-code"
  - "配置 Claude Code 智能体和技能"
  - "设置 AI 智能体框架"
  - "优化 Claude Code 性能和令牌使用"
  - "安装 everything-claude-code 插件"
  - "创建自定义 AI 智能体和技能"
  - "实现持续学习和内存持久化"
  - "使用 everything-claude-code 中的多智能体编排"
---

# Everything Claude Code 中文版

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## 概述

Everything Claude Code (ECC) 是一个为 AI 智能体框架打造的生产级性能优化系统，源自 Anthropic 黑客松获胜作品。它提供：

- **13 个专业智能体 (Agents)** — 用于规划、架构、TDD、代码审查、安全审查等
- **56 个领域技能 (Skills)** — 覆盖编码标准、后端/前端模式、ClickHouse、幻灯片生成、文章写作、市场研究等
- **32 个命令 (Commands)** — 用于会话管理、评测、并行化、PM2 编排等
- **60+ 规则 (Rules)** — 适用于 TypeScript、Python、Go、Java 的令牌优化和最佳实践
- **跨平台钩子 (Hooks)** — 内存持久化、持续学习、安全扫描

支持 **Claude Code**、**Cursor**、**Codex**、**Cowork** 和 **OpenCode**。

## 安装

### 方式 1：通过插件市场（推荐）

```bash
# 添加市场
/plugin marketplace add affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

### 方式 2：手动克隆与安装

```bash
# 克隆仓库
git clone https://github.com/xu-xiang/everything-claude-code-zh.git
cd everything-claude-code-zh

# 使用安装脚本（推荐）
./install.sh typescript    # 或 python, golang

# 支持多语言
./install.sh typescript python golang

# 为 Cursor 安装
./install.sh --target cursor typescript
```

### 方式 3：为 Codex CLI 安装

```bash
# 在 Claude Code 中运行
/codex-setup

# 或手动复制
cp codex.md ~/.config/codex/codex.md
```

### 安装规则（必需）

⚠️ **重要提示**：Claude Code 插件无法自动分发规则文件。必须手动安装：

```bash
# 使用安装脚本（安全处理通用和语言特定规则）
./install.sh typescript

# 或手动复制
cp -r rules/common/* ~/.claude/rules/
cp -r rules/typescript/* ~/.claude/rules/
```

## 核心组件

### 智能体 (Agents)

智能体是专门的子智能体，通过 `/delegate` 命令委派特定任务。

```bash
# 规划功能实现
/delegate planner "为用户添加身份验证"

# 架构设计决策
/delegate architect "设计微服务架构"

# TDD 工作流
/delegate tdd-guide "为用户服务编写测试"

# 代码审查
/delegate code-reviewer "审查 PR #123"

# 安全审查
/delegate security-reviewer "扫描 auth.ts 的漏洞"

# 构建错误修复
/delegate build-error-resolver "修复 TypeScript 编译错误"

# E2E 测试
/delegate e2e-runner "运行登录流程测试"

# 重构和清理
/delegate refactor-cleaner "移除未使用的导入"

# 文档更新
/delegate doc-updater "更新 API 文档"

# Go 代码审查
/delegate go-reviewer "审查 handler.go"

# Python 代码审查
/delegate python-reviewer "审查 views.py"

# 数据库审查
/delegate database-reviewer "审查 Supabase 迁移"
```

### 技能 (Skills)

技能为智能体提供领域知识和工作流模式。

#### 安装技能

```bash
# 通过插件（命名空间形式）
/everything-claude-code:use-skill coding-standards/typescript

# 手动安装（简短形式）
/use-skill backend-patterns/api-design
/use-skill frontend-patterns/react-hooks
/use-skill clickhouse-io/query-optimization
```

#### 主要技能类别

**编码标准**：
- `coding-standards/typescript` — TypeScript 最佳实践
- `coding-standards/python` — Python 风格指南
- `coding-standards/golang` — Go 惯用法
- `coding-standards/java` — Java Spring Boot 模式

**后端模式**：
- `backend-patterns/api-design` — RESTful API 设计
- `backend-patterns/database-optimization` — 数据库查询优化
- `backend-patterns/caching-strategies` — Redis/内存缓存
- `backend-patterns/django-patterns` — Django ORM 和视图模式

**前端模式**：
- `frontend-patterns/react-hooks` — React Hooks 最佳实践
- `frontend-patterns/nextjs-routing` — Next.js App Router 模式
- `frontend-patterns/state-management` — Zustand/Redux 模式
- `frontend-slides` — 无依赖 HTML 幻灯片生成器

**业务与内容**：
- `article-writing` — 使用指定语气进行长文写作
- `content-engine` — 多平台社交内容工作流
- `market-research` — 带源码引用的市场研究
- `investor-materials` — 商业计划书和财务模型
- `investor-outreach` — 个性化融资对接

**ClickHouse**：
- `clickhouse-io/query-optimization` — 高性能分析查询
- `clickhouse-io/data-engineering` — ETL 和数据管道
- `clickhouse-io/schema-design` — 表设计和分区

**持续学习**：
- `continuous-learning` — 从会话中自动提取模式
- `continuous-learning-v2` — 基于本能的带置信评分的学习

### 命令 (Commands)

命令是可执行操作，用于项目管理和编排。

#### 规划与开发

```bash
# 规划功能
/plan "添加用户身份验证"

# 重构代码
/refactor "将 auth 逻辑移至服务层"

# 运行测试
/test "运行单元测试"

# 安全扫描（需要 AgentShield）
/security-scan
```

#### 会话管理

```bash
# 列出会话历史
/sessions

# 保存当前会话
/save-session "auth-implementation"

# 加载之前的会话
/load-session "auth-implementation"
```

#### 多智能体编排（PM2）

```bash
# 启动 PM2
/pm2

# 规划多服务工作流
/multi-plan "为前端和后端实现功能 X"

# 执行多智能体工作流
/multi-execute

# 后端专用工作流
/multi-backend "优化数据库查询"

# 前端专用工作流
/multi-frontend "添加响应式导航"

# 完整栈工作流
/multi-workflow "端到端用户注册"
```

#### 包管理器设置

```bash
# 检测当前设置
/setup-pm --detect

# 设置首选包管理器
/setup-pm --project pnpm
/setup-pm --global bun

# 通过环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm
```

#### 评测与验证

```bash
# 运行验证
/eval "测试身份验证流程"

# 生成测试用例
/eval-generate "为用户 API 创建测试"

# 检查覆盖率
/coverage
```

#### 持续学习

```bash
# 导出学到的模式
/instinct-export

# 导入模式
/instinct-import path/to/pattern.md

# 查看学到的本能
/instinct-list
```

#### Codex 集成

```bash
# 为 Codex CLI 生成配置
/codex-setup
```

## 配置

### 包管理器检测

ECC 自动检测你首选的包管理器，优先级如下：

1. **环境变量**: `CLAUDE_PACKAGE_MANAGER`
2. **项目配置**: `.claude/package-manager.json`
3. **package.json**: `packageManager` 字段
4. **锁文件**: package-lock.json, yarn.lock, pnpm-lock.yaml, bun.lockb
5. **全局配置**: `~/.claude/package-manager.json`
6. **备选方案**: 首个可用的包管理器

```bash
# 设置项目级别
node scripts/setup-package-manager.js --project pnpm

# 设置全局
node scripts/setup-package-manager.js --global bun

# 检测当前
node scripts/setup-package-manager.js --detect
```

### 规则配置

规则按语言组织：

```
rules/
├── common/           # 所有语言通用
├── typescript/       # TypeScript 特定
├── python/           # Python 特定
├── golang/           # Go 特定
└── java/             # Java 特定
```

仅安装你需要的：

```bash
./install.sh typescript python    # 仅 TS 和 Python
./install.sh golang               # 仅 Go
```

### 钩子配置

钩子在 `.claude/hooks/` 中自动启用：

- `pre-session.js` — 加载之前会话的内存
- `post-session.js` — 保存当前会话的内存
- `continuous-learning.js` — 从会话中提取模式
- `security-scan.js` — 运行 AgentShield 扫描

## 实际代码示例

### TypeScript：使用智能体进行 API 开发

```typescript
// 1. 使用 planner 智能体规划
// /delegate planner "为用户认证创建 REST API"

// 2. 使用技能加载模式
// /use-skill backend-patterns/api-design
// /use-skill coding-standards/typescript

// 3. 智能体生成代码
import express from 'express';
import { z } from 'zod';

const app = express();

// 使用 Zod 进行请求验证的模式
const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

app.post('/api/auth/login', async (req, res) => {
  try {
    // 验证输入
    const { email, password } = LoginSchema.parse(req.body);
    
    // 业务逻辑（委派给服务层）
    const token = await authService.login(email, password);
    
    res.json({ token });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors });
    }
    res.status(500).json({ error: '内部服务器错误' });
  }
});

// 4. 使用 code-reviewer 审查
// /delegate code-reviewer "审查 auth API"

// 5. 使用 security-reviewer 检查安全性
// /delegate security-reviewer "扫描 auth.ts 的漏洞"
```

### Python：使用 Django 技能

```python
# 1. 加载 Django 技能
# /use-skill backend-patterns/django-patterns
# /use-skill coding-standards/python

# 2. 使用 planner 规划
# /delegate planner "创建 Django 用户配置文件模型"

# models.py
from django.db import models
from django.contrib.auth.models import User

class UserProfile(models.Model):
    """
    扩展基础 User 模型的用户配置文件。
    遵循 Django 最佳实践：一对一关系，使用信号。
    """
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile'
    )
    bio = models.TextField(max_length=500, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = '用户配置文件'
        verbose_name_plural = '用户配置文件'
    
    def __str__(self):
        return f"{self.user.username} 的配置文件"

# signals.py - 自动创建配置文件
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)

# 3. 使用 TDD 智能体编写测试
# /delegate tdd-guide "为 UserProfile 编写测试"

# tests.py
from django.test import TestCase
from django.contrib.auth.models import User

class UserProfileTestCase(TestCase):
    def test_profile_created_on_user_creation(self):
        """创建用户时配置文件应自动创建"""
        user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.assertTrue(hasattr(user, 'profile'))
        self.assertEqual(user.profile.user, user)
```

### Go：使用 Go 审查智能体

```go
// 1. 加载 Go 技能
// /use-skill coding-standards/golang

// 2. 使用 planner 规划
// /delegate planner "创建 HTTP 处理器与中间件"

package main

import (
    "encoding/json"
    "log"
    "net/http"
    "time"
)

// LoginRequest 表示登录请求负载
type LoginRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}

// LoginResponse 表示登录响应
type LoginResponse struct {
    Token string `json:"token"`
}

// loggingMiddleware 记录所有 HTTP 请求
func loggingMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        log.Printf("%s %s", r.Method, r.URL.Path)
        next(w, r)
        log.Printf("完成耗时 %v", time.Since(start))
    }
}

// handleLogin 处理用户登录
func handleLogin(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "仅允许 POST 方法", http.StatusMethodNotAllowed)
        return
    }
    
    var req LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "请求体无效", http.StatusBadRequest)
        return
    }
    
    // 验证
    if req.Email == "" || req.Password == "" {
        http.Error(w, "邮箱和密码为必填项", http.StatusBadRequest)
        return
    }
    
    // 业务逻辑（委派给服务层）
    token, err := authService.Login(req.Email, req.Password)
    if err != nil {
        http.Error(w, "身份验证失败", http.StatusUnauthorized)
        return
    }
    
    resp := LoginResponse{Token: token}
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(resp)
}

func main() {
    http.HandleFunc("/api/auth/login", loggingMiddleware(handleLogin))
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// 3. 使用 go-reviewer 审查
// /delegate go-reviewer "审查 handler.go"

// 4. 修复构建错误
// /delegate go-build-resolver "修复编译错误"
```

### React：使用前端模式

```typescript
// 1. 加载前端技能
// /use-skill frontend-patterns/react-hooks
// /use-skill frontend-patterns/state-management

// 2. 规划组件
// /delegate planner "使用 Zustand 创建身份验证流程"

// store/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  token: string | null;
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      
      login: async (email: string, password: string) => {
        const response = await fetch('/api/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });
        
        if (!response.ok) throw new Error('登录失败');
        
        const { token } = await response.json();
        set({ token });
      },
      
      logout: () => set({ token: null, user: null }),
    }),
    { name: 'auth-storage' }
  )
);

// components/LoginForm.tsx
import { useState } from 'react';
import { useAuthStore } from '@/store/authStore';

export function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const login = useAuthStore((state) => state.login);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login(email, password);
    } catch (error) {
      console.error('登录错误:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="邮箱"
        required
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="密码"
        required
      />
      <button type="submit">登录</button>
    </form>
  );
}

// 3. 审查组件
// /delegate code-reviewer "审查 LoginForm"
```

## 常见模式

### 模式 1：TDD 工作流

```bash
# 1. 规划功能
/delegate planner "为购物车添加结账功能"

# 2. 启用 TDD 模式
/delegate tdd-guide "为购物车服务编写测试"

# 3. 运行测试
/test

# 4. 审查代码
/delegate code-reviewer "审查购物车实现"

# 5. 检查覆盖率
/coverage
```

### 模式 2：多智能体并行开发

```bash
# 1. 规划多服务功能
/multi-plan "实现支付集成：前端表单、后端 API、数据库模式"

# 2. 执行并行开发
/multi-execute

# 3. 后端任务（单独的智能体）
/multi-backend "创建 Stripe API 集成"

# 4. 前端任务（单独的智能体）
/multi-frontend "构建支付表单 UI"

# 5. 协调工作流
/multi-workflow "端到端支付集成"
```

### 模式 3：持续学习

```bash
# 1. 启用持续学习
# （自动由 post-session 钩子运行）

# 2. 查看学到的模式
/instinct-list

# 3. 导出以共享
/instinct-export

# 4. 导入团队模式
/instinct-import team-patterns/api-design.md
```

### 模式 4：安全优先开发

```bash
# 1. 使用安全技能规划
/use-skill backend-patterns/security-best-practices
/delegate planner "添加 OAuth 身份验证"

# 2. 实现功能
/plan "创建 OAuth 流程"

# 3. 安全审查
/delegate security-reviewer "扫描 OAuth 实现"

# 4. 运行 AgentShield（如果已安装）
/security-scan

# 5. 修复漏洞
/delegate build-error-resolver "修复安全问题"
```

### 模式 5：内容创作工作流

```bash
# 1. 市场研究
/use-skill market-research
/delegate planner "研究 AI 编码工具市场"

# 2. 创建文章
/use-skill article-writing
"写一篇关于 AI 智能体在现代开发中的文章"

# 3. 生成社交内容
/use-skill content-engine
"从文章创建推文帖子和 LinkedIn 帖子"

# 4. 准备投资者材料
/use-skill investor-materials
"创建一页纸介绍和财务模型"
```

## 故障排除

### 问题：插件命令使用命名空间前缀

**症状**：必须使用 `/everything-claude-code:plan` 而不是 `/plan`

**解决方案**：
```bash
# 选项 1：使用完整命名空间（插件安装）
/everything-claude-code:plan "我的功能"

# 选项 2：手动安装以使用简短形式
./install.sh typescript
/plan "我的功能"
```

### 问题：规则未应用

**症状**：智能体不遵循 TypeScript/Python/Go 规则

**解决方案**：
```bash
# 规则必须手动安装
./install.sh typescript

# 验证规则已复制
ls ~/.claude/rules/

# 或手动
cp -r rules/common/* ~/.claude/rules/
cp -r rules/typescript/* ~/.claude/rules/
```

### 问题：包管理器检测错误

**症状**：使用 npm 但希望使用 pnpm

**解决方案**：
```bash
# 设置项目首选项
/setup-pm --project pnpm

# 或设置全局默认值
/setup-pm --global pnpm

# 或使用环境变量
export CLAUDE_PACKAGE_MANAGER=pnpm
```

### 问题：钩子未运行

**症状**：会话后无内存持久化或学习

**解决方案**：
```bash
# 检查钩子已安装
ls ~/.claude/hooks/

# 确保它们可执行
chmod +x ~/.claude/hooks/*.js

# 手动运行以测试
node ~/.claude/hooks/continuous-learning.js
```

### 问题：智能体委派失败

**症状**：`/delegate` 命令不起作用

**解决方案**：
```bash
# 验证智能体已安装
ls ~/.claude/agents/

# 手动复制智能体
cp -r agents/* ~/.claude/agents/

# 使用完整路径（插件安装）
/everything-claude-code:delegate planner "我的任务"
```

### 问题：Codex CLI 未识别命令

**症状**：Codex 不识别 ECC 命令

**解决方案**：
```bash
# 生成 Codex 配置
/codex-setup

# 或手动创建 codex.md
cp codex.md ~/.config/codex/codex.md

# 验证配置
cat ~/.config/codex/codex.md
```

### 问题：多智能体 PM2 命令失败

**症状**：`/pm2` 或 `/multi-*` 命令错误

**解决方案**：
```bash
# 确保 PM2 已全局安装
npm install -g pm2

# 验证 PM2 正在运行
pm2 list

# 检查 PM2 命令已安装
ls ~/.claude/commands/pm2/
```

### 问题：技能未加载

**症状**：`/use-skill` 未应用技能知识

**解决方案**：
```bash
# 验证技能已安装
ls ~/.claude/skills/

# 使用完整命名空间（插件安装）
/everything-claude-code:use-skill backend-patterns/api-design

# 或手动安装
cp -r skills/* ~/.claude/skills/
/use-skill backend-patterns/api-design
```

## 环境变量

```bash
# 包管理器首选项
export CLAUDE_PACKAGE_MANAGER=pnpm  # npm, pnpm, yarn, bun

# AgentShield 集成（可选）
export AGENTSHIELD_API_KEY=your_key_here

# 持续学习配置
export CLAUDE_LEARNING_ENABLED=true
export CLAUDE_LEARNING_THRESHOLD=0.7  # 置信度阈值

# 会话持久化
export CLAUDE_SESSION_DIR=~/.claude/sessions
export CLAUDE_MAX_SESSIONS=50
```

## 高级用法

### 创建自定义智能体

```markdown
# .claude/agents/my-custom-agent.md

你是一个专门从事 [领域] 的专家智能体。

## 职责
- [职责 1]
- [职责 2]

## 工作流
1. [步骤 1]
2. [步骤 2]

## 输出格式
[期望输出]
```

### 创建自定义技能

```markdown
---
name: my-custom-skill
description: 自定义领域知识
triggers:
  - "使用我的自定义技能"
---

# 自定义技能

## 概念
[解释核心概念]

## 模式
[记录通用模式]

## 示例
\`\`\`typescript
// 代码示例
\`\`\`
```

### 扩展规则

```markdown
# .claude/rules/my-custom-rule.md

## 规则：[规则名称]

### 上下文
[何时应用此规则]

### 指南
- [指南 1]
- [指南 2]

### 示例
\`\`\`
// 良好示例
\`\`\`

\`\`\`
// 不良示例
\`\`\`
```

## 参考资源

- **完整文档**: https://github.com/xu-xiang/everything-claude-code-zh
- **原始英文版本**: https://github.com/affaan-m/everything-claude-code
- **简明指南**: https://x.com/affaanmustafa/status/2012378465664745795
- **详细指南**: https://x.com/affaanmustafa/status/2014040193557471352
- **GitHub 市场**: https://github.com/marketplace/ecc-tools
- **OneSkill 主页**: https://oneskill.one

## 许可证

MIT License - 完全开源且可免费使用。
