# 🎯 图标与文字的基线对齐：从视觉误差到完美修正

## 一、引言

“为什么我的问号图标总感觉比文字低半个像素？”  
这个问题几乎困扰过所有注重细节的前端。

---

## 二、问题起源

文字基于**排版基线（baseline）**对齐，  
而 SVG 图标基于**几何中心（bounding box center）**对齐。  
两者不在同一水平线上，于是出现了“掉下去”的错觉。

---

## 三、解决方案

通过 `translateY` 对图标进行轻微上移：

```tsx
<Box as="span" display="inline-flex" alignItems="center" verticalAlign="middle" lineHeight="inherit">
  <Icon as={AiOutlineQuestionCircle} boxSize="1em" style={{ transform: 'translateY(0.15em)' }} />
</Box>
```

- `boxSize="1em"`：图标高度随字号变化；  
- `translateY(0.15em)`：向上微调，修正几何中心与基线的差距。

---

## 四、调节建议

| 字体 | translateY 推荐值 |
|------|------------------|
| Inter / Roboto | 0.15em |
| Noto Sans TC | 0.10em |
| 微软雅黑 | 0.12em |
| 宋体 | 0.18em |

---

## 五、核心结论

> 这不是几何问题，而是基线问题。  
> `translateY` 是让 SVG 重新回到排版秩序中的桥梁。

---

## 六、视觉的温度

修正 0.15em 的误差，往往带来肉眼可感的平衡感。  
这种细节，就是前端界面设计的温度。
