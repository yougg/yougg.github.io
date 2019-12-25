---
Categories: []
Description: ""
Tags: []
title: "JLabel 文本内容自动换行"
date: 2013-03-13T00:00:00+08:00
draft: false
---

```java
JLabel content = new JLabel("<HTML>JLabel 文字内容自动换行测试.JLabel 文字内容自动换行测试.</HTML>");
View labelView = BasicHTML.createHTMLView(content, content.getText());
int labelWidth = 200;
int labelHeight = 100;
labelView.setSize(labelWidth, labelHeight);
// 根据限定宽度计算文本内容换行后的真实高度.
int trueHight = (int) labelView.getMinimumSpan(View.Y_AXIS);
content.setPreferredSize(new Dimension(labelWidth, trueHight));
```
