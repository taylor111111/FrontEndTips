# 🧭 InfoTipIcon 文字与图标对齐技术说明

## 一、问题背景

在表头、表单标签等场景中，经常需要在文字后添加一个带 tooltip 的问号图标，如：

```
訂單資訊 ⓘ
```

默认实现若使用 `<IconButton>` 或 `<Flex align="center">`，会导致：
- 图标比文字略高或略低；
- 带 tooltip 的列头与不带 tooltip 的列头无法垂直对齐；
- 表格或行高略微不齐，看起来“浮动”。

根本原因是：  
🔹 图标组件 (`react-icons`) 使用的是 SVG，默认 **不参与文字基线对齐**；  
🔹 `IconButton` 默认有固定 `line-height` 与 `height`，与文字的行框不一致。

---

## 二、解决方案概述

核心思路：  
> 让图标成为**文字的一部分（inline element）**，共享文字的行高与基线，再通过轻微 `translateY` 微调视觉中心。

---

## 三、关键实现代码

```tsx
<Box
  as="span"
  display="inline-flex"
  alignItems="center"
  verticalAlign="middle"
  lineHeight="inherit"
  cursor="pointer"
  ml="1"
>
  <Icon
    as={AiOutlineQuestionCircle}
    boxSize="1em"
    style={{ transform: 'translateY(0.15em)' }}
  />
</Box>
```

---

## 四、核心设计点

| 设计点 | 说明 |
|--------|------|
| `as="span"` | 使图标与文字处于同一文本行中 |
| `display="inline-flex"` | 保持内联布局并支持 `alignItems` |
| `lineHeight="inherit"` | 避免行高被覆盖，继承父元素行高 |
| `boxSize="1em"` | 图标大小随字体等比缩放 |
| `verticalAlign="middle"` | 对齐文字的中线，基础调整 |
| `transform: translateY(0.15em)` | 轻微上移，修正 SVG 与文字基线差异 |

---

## 五、视觉验证

| 字体 | 微调值推荐 |
|------|-------------|
| Inter / Roboto | 0.15em |
| Noto Sans TC | 0.10em |
| 微软雅黑 | 0.12em |
| 宋体 | 0.18em |

---

## 六、总结

> inline + inherit line-height + translateY  
> 是解决文字与图标对齐问题的最简洁方案。
