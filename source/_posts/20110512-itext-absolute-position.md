---
layout: post
title: itext中文本的绝对位置放置
date: 2011-05-12 12:02:00
description: itext中文本的绝对位置放置
tags: [Java]
urlname: itext-absolute-position
---

text中表格可以使用

	table.writeSelectedRows(0, -1, x, y, writer.getDirectContent());

来实现绝对位置放置。那文本段落呢... 

当我们不需要itext对每个单词、句子、段落实现自动格式的时候，或是想使用特殊布局，就可以使用PdfContentByte来实现绝对位置放置。

PdfContentByte的初始化：

	PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream("test.pdf"));  
	PdfContentByte cb = writer.DirectContent;

将文本写入ContentByte中时，必须使用方法beginText()和endText()，同时也必须设置字体和尺寸。有两种方法来写入和放置文本。

**方法一：**

	BaseFont bf = BaseFont.createFont(BaseFont.HELVETICA, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);  cb.beginText();  
	cb.setFontAndSize(bf, 12);  
	cb.showTextAligned(PdfContentByte.ALIGN_CENTER, text + "This text is centered", 250, 700, 0);  
	cb.endText();

**方法二：**

	BaseFont bf = BaseFont.createFont(BaseFont.HELVETICA, BaseFont.CP1252, BaseFont.NOT_EMBEDDED);  
	cb.beginText();  
	cb.setFontAndSize(bf, 12);  
	cb.setTextMatrix(100, 400);  
	cb.showText("Text at position 100,400.");  
	cb.endText(); 
