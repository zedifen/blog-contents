---
title: 关于 draw.io 矢量图 SVG 导出的研究
date: 2022-05-18 04:06:11
description: draw.io 是一个很简单易用的矢量图绘制工具, 画一些简单的图标或者流程图等等都很适合. 虽然好用, 但是在导出 *.drawio 文件为其他格式 (位图 PNG, 矢量图 SVG) 时, 大多数时候都需要额外注意.
tags: [drawio, SVG]
---

[draw.io](https://github.com/jgraph/drawio) 是一个简单易用的图表 (diagram) 绘制工具 [^app-page], 可以用作一个简单的矢量图绘制工具, 画一些简单的图标或者流程图等等都很适合.

虽然好用, 但是在导出 `*.drawio` 文件为其他格式时, 大多数时候都需要额外注意.

[^app-page]:  可以访问 [app.diagrams.net][drawio-app] 在线使用, 也有很多平台提供 draw.io 的换皮版本.

## 导出设计时会遇到的问题及原因

对于 PNG 导出, 默认的设置下, 导出的图像会很模糊.  幸运的是, 可以在导出时通过将图表的缩放比例调制 250% ~ 300% (甚至更高) 来解决. [^png-export]


至于 SVG, 事情就变得有点复杂. 在浏览器中查看软件导出的 SVG 文件时, 一切似乎都很好. 但是, 当尝试将这些 SVG 导入 Word 或 Inkscape 等软件时，则可能会看到图片上出现 "**Viewer does not support full SVG 1.1** (查看器不支持完整的 SVG 1.1)" 字样 [^info-updated].


[^info-updated]: 最新版中, 文本已更改为 "Text is not SVG - cannot display", 即 "文本不是 SVG - 无法显示".


<figure>
<p>图片暂时省略</p>
<figcaption>图 1: 在 draw.io 中绘制的图片</figcaption>
</figure>


<figure>
<p>图片暂时省略</p>
<figcaption>图 2: 显示异常的图片</figcaption>
</figure>


大多数情况下, 这种情况是因为 draw.io 使用了 HTML 代码来标记文本框中的自动换行的长文本 / 复杂格式的文本 / (或者还有其他的图元), 并将这些代码直接嵌入到了 SVG 的 `<foreignObject>` 标签中. 

现代浏览器自然是能够解析这些 HTML 代码的, 因此在其中查看不会出现太多的问题. 然而这些 "外来 (foreign)" 的 HTML 代码并不能被 Inkscape 等专门处理 SVG 的软件理解.

因此, draw.io 的开发人员在 SVG 中加入了一个 `<switch>` 标签, 使得不支持 `<foreignObject>` 中内容的查看器在渲染这样的 SVG 时, 会显示上述的提示信息文本.

总之, 并不是这些查看器 "不支持 SVG 1.1", 而是因为这些 SVG 中 `<foreignObject>` 部分包含了不能被浏览器 (或者其他能渲染 HTML 的程序) 之外的程序所解析的内容.

> 另一方面, 尽管浏览器能够查看这些 SVG, 但是对于浏览器来说, 事实上 SVG 代码才是 "外来 (foreign)' 之物. [^svg-foreign-to-web-browser]

至于目前 draw.io 对于这长文本 / 复杂格式文本的 SVG 后备方案, draw.io 只是输出了一段修剪过后的文本, 十分没用.

[^png-export]: 参见 [Export a diagram as a higher resolution PNG image : draw.io is becoming diagrams.net](https://desk.draw.io/support/solutions/articles/16000059691-export-a-diagram-as-a-higher-resolution-png-image). 

[^svg-foreign-to-web-browser]: 引用自 [Inkscape 论坛][inkscape-fourm] 中 [@Xav 的回答](https://inkscape.org/forums/other/inkscape-viewer-does-not-support-full-svg-11/#c32375).

不过, 按照 draw.io 的一位开发人员的说法, 其实 draw.io 这个项目最开始并不是为了画图, 更没有导出 SVG 的选项; 就连现在, 其定位也不是一个矢量图绘制软件. 所以, 对于开发者来说, 上述这些, 并不能算是软件的 BUG, 甚至不用去理会.

## 解决方案

### 法 1: 禁用换行与文本格式

目前, 官方博客中给出的解决方法, 就是手动全选图表上的元素, 在 "格式面板 (Format Pannel)" 的 "**Text** (文本)" 选项卡下, **手动禁用**掉 "**Word Wrap** (自动换行)" 和 "**Formatted Text** (格式化文本)". [^disable-text-options] 

这样处理之后, draw.io 导出的 SVG 就会干干净净的了 (不含 `<foreignObject>`, 可以被 Word 等软件正常读取). 

需要注意的是:

- 禁用 "自动换行" 后, 之前适应文本框大小自动换行的文本将变为一行, 需要手动键入换行符将长文本分成多行;
- 禁用 "格式化文本" 后, 一个文本框中将不支持多种格式的文本, 必须以一个文本框为单位整体调整文本的样式.

[^disable-text-options]: 参见 [Why text in exported SVG images may not display correctly (diagrams.net)](https://www.diagrams.net/doc/faq/svg-export-text-problems)

每次手动禁用这些选项可能会有点麻烦, 可以直接 "禁用 `<foreignObject>` 的使用":

- 如果是在线使用 [app.diagrams.net][drawio-app], 可以直接访问 [app.diagrams.net/#\_CONFIG\_UzV3UjUyyk0tSk8F0qrGjqpggeLM3IKcVJ/EpNScYoh4SVFpqqq5CxABAA==](https://app.diagrams.net/#_CONFIG_UzV3UjUyyk0tSk8F0qrGjqpggeLM3IKcVJ/EpNScYoh4SVFpqqq5CxABAA==), 直接配置对应的设置 [^click-to-config];
- 或者在软件的 "其他" 选项中选择 "配置 (Configuration)", 添加键值对 `"simpleLabels": true`, 结果如下:
   ```json
   {
     "simpleLabels": true
     // Other configurations...
   }
   ```

[^click-to-config]: 来自 [Inkscape 论坛][inkscape-fourm] 下的 [回复](https://inkscape.org/forums/other/inkscape-viewer-does-not-support-full-svg-11/#c32511)

[drawio-app]: https://app.diagrams.net

### 法 2: 导出 PDF 后再转换为 SVG

我们知道, PDF 中的数据一般是没有上下文语义的; 可以说, 它只包含各个元素的样式以及对应的位置, 是一种用来存储排版结果的格式, 并不会轻易改变. [^pdf]

[^pdf]: 可以参考视频 "[PDF 里, 到底都是些什么 - 哔哩哔哩](https://www.bilibili.com/video/BV1Mr4y1679f)".

由于浏览器能够渲染排版这样包含 HTML 的 SVG, 并且浏览器一般具有 (将排版后的页面内容) 导出为 (打印为) PDF 的功能 [^print-to-pdf]. 于是, 可以利用该功能, 将浏览器对该 SVG 的排版结果导出到 PDF 里, 再通过 Inkscape 从 PDF 中还原 SVG.

[^print-to-pdf]: 通常, 这个选项可以在上下文菜单的 "分享", "打印" / "另存为 PDF" 中找到.

PDF 中包含的一般是稳定的格式, 从中还原出的 SVG 自然也相对稳定 (可能由于字体因素产生些许失真).

不过, 经过生成 PDF 再复原的过程, 一般会丧失许多 SVG 的语义, 比如文本不再能够被连贯的复制出来; 如果生成 PDF 的过程进行了转曲的操作, 这些文本可能甚至不再能够被选取复制.

> 最新 draw.io 的 SVG 导出选项中可以修改 "文本设置", 有一个 "将文本标签 (label) 转换为 SVG" 的选项. 据开发者所说, 就是将该文件上传到他们的服务器, 以进行上述的转换.

### 小结

综上, 目前有这些可行的解决方案：

- 对于简单的 SVG 图表 (不包含长文本 / 复杂格式文本), 只需按照上面提到的方法操作即可; 
- 尝试 (使用浏览器) 将 SVG 导出为 PDF, 然后使用 Inkscape 从 PDF 生成 SVG.

## 其他资料

关于这个事情, 有一些相关的有意思的讨论可以围观:

- [Issue #774 · jgraph/drawio][issue-page];
- [Inkscape "Viewer does not support full SVG 1.1" - Using Inkscape with Other Programs - Inkscape Forum][inkscape-fourm] (还提到了 [XSLT](https://en.wikipedia.org/wiki/XSLT "XSLT - Wikipedia"))
- [Truly Headless Draw.io Exports - Hacker News (ycombinator.com)][hacker-news] (提到了关于 "导出", "SVG 暗黑模式适配" 等问题)

---




其他可能有用的文章:

- [Things you need to know about working with SVG in VS Code (freecodecamp.org)](https://www.freecodecamp.org/news/things-you-need-to-know-about-working-with-svg-in-vs-code-63be593444dd/)

- [How I use draw.io at the command line | Tom Donohue](https://tomd.xyz/how-i-use-drawio/) 以及下边的评论:

   - 用代码画图 / 代码形式的图表: [Structurizr](https://www.structurizr.com/);
   - 一个基于 Web 的 PlantUML 编辑器: <http://www.plantuml.com/plantuml>;
   - ...

- 处理已压缩的 `.drawio` 文件 (可以在 'properties' 中删除): [Text-editing a draw.io file exported as SVG with embedded drawing - Stack Overflow](https://stackoverflow.com/questions/46699903/text-editing-a-draw-io-file-exported-as-svg-with-embedded-drawing)

- [4. Multiline SVG Text - SVG Text Layout \[Book\] (oreilly.com)](https://www.oreilly.com/library/view/svg-text-layout/9781491933817/ch04.html); 

- [SVG element reference \| MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Element#svg_elements_by_category) 以及 [`<text>` 元素的文档](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/text);

[issue-page]: https://github.com/jgraph/drawio/issues/774
[inkscape-fourm]: https://inkscape.org/forums/other/inkscape-viewer-does-not-support-full-svg-11/
[hacker-news]: https://news.ycombinator.com/item?id=29753638


---

- [解决draw.io生成SVG矢量图导入Word显示有误的问题以及推荐几种SVG绘图方法 - CSDN](https://blog.csdn.net/Casperflip/article/details/110129607)
- [draw.io 导出 SVG 图片报错](https://zodiaclab.top/随笔/draw-io-导出-SVG-图片报错/)
