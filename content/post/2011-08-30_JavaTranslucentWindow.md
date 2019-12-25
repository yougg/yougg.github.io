---
Categories: []
Description: ""
Tags: []
title: "Java透明窗口的例子"
date: 2011-08-30T00:00:00+08:00
draft: false
---

```java
public class TranslucentWindow extends JFrame
{
	public TranslucentWindow()
	{
		super("TranslucentWindow");
		setLayout(new GridBagLayout());
		setSize(300, 200);
		setLocationRelativeTo(null);
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		add(new JButton("I am a Button"));

		//设置窗口的透明度为 55%
		AWTUtilities.setWindowOpacity(this, 0.55f);
	}

	public static void main(String[] args)
	{
		//如果当前JDK不支持透明则退出
		if (!AWTUtilities.isTranslucencySupported(AWTUtilities.Translucency.TRANSLUCENT))
		{
			System.err.println("Translucency is not supported");
			System.exit(0);
		}

		TranslucentWindow tw = new TranslucentWindow();
		tw.setVisible(true);
	}
}
```
