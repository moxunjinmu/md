# 社区主题收集

> 从 GitHub Discussion #426 提取的社区优秀主题

## 1. 童趣彩虹主题（rainbow）

**作者**：社区贡献  
**风格**：活泼可爱的儿童风格，彩虹色彩搭配  
**适用**：教育、亲子、儿童相关内容  
**特点**：
- 彩虹色彩搭配
- 淡蓝网格背景
- 支持深色模式
- 丰富的 GFM Alert 样式

**CSS 代码**：

```css
/**
 * 童趣彩虹主题（rainbow）
 * 活泼可爱的儿童风格，彩虹色彩搭配，适合教育、亲子、儿童相关内容
 * 注：尽量遵循现有系统变量；如未提供二级/强调色，使用固定色确保观感
 * 支持深色模式
 */

/* 背景：淡蓝网格（浅色模式） */
.md-container {
  background-color: hsl(var(--background));
  background-image:
    linear-gradient(to right, rgba(135, 206, 235, 0.12) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(135, 206, 235, 0.12) 1px, transparent 1px);
  background-size: 24px 24px;
  background-position: 0 0;
}

/* 深色模式下的背景网格 */
.dark .md-container {
  background-image:
    linear-gradient(to right, rgba(135, 206, 235, 0.08) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(135, 206, 235, 0.08) 1px, transparent 1px);
}

/* 调整基础排版氛围 */
p {
  line-height: 1.8;
  color: hsl(var(--foreground));
}

/* ===== 标题 ===== */
h1 {
  font-size: calc(var(--md-font-size) * 1.4);
  font-weight: 800;
  color: #ff69b4; /* 粉色 */
  margin: 30px 8px 25px;
  letter-spacing: 0.5px;
}

h2 {
  padding: 0.3em 1.2em;
  font-size: calc(var(--md-font-size) * 1.3);
  border-radius: 8px 124px 8px 24px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.06);
}

h3 {
  font-size: calc(var(--md-font-size) * 1.1);
  font-weight: 700;
  color: #2e8b57; /* 绿色文字 */
  margin: 20px 8px 15px;
  padding: 6px 0 6px 14px;
  border-left: 4px solid #ffd700; /* 金黄色左边框 */
}

/* ... 更多样式见完整 CSS ... */
```

**预览图**：
![童趣彩虹主题](https://private-user-images.githubusercontent.com/71597859/526177453-ac12ead6-cc9a-4f05-afa2-cbc73588c3af.png)

---

## 2. 其他主题（待补充）

从 Discussion #426 中还有其他优秀主题，需要进一步提取和整理。

---

**更新时间**：2026-03-07 04:15  
**来源**：https://github.com/doocs/md/discussions/426
