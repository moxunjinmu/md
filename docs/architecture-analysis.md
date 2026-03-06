# 项目架构分析

> 微信 Markdown 编辑器（doocs/md）深度分析

**分析时间**：2026-03-07 04:43  
**分析者**：莫码 (moma)  
**项目版本**：2.1.0

---

## 1. 项目整体架构

### 1.1 技术栈

- **前端框架**：Vue 3 + TypeScript
- **构建工具**：Vite 7.3.1
- **包管理器**：pnpm（workspace）
- **样式**：Less
- **状态管理**：Pinia
- **编辑器**：CodeMirror v6

### 1.2 Monorepo 结构

```
md/
├── apps/
│   └── web/              # 主应用
│       ├── src/
│       │   ├── components/   # 组件
│       │   ├── stores/       # 状态管理
│       │   ├── assets/       # 静态资源
│       │   └── utils/        # 工具函数
│       └── wxt.config.ts     # WXT 配置（浏览器扩展）
├── packages/
│   ├── core/             # 核心功能
│   │   └── src/theme/    # 主题系统
│   ├── shared/           # 共享配置
│   │   └── src/configs/  # 配置文件
│   │       └── theme-css/  # CSS 主题
│   └── config/           # 构建配置
└── public/               # 公共资源
```

---

## 2. 主题系统详解

### 2.1 主题架构

**三层架构**：

1. **配置层**（`packages/shared/src/configs/`）
   - `theme.ts` - 主题选项定义
   - `theme-css/` - CSS 主题文件

2. **核心层**（`packages/core/src/theme/`）
   - `themeApplicator.ts` - 主题应用器
   - `themeExporter.ts` - 主题导出器
   - `themeInjector.ts` - 主题注入器

3. **应用层**（`apps/web/src/`）
   - `stores/theme.ts` - 主题状态管理
   - `assets/less/theme.less` - 主题样式

### 2.2 主题文件结构

```
packages/shared/src/configs/theme-css/
├── index.ts          # 主题导出（ES Module）
├── base.css          # 基础样式（所有主题共享）
├── default.css       # 经典主题
├── grace.css         # 优雅主题（@brzhang）
└── simple.css        # 简洁主题（@okooo5km）
```

### 2.3 主题定义

**theme.ts**：
```typescript
export const themeOptions = [
  { label: `经典`, value: `default`, desc: `` },
  { label: `优雅`, value: `grace`, desc: `@brzhang` },
  { label: `简洁`, value: `simple`, desc: `@okooo5km` },
]
```

**theme-css/index.ts**：
```typescript
export const themeMap = {
  default: defaultCSS,
  grace: graceCSS,
  simple: simpleCSS,
} as const

export type ThemeName = keyof typeof themeMap
```

### 2.4 主题加载流程

```
1. 用户选择主题
   ↓
2. useThemeStore.theme 更新
   ↓
3. applyTheme() 调用
   ↓
4. themeInjector 注入 CSS
   ↓
5. 预览区域应用新样式
```

---

## 3. 关键组件分析

### 3.1 编辑器组件

**位置**：`apps/web/src/components/editor/`

**核心组件**：
- `CssEditor.vue` - CSS 自定义编辑器（16KB）
- `EditorStateDialog.vue` - 编辑器状态对话框（17KB）
- `RightSlider.vue` - 右侧滑动面板（14KB）
- `TemplateDialog.vue` - 模板对话框（12KB）
- `ThemeCustomizer.vue` - 主题定制器（4KB）

**编辑器头部**：
- `editor-header/` - 顶部工具栏组件

### 3.2 预览组件

**位置**：`apps/web/src/components/preview/`（推测）

**功能**：
- 实时渲染 Markdown
- 应用主题样式
- 支持深色模式

### 3.3 工具栏组件

**位置**：`apps/web/src/components/editor/editor-header/`

**功能**：
- 格式化工具
- 插入元素
- 主题切换
- 导出功能

---

## 4. 状态管理

### 4.1 主题 Store

**文件**：`apps/web/src/stores/theme.ts`

**管理的状态**：
```typescript
- theme: ThemeName              // 文本主题
- fontFamily: string            // 文本字体
- fontSize: string              // 文本大小
- primaryColor: string          // 主色
- codeBlockTheme: string        // 代码块主题
- legend: string                // 图注格式
- isMacCodeBlock: boolean       // Mac 代码块
- isShowLineNumber: boolean     // 代码行号
- isCiteStatus: boolean         // 外链引用
- isCountStatus: boolean        // 字数统计
- isUseIndent: boolean          // 首行缩进
- isUseJustify: boolean         // 两端对齐
- previewWidth: string          // 预览宽度
```

**持久化**：
- 使用 `store.reactive()` 自动持久化到 localStorage
- 前缀：`md-` 或自定义前缀

### 4.2 CSS 编辑器 Store

**文件**：`apps/web/src/stores/cssEditor.ts`

**功能**：
- 管理自定义 CSS
- 实时预览
- CSS 验证

---

## 5. 如何添加新主题

### 5.1 方法 A：添加内置主题（推荐）

**步骤**：

1. **创建 CSS 文件**
   ```bash
   # 位置：packages/shared/src/configs/theme-css/
   touch packages/shared/src/configs/theme-css/rainbow.css
   ```

2. **编写主题样式**
   ```css
   /* rainbow.css */
   h1 { color: #ff69b4; }
   h2 { color: #87ceeb; }
   /* ... */
   ```

3. **导出主题**
   ```typescript
   // packages/shared/src/configs/theme-css/index.ts
   import rainbowCSS from './rainbow.css?raw'
   
   export const themeMap = {
     default: defaultCSS,
     grace: graceCSS,
     simple: simpleCSS,
     rainbow: rainbowCSS,  // 新增
   } as const
   ```

4. **注册主题选项**
   ```typescript
   // packages/shared/src/configs/theme.ts
   export const themeOptions = [
     { label: `经典`, value: `default`, desc: `` },
     { label: `优雅`, value: `grace`, desc: `@brzhang` },
     { label: `简洁`, value: `simple`, desc: `@okooo5km` },
     { label: `童趣彩虹`, value: `rainbow`, desc: `社区贡献` },  // 新增
   ]
   ```

5. **测试**
   ```bash
   pnpm web dev
   # 访问 http://localhost:5173/md/
   # 切换主题测试
   ```

### 5.2 方法 B：自定义 CSS（用户级）

**步骤**：

1. 打开编辑器
2. 点击"自定义 CSS"
3. 粘贴 CSS 代码
4. 实时预览

**优点**：
- 无需修改代码
- 用户可自定义

**缺点**：
- 不能分享
- 不能预设

---

## 6. 深色模式支持

### 6.1 实现方式

**CSS 类名**：`.dark`

**示例**：
```css
/* 浅色模式 */
.md-container {
  background-color: #fff;
}

/* 深色模式 */
.dark .md-container {
  background-color: #191919;
}
```

### 6.2 切换逻辑

**位置**：`apps/web/src/stores/theme.ts`

**变量**：`isDark`

**切换**：
```typescript
const toggleDark = () => {
  isDark.value = !isDark.value
}
```

---

## 7. 定制建议

### 7.1 界面定制优先级

**高优先级**（影响大）：
1. ✅ 主题系统（添加 3+ 社区主题）
2. ✅ 主色调整（莫循品牌色）
3. ✅ 主题选择器优化

**中优先级**（提升体验）：
4. 导航栏优化（Logo、布局）
5. 编辑器优化（字体、行高）
6. 预览区优化（响应式）

**低优先级**（锦上添花）：
7. 动画效果
8. 快捷键提示
9. 移动端适配

### 7.2 技术方案

**方案 1：最小改动**
- 只添加主题 CSS 文件
- 不修改组件代码
- 快速上线

**方案 2：深度定制**
- 修改组件样式
- 调整布局结构
- 添加新功能

**推荐**：先执行方案 1，再逐步实施方案 2

### 7.3 品牌色集成

**莫循品牌色**：
- 主色：#FF6B35（日落橙）
- 辅色：#F7931E（金橙）
- 点缀：#FFD23F（暖黄）

**应用位置**：
- 主题主色（primaryColor）
- 链接颜色
- 按钮颜色
- 强调文本

---

## 8. 构建和部署

### 8.1 开发模式

```bash
pnpm web dev
# 访问 http://localhost:5173/md/
```

### 8.2 生产构建

```bash
pnpm web build
# 输出：apps/web/.output/public/
```

### 8.3 Docker 部署

```bash
# 构建镜像
docker build -t moxun/md-editor:latest .

# 运行容器
docker run -d -p 8080:80 moxun/md-editor:latest
```

---

## 9. 关键文件清单

### 9.1 主题相关

| 文件 | 功能 | 优先级 |
|------|------|--------|
| `packages/shared/src/configs/theme-css/` | CSS 主题文件 | ⭐⭐⭐ |
| `packages/shared/src/configs/theme.ts` | 主题选项配置 | ⭐⭐⭐ |
| `apps/web/src/stores/theme.ts` | 主题状态管理 | ⭐⭐ |
| `apps/web/src/assets/less/theme.less` | 主题样式 | ⭐⭐ |
| `packages/core/src/theme/` | 主题核心逻辑 | ⭐ |

### 9.2 组件相关

| 文件 | 功能 | 大小 |
|------|------|------|
| `apps/web/src/components/editor/CssEditor.vue` | CSS 编辑器 | 16KB |
| `apps/web/src/components/editor/EditorStateDialog.vue` | 状态对话框 | 17KB |
| `apps/web/src/components/editor/RightSlider.vue` | 右侧面板 | 14KB |
| `apps/web/src/components/editor/TemplateDialog.vue` | 模板对话框 | 12KB |
| `apps/web/src/components/editor/ThemeCustomizer.vue` | 主题定制器 | 4KB |

---

## 10. 下一步行动

### 10.1 立即执行（检查点 3-4）

1. **集成童趣彩虹主题**
   - 复制 CSS 到 `packages/shared/src/configs/theme-css/rainbow.css`
   - 更新 `index.ts` 和 `theme.ts`
   - 测试主题切换

2. **添加 2-3 个社区主题**
   - 从 Discussion #426 提取
   - 测试兼容性
   - 验证深色模式

### 10.2 后续优化（检查点 5-9）

3. **界面定制**
   - 导航栏：添加 Logo
   - 编辑器：优化字体
   - 预览区：响应式布局

4. **品牌色集成**
   - 调整主色为 #FF6B35
   - 更新链接颜色
   - 统一按钮样式

---

## 11. 风险和注意事项

### 11.1 技术风险

⚠️ **CSS 冲突**
- 新主题可能与现有样式冲突
- 解决：使用 CSS 优先级和 `!important`

⚠️ **深色模式兼容**
- 部分主题可能不支持深色模式
- 解决：为每个主题添加 `.dark` 样式

⚠️ **构建失败**
- CSS 文件导入可能失败
- 解决：确保使用 `?raw` 后缀

### 11.2 用户体验风险

⚠️ **主题过多**
- 选择困难
- 解决：分类展示，推荐默认

⚠️ **加载性能**
- CSS 文件过大
- 解决：按需加载，懒加载

---

## 12. 总结

### 12.1 项目特点

✅ **优点**：
- Monorepo 架构清晰
- 主题系统设计良好
- 支持自定义 CSS
- 深色模式完善

⚠️ **改进空间**：
- 主题数量较少（3 个）
- 缺少主题预览
- 品牌定制不足

### 12.2 定制难度

| 任务 | 难度 | 预计时间 |
|------|------|----------|
| 添加新主题 | ⭐ 简单 | 20 分钟 |
| 修改主色 | ⭐ 简单 | 10 分钟 |
| 优化组件 | ⭐⭐ 中等 | 1-2 小时 |
| 深度定制 | ⭐⭐⭐ 困难 | 4-8 小时 |

---

**分析完成时间**：04:55  
**用时**：12 分钟  
**下一步**：检查点 3 - 主题系统理解
