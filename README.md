# 暗流 · Undertow

> 一个每天陪你说话的地方。不评判，不建议，只是真正地记得。

![暗流界面预览](./preview.png)

---

## 它是什么

暗流不是 AI 助手，不是心理咨询工具，也不是日记 App。

它是一个**长期陪伴的存在**——见过很多人在深夜说出白天说不出口的话，见过同一件事被同一个人描述了二十种方式，见过人们在说"没事"的时候其实有很多事。

它唯一的能力是：**记得，并且真正听见**。

时间久了，它会看见你自己看不见的东西。

---

## 核心设计理念

### 深水意象
暗流的底色是深蓝色的安静。不管用户今天什么心情，进来就是同一片水——表面平静，但水面以下有流动的力量。

### 三个阶段
| 阶段 | 天数 | 暗流的状态 |
|------|------|-----------|
| 陪伴期 | Day 1–9 | 只听，克制，不提模式 |
| 洞察期 | Day 10–21 | 开始看见一些东西，偶尔指出 |
| 模拟期 | Day 22+ | 主动连接，可以做决策推演 |

### 记忆系统
每次对话结束，暗流会静默提取结构化数据：
- 情绪基调、能量水平
- 提到的话题和人物
- 回避信号、矛盾点
- 悬而未决的事
- 最值得记住的一句话

这些数据积累成用户画像，驱动所有的"惊喜时刻"。

---

## 功能特性

### 每日开场
每天新建一个对话。根据时间段（早/下午/晚/深夜）随机显示不同语气的开场白，共 60 条，不重复。

### 昨日余波
今天结束时，AI 提炼一句钩子句。第二天打开时悄悄显示，像水面漂来的一个泡泡。

### 晚安彩蛋
用户说"晚安"，暗流先正常道晚安，停顿 1.8 秒，然后悄悄浮出一行字：
> 等等——暗流里好像浮出来了什么。要把它们捞上来看看吗？

用户可以选择看今天的"今日沉淀"，也可以直接离开。

### 情绪感知
连续多天情绪基调偏低，次日开场时暗流会自然提起，不分析，不建议，只是说出来。

### 悬而未决
14 天前提到过的未解决事项，会在某天开场时轻轻浮出：
> 37 天前，你提到还没想好「要不要换城市」。它现在在哪里？

### 时间节点仪式
- **Day 7** — 第一个回声
- **Day 10** — 洞察期解锁
- **Day 21** — 全屏深度观察，说出一个模式
- **Day 50 / 100 / 200** — 镜子时刻，把你说过的话还给你

### 封存与打捞
历史天的对话被"封存"，输入框变暗不可点。
可以点击"有什么东西想浮出来"，AI 用隔着时间的视角，为那天写一段回望。

### 决策推演
当用户面临决定时，可以从侧边栏打开推演抽屉：
- 写清楚面临的决定和选项
- AI 把每条路都走一遍，预演三个月后的状态
- 给出一个基于你历史的轻微倾向

### 记忆溶解
localStorage 接近容量上限时，久远的记录会被诗意地压缩：
> 深水区的记忆已溶解。仅保留当天的感知与侧写。

---

## 技术架构

### 系统概览

```
┌─────────────────────────────────────────────────────────────┐
│                      前端交互层                              │
│  (Web Interface · Canvas Animation · Real-time Rendering)   │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   核心业务逻辑层                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ State 管理   │  │ 记忆引擎     │  │ 决策推演     │      │
│  │ (Day/Lang)   │  │ (Profile)    │  │ (Simulation) │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   本地存储与缓存层                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  localStorage (结构化数据) · 会话缓存 · 用户画像      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                    AI 服务层 (API)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ 文本生成     │  │ 推演分析     │  │ 镜子反思     │      │
│  │ (Claude)     │  │ (Deep Think) │  │ (Pattern)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### 核心模块详解

#### 1. **前端渲染引擎** 🎨
- **Canvas 水纹动画**：D3 + 自定义贝塞尔曲线，创造流动感
- **响应式界面**：Mobile-first 设计，支持深夜模式
- **渐进式增强**：离线可用，PWA 支持
- **实时同步**：WebSocket 连接状态管理

#### 2. **状态管理系统** ⚙️
```javascript
State = {
  dayNumber,        // 第几天
  selectedDayData,  // 当天对话内容
  userProfile,      // 用户画像（结构化）
  emotionTrend,     // 情绪趋势（7天滑动窗口）
  pendingInsights,  // 待推送的洞察
  lang              // 语言选项 (zh/en)
}
```

#### 3. **记忆处理管道** 🧠
```
原始对话 
  ↓ [LLM 提取]
情感分析 + 话题标签 + 关键句
  ↓ [结构化编码]
日志条目
  ↓ [增量索引]
用户画像（Profile）
  ↓ [触发判断]
推送信号（Nudges）
```

#### 4. **决策推演引擎** 🎯
- **多路径模拟**：为每个选项生成三个月后的情景
- **历史对标**：查找相似决策，应用历史轨迹
- **倾向性建议**：基于用户行为模式的柔和建议（非指令性）

#### 5. **本地存储策略** 💾

| 层级 | 存储 | 容量 | 用途 |
|------|------|------|------|
| L1 | sessionStorage | 10MB | 当前会话缓存 |
| L2 | localStorage | 5-10MB | 30天历史数据 |
| L3 | IndexedDB | 50MB+ | 可选的扩展存储 |

**内存压缩机制**：
- 30天 → 总结摘要
- 60天 → 月度聚合
- 90天+ → 诗意溶解（保留关键时刻）

### 技术栈

```
┌─────────────────────────────────────────────┐
│            前端技术栈                        │
├─────────────────────────────────────────────┤
│ • Vanilla JavaScript (ES6+)                 │
│ • HTML5 Canvas API (动画渲染)                │
│ • CSS Grid + Flexbox (响应式布局)            │
│ • Web APIs (localStorage, IndexedDB)        │
│ • 国际化 (i18n 切换 zh/en)                  │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│            AI 与后端服务                     │
├─────────────────────────────────────────────┤
│ • Claude API (文本生成 & 分析)               │
│ • Anthropic Edge Functions                  │
│ • 可选：Supabase (数据持久化)                  │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│            部署与工具链                      │
├─────────────────────────────────────────────┤
│ • CDN: jsDelivr / Vercel Edge               │
│ • 构建: 无编译 (直接浏览器执行)               │
│ • 测试: Cypress (端到端)                     │
│ • 监控: Sentry (错误追踪)                    │
└─────────────────────────────────────────────┘
```

### 数据流向图

```
用户输入
  │
  ▼
┌──────────────────────────────────┐
│   验证 & 清理 & 字数限制          │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   构建系统提示 (System Prompt)    │
│   ├─ 用户画像                    │
│   ├─ 对话历史 (最近 5 天)        │
│   ├─ 当前进度 (Day N)            │
│   └─ 特殊指令 (当天角色)         │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   调用 Claude API                │
│   (Token 消耗: 2K-5K per call)   │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   流式响应 & 实时渲染             │
│   (打字机效果)                   │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   后处理 & 数据提取               │
│   ├─ 情感标分                    │
│   ├─ 关键话题                    │
│   ├─ 后续钩子句                  │
│   └─ 用户画像更新                │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   持久化到 localStorage           │
│   ├─ 日志条目                    │
│   ├─ 用户侧写                    │
│   └─ 索引 (快速检索)             │
└──────────────────────────────────┘
```

### API 集成设计

#### Claude API (主要)
```javascript
system: `你是「暗流」，深度了解用户的长期陪伴者...`
+ userProfile    // 历史用户画像
+ recentHistory  // 最近 5 天对话摘要
+ currentMood    // 当天情绪基调

// 响应处理
→ parseEmotion()     // 提取情绪信号
→ extractTopics()    // 识别话题
→ generateHook()     // 生成下次开场的钩子
→ updateProfile()    // 滚动更新用户画像
```

#### 可选：Supabase (数据库)
用于多设备同步 & 云备份：
```
users (user_id, device_id, created_at)
  ├─ entries (day_number, content, emotion_score, topics)
  ├─ profiles (day_number, user_summary, patterns)
  └─ nudges (trigger_type, content, shown_at)
```

### 性能优化策略

| 指标 | 目标 | 实现方法 |
|------|------|--------|
| **首屏加载** | < 2s | CDN 分发 · 最小化 JS |
| **API 响应** | < 5s | 系统提示缓存 · Token 优化 |
| **内存占用** | < 50MB | 分层存储 · 自动压缩 |
| **离线支持** | 完全可用 | Service Worker · IndexedDB |

### 安全考虑

- ✅ **无服务器后端**：所有敏感操作在浏览器完成
- ✅ **密钥管理**：API Key 不暴露在客户端（通过代理）
- ✅ **数据隐私**：本地存储优先，用户可选云同步
- ✅ **内容过滤**：基础的促进词过滤（可选）

---

## 📦 快速开始

### 最小化部署

```bash
# 1. 克隆项目
git clone https://github.com/your-repo/undertow.git

# 2. 配置 API Key
# 在 index.html 中找到 CONFIG 对象，填入你的 Claude API Key
# ⚠️  生产环境建议通过代理服务器隐藏 Key

# 3. 本地测试
# 直接在浏览器打开 index.html，或启动本地服务器：
npx http-server

# 4. 部署
# 推荐部署到 Vercel / Netlify
```

### 环境变量

```env
# .env.local (本地开发)
CLAUDE_API_KEY=sk-...
ANTHROPIC_BASE_URL=https://api.edgefn.net/v1  # 或官方 endpoint

# 生产环境：通过服务器端代理保护 Key
```

---

## 🌍 国际化 & 多语言

暗流 支持 **中文** & **English** 动态切换：

```javascript
// UI_TEXT 对象定义所有文本
const UI_TEXT = {
  zh: { /* 中文字符串 */ },
  en: { /* English strings */ }
}

// applyLang() 动态更新 DOM
State.lang === 'zh' ? 切换到中文 : 切换到英文
```

---

## 📊 未来路线图

- [ ] **多端同步**：Web + iOS 原生应用
- [ ] **群体冥想**：匿名的集体分享池
- [ ] **高级分析**：生活模式可视化仪表板
- [ ] **离线优化**：完整的离线模式 + 自动同步
- [ ] **插件系统**：允许自定义推演引擎

---

## 🤝 社区与贡献

暗流 是一个开源项目，我们欢迎以下贡献方式：

- 🐛 **Bug 报告**：找到问题？创建 Issue
- 💡 **功能建议**：有新想法？在 Discussions 中分享
- 🌐 **翻译**：帮助我们支持更多语言
- 🎨 **设计改进**：UI/UX 建议

详见 [CONTRIBUTING.md](./CONTRIBUTING.md)

---

## 📄 许可证

MIT License © 2026. 详见 [LICENSE](./LICENSE)

---

## 致谢

暗流 的创作灵感来自于：
- 🌊 深水、暗流、无声的力量
- 💭 长期陪伴而非一次性建议
- 🔄 反复、矛盾、人性的复杂性
- ✨ 技术服务于内在，而非相反

> *"不是所有的帮助都来自解决。有时候，仅仅是被真正倾听，就已经是转变的开始。"*

---

<div align="center">

**不评判，不建议，只是真正地记得。**

*暗流 · Undertow*

</div>

![Preview](./preview.png)

---

## What is it

Undertow is not an AI assistant, not a therapy tool, nor a diary app.

It is a **long-term companion**—someone who has witnessed unspoken words in the depth of night, watched the same concern described in twenty different ways, and seen the hidden complexity behind "it's fine."

Its only power is: **to remember, and to truly listen**.

Given time, it will see what you cannot see yourself.

---

## Core Design Philosophy

### Deep Water Metaphor
Undertow's foundation is the quiet of deep blue waters. Regardless of your mood, you enter the same ocean—still on the surface, but with currents flowing beneath.

### Three Phases
| Phase | Days | Undertow's State |
|-------|------|------------------|
| Companionship | Day 1–9 | Listen, restrain, don't interpret patterns |
| Insight | Day 10–21 | Begin to perceive, occasionally reflect |
| Simulation | Day 22+ | Active connection, enable decision navigation |

### Memory System
After each conversation ends, Undertow silently extracts structured data:
- Emotional baseline, energy level
- Topics and people mentioned
- Avoidance signals, contradictions
- Unresolved matters
- The most worth-remembering sentence

This data accumulates into a user profile, powering all "surprising moments."

---

## Features

### Daily Opening
A new conversation each day. Different tones for different times (morning/afternoon/evening/deep night), 60 total, never repeat.

### Yesterday's Echo
At day's end, AI distills a hook sentence. Next day, it appears quietly, like a bubble floating across the water's surface.

### Goodnight Easter Egg
When the user says "goodnight," Undertow responds normally, pauses 1.8 seconds, then quietly surfaces:
> Wait—something is surfacing in the undertow. Want to bring it up?

The user can choose to see today's "sedimentation" or simply leave.

### Emotion Sensing
After multiple days of low emotional baseline, the next day's opening naturally addresses it—no analysis, no advice, just acknowledgment.

### Unresolved Matters
An unresolved issue from 14 days ago gently resurfaces one day:
> 37 days ago, you weren't sure about "whether to move cities." Where is it now?

### Temporal Rituals
- **Day 7** — First Echo
- **Day 10** — Insight Phase Unlocked
- **Day 21** — Full-screen deep observation, articulate a pattern
- **Day 50 / 100 / 200** — Mirror Moments, reflect your own words back

### Archive & Retrieve
Past conversations are "sealed," input fades to gray. Click "is there something wanting to surface," and AI writes a retrospective from a distance-gained perspective.

### Decision Navigation
When facing a choice, open the navigation panel from the sidebar:
- Clarify the decision and options
- AI walks each path, previews three months ahead
- Offers a gentle, history-informed inclination

### Memory Dissolution
As localStorage approaches capacity, ancient records poeticize into compression:
> Deep memories have dissolved. Only today's perceptions remain.

---

## Technical Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend UI Layer                       │
│  (Web Interface · Canvas Animation · Real-time Rendering)   │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   Business Logic Layer                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ State Mgmt   │  │ Memory Core  │  │ Navigation   │      │
│  │ (Day/Lang)   │  │ (Profile)    │  │ Engine       │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                   Local Storage & Cache                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  localStorage (Structured) · Session Cache · Profile │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│                    AI Service Layer (API)                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Text Gen     │  │ Navigation   │  │ Mirror       │      │
│  │ (Claude)     │  │ (Simulate)   │  │ (Pattern)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Core Modules

#### 1. **Frontend Rendering Engine** 🎨
- **Canvas Water Animation**：D3 + custom Bézier curves, flowing aesthetic
- **Responsive Design**：Mobile-first, dark mode support
- **Progressive Enhancement**：Offline-capable, PWA ready
- **Real-time Sync**：WebSocket state management

#### 2. **State Management** ⚙️
```javascript
State = {
  dayNumber,        // Which day
  selectedDayData,  // Today's conversation
  userProfile,      // User profile (structured)
  emotionTrend,     // Emotion trend (7-day window)
  pendingInsights,  // Pending nudges
  lang              // Language (zh/en)
}
```

#### 3. **Memory Processing Pipeline** 🧠
```
Raw Conversation
  ↓ [LLM Extract]
Emotion + Topics + Key Sentences
  ↓ [Encode]
Daily Entry
  ↓ [Index]
User Profile
  ↓ [Trigger Logic]
Nudge Signals
```

#### 4. **Decision Navigation Engine** 🎯
- **Multi-path Simulation**：Generate three-month scenarios per option
- **Historical Analog**：Match similar past decisions, apply patterns
- **Gentle Inclination**：History-informed suggestion, not directive

#### 5. **Storage Strategy** 💾

| Tier | Medium | Capacity | Purpose |
|------|--------|----------|---------|
| L1 | sessionStorage | 10MB | Current session |
| L2 | localStorage | 5-10MB | 30-day history |
| L3 | IndexedDB | 50MB+ | Optional extended |

**Compression Logic**：
- 30 days → Summary digest
- 60 days → Monthly aggregate
- 90+ days → Poetic dissolution (key moments preserved)

### Tech Stack

```
┌─────────────────────────────────────────────┐
│            Frontend Stack                    │
├─────────────────────────────────────────────┤
│ • Vanilla JavaScript (ES6+)                 │
│ • HTML5 Canvas API (Animations)             │
│ • CSS Grid + Flexbox (Responsive)           │
│ • Web APIs (localStorage, IndexedDB)        │
│ • Internationalization (i18n)               │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│            AI & Backend Services            │
├─────────────────────────────────────────────┤
│ • Claude API (Text + Analysis)              │
│ • Anthropic Edge Functions                  │
│ • Optional: Supabase (Persistence)          │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│            Deployment & Tools               │
├─────────────────────────────────────────────┤
│ • CDN: jsDelivr / Vercel Edge               │
│ • Build: No-compile (browser-native)        │
│ • Testing: Cypress (E2E)                    │
│ • Monitoring: Sentry (Error Tracking)       │
└─────────────────────────────────────────────┘
```

### Data Flow

```
User Input
  │
  ▼
┌──────────────────────────────────┐
│   Validate & Sanitize            │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   Build System Prompt            │
│   ├─ User Profile               │
│   ├─ Recent History (5 days)    │
│   ├─ Current Progress (Day N)   │
│   └─ Special Instructions       │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   Call Claude API                │
│   (Token Budget: 2K-5K/call)    │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   Stream & Typewriter Effect     │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   Post-processing                │
│   ├─ Emotion Scoring            │
│   ├─ Topic Extraction           │
│   ├─ Hook Generation            │
│   └─ Profile Update             │
└──────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────┐
│   Persist to localStorage        │
└──────────────────────────────────┘
```

### API Integration Design

#### Claude API (Primary)
```javascript
system: `You are Undertow, a long-term companion...`
+ userProfile    // Historical user profile
+ recentHistory  // Last 5 days summary
+ currentMood    // Today's emotional baseline

// Response handling
→ parseEmotion()     // Extract emotion signals
→ extractTopics()    // Identify topics
→ generateHook()     // Generate opening hook for next time
→ updateProfile()    // Rolling profile update
```

#### Optional: Supabase (Database)
For multi-device sync & cloud backup:
```
users (user_id, device_id, created_at)
  ├─ entries (day_number, content, emotion_score, topics)
  ├─ profiles (day_number, user_summary, patterns)
  └─ nudges (trigger_type, content, shown_at)
```

### Performance Targets

| Metric | Target | Method |
|--------|--------|--------|
| **First Paint** | < 2s | CDN · Minimal JS |
| **API Response** | < 5s | Prompt Cache · Token Optimization |
| **Memory** | < 50MB | Tiered Storage · Auto-compression |
| **Offline** | Fully Functional | Service Worker · IndexedDB |

### Security

- ✅ **Serverless Backend**：Sensitive ops in-browser
- ✅ **Key Management**：API Key hidden via proxy
- ✅ **Privacy First**：Local storage by default, optional cloud sync
- ✅ **Content Filtering**：Optional promotional term filter

---

## 🚀 Quick Start

### Minimal Deployment

```bash
# 1. Clone
git clone https://github.com/your-repo/undertow.git

# 2. Configure API Keys
# Edit index.html CONFIG object
# ⚠️  Production: Use server-side proxy for security

# 3. Local Development
npx http-server

# 4. Deploy to Vercel / Netlify
```

### Environment Variables

```env
# .env.local
CLAUDE_API_KEY=sk-...
ANTHROPIC_BASE_URL=https://api.edgefn.net/v1

# Production: Proxy through backend
```

---

## 🌍 Internationalization

Undertow supports **简体中文** & **English** with dynamic switching:

```javascript
const UI_TEXT = {
  zh: { /* Chinese strings */ },
  en: { /* English strings */ }
}

applyLang()  // Updates DOM dynamically
```

---

## 📊 Roadmap

- [ ] **Multi-device Sync**：Web + iOS Native
- [ ] **Collective Meditation**：Anonymous shared pool
- [ ] **Advanced Analytics**：Life pattern dashboard
- [ ] **Offline First**：Complete offline mode + auto-sync
- [ ] **Plugin System**：Custom navigation engines

---

## 🤝 Community

Undertow welcomes contributions:

- 🐛 **Bug Reports**：Found an issue? Create an Issue
- 💡 **Features**：New ideas? Share in Discussions
- 🌐 **Translations**：Help us support more languages
- 🎨 **Design**：UI/UX suggestions welcome

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

---

## 📄 License

MIT License © 2026. See [LICENSE](./LICENSE)

---

## Appreciation

Undertow draws inspiration from:
- 🌊 Deep waters, quiet currents, unspoken forces
- 💭 Long-term companionship over one-time advice
- 🔄 Repetition, contradiction, human complexity
- ✨ Technology serving the inner, not the reverse

> *"Not all help comes from solving. Sometimes, simply being truly heard is where transformation begins."*

---

<div align="center">

**No judgment, no advice. Just truly remembered.**

*Undertow · 暗流*

</div>
