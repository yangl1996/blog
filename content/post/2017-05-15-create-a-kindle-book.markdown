---
date: "2017-05-15T17:11:20Z"
image: /content/images/2017/05/Bright_green_tree_-_Waikato.jpg
title: 制作 Kindle 电子书
---

在看一般的书的时候，Kindle 相比纸质书有压倒性的优势。之前我一直误认为 Amazon 对 Kindle 电子书的制作并不提供官方的公开支持，而是选择和大出版商合作。事实上，Kindle 电子书的文件格式相当开放，Amazon 也提供制作需要的工具链，而且用起来也非常方便。

#### 1. 准备源文件和工具

源文件方面，只需要准备文本和插图就可以。如果书的内容比较复杂，可能还会有视频、声音等素材。

工具方面，Amazon 提供了 `kindlegen`，用它就可以把源文件打包成 Kindle 可用的格式。`kindlegen` 可以在 Amazon 的[对应页面](https://www.amazon.com/gp/feature.html?docId=1000765211)下载。顺便，也建议准备好这些官方的[样例](http://www.amazon.com/gp/redirect.html/ref=amb_link_1?location=http://kindlegen.s3.amazonaws.com/samples.zip&token=321FBC360D6D2CE41E4ED829508B1F8017D89641&source=standards&pf_rd_m=ATVPDKIKX0DER&pf_rd_s=right-6&pf_rd_r=ZW6ZMPXKDNNV4GHVAHT7&pf_rd_r=ZW6ZMPXKDNNV4GHVAHT7&pf_rd_t=1401&pf_rd_p=72de5f3f-c02a-44aa-af85-3913a05078e3&pf_rd_p=72de5f3f-c02a-44aa-af85-3913a05078e3&pf_rd_i=1000765211)，一会儿一些文件直接在样例基础上修改就行。

#### 2. 处理文本

Kindle 书的文本、排版实际上是用 HTML（和 CSS）描述的，因此首要的就是把排版、文本都处理好，转换成 HTML。这一步怎么做要看源文件的格式。比如源文件是普通的 text，那需要在每一段加上 `<p></p>`。图片这个时候也要插进去，就像写一个简单的网页一样。Kindle 支持不少 HTML 和 CSS 的特性，例如图片可以用 `width="100%"` 修饰，实现类似单页插图的效果；再例如可以规定字体、文字大小之类的。完成之后可以用一般的浏览器预览下，预览的效果如何，基本在 Kindle 就会显示成那样。一本 Kindle 书可以用很多 HTML 组成，后面会有地方规定这些 HTML 出场的顺序。

需要注意的是，对于中文文本（和所有希望用 UTF-8 编码的文本），要在文件开头用 `<?xml version="1.0" encoding="UTF-8" ?>` 注明编码。在 `<head>` 里用 `<meta charset="utf-8" />` 规定编码，Kindle 是不理的。

下面是 Kindle 书中一个 HTML 文件的例子。可以看到，章节标题、图片位置之类的都是人工设定的格式和位置。之前我以为这些 metadata 会有专门地方去写，然后 Kindle 自动排版，看来并不是这样。一个 Kindle 和一个 HTML 描述的页面没有本质上的区别，只是需要制作者对 Kindle 的屏幕（黑白、大小）做优化。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!doctype html>
<html lang="zh">
<head>
    <meta charset="utf-8"/>
    <title>第六·五章</title>
    <link rel="stylesheet" href="text.css"  type="text/css"/>
</head>
<body>
<h1>第六·五章 从毕业典礼经过三天后</h1>
<p>blah blah</p>
<img src="images/9.jpg" width="100%">
<p>blah blah</p>
<p>「感觉，你是不是，完全，弄错了呢……！」</p>
<div class="pagebreak"></div>
</body>
</html>
```

#### 3. 目录

Kindle 书的目录有两种，一个就是普通的 HTML，这个 HTML 和正文也没有本质的区别。还有一个是。`.ncx` 文件，事实上是个 XML，用来存放元数据，提供给 Kindle 用于在阅读界面中显示目录（章节、位置索引），就像下图就是根据 `.ncx` 来的。

![Kindle Table of Content](/content/images/2017/05/IMG_4043.JPG)

`.ncx` 文件里面会包含书名和一个个 `navPoint`，每个 `navPoint` 会包含条目的显示文字（`<text>`）、源文件位置（`<content>`）和顺序（`playOrder`）。下面是一个 `.ncx` 文件的例子。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE ncx PUBLIC "-//NISO//DTD ncx 2005-1//EN" 
   "http://www.daisy.org/z3986/2005/ncx-2005-1.dtd">
<ncx xmlns="http://www.daisy.org/z3986/2005/ncx/" version="2005-1">
  <head>
  </head>
  <docTitle>
    <text>路人女主的养成方法：第七卷</text>
  </docTitle>
<navMap>
  <navPoint id="toc" playOrder="1"><navLabel><text>目录</text></navLabel><content src="toc.html#toc"/></navPoint>
  <navPoint id="pr" playOrder="2"><navLabel><text>序章</text></navLabel><content src="62257.htm"/></navPoint>
  <navPoint id="ch1" playOrder="3"><navLabel><text>第一章 咦？照这样看，这次是不是走进个别剧情线了？</text></navLabel><content src="62258.htm"/></navPoint>
  <navPoint id="au" playOrder="15"><navLabel><text>后记</text></navLabel><content src="69021.htm"/></navPoint>
</navMap>
</ncx>
```

HTML 目录，写起来就自由很多，可以随意控制目录的显示效果。它也是不涉及任何 metadata 的，甚至不做这个目录也行。像下面这个例子，就只是用 `<ul>` 简单弄了一个。

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title>目录</title>
</head>
<body>
<h1 id="toc">目录</h1>
<ul>
	<li><a href="62257.htm">序章</a></li>
	<li><a href="62258.htm">第一章 咦？照这样看，这次是不是走进个别剧情线了？</a></li>
	<li><a href="69021.htm">后记</a></li>
</ul>
</body>
</html>
```

#### 3. Metadata

Metadata 是存放在 `.opf` 文件里的。里面可以包含的 metadata 非常多。这个文件主要包含以下几块

* `<metadata>`，放有关作品的 metadata，比如作品名、作者名
* `<manifest>`，列出整个电子书涉及的文件，主要是 HTML、`.ncx`，CSS 和图片列不列都没关系
* `<spine>`，描述显示给用户的时候，一个个 HTML 按什么顺序出现
* `<guide>`，会列整本书关键的位置，例如正文的开始之类的，用于确定在上面图片中的“开始”标记

下面是一个 `.opf` 文件的例子，

```
<?xml version="1.0" encoding="utf-8"?>
<package unique-identifier="uid" xmlns:opf="http://www.idpf.org/2007/opf" xmlns:asd="http://www.idpf.org/asdfaf">
    <metadata></metadata>
    <manifest></manifest>
    <spine toc="ncx"></spine>
    <guide></guide>
</package>
```

对于 `<metadata>`，像这种做完自己看看，不去发布不去卖的书，放一个封面、作品名、作者名就可以了。这里支持的 metadata 其实挺多的，在 Open Package Format 的 spec 里有详细地列出。下面是一个 `<metadata>` 的例子。

```
<metadata>
  <dc-metadata  xmlns:dc="http://purl.org/metadata/dublin_core" xmlns:oebpackage="http://openebook.org/namespaces/oeb-package/1.0/">
    <dc:Title>路人女主的养成方法：第七卷</dc:Title>
    <dc:Language>zh</dc:Language>
    <dc:creator opf:role="aut">丸户史明</dc:creator>
    <x-metadata>
      <EmbeddedCover>images/cover.jpg</EmbeddedCover>
    </x-metadata>
  </dc-metadata>
</metadata>
```

可以看到列出了书名、作者、封面图片位置。

对于 `<manifest>`，只要列出重要的正文文件即可。

```
<manifest>
  <item id="content" media-type="text/x-oeb1-document" href="toc.html"></item>
  <item id="ncx" media-type="application/x-dtbncx+xml" href="toc.ncx"/>
  <item id="t1" media-type="text/x-oeb1-document" href="62257.htm"></item>
  <item id="t2" media-type="text/x-oeb1-document" href="62258.htm"></item>
  <item id="t3" media-type="text/x-oeb1-document" href="62259.htm"></item>
</manifest>
```

这里对每个文件都加了个 ID。这个 ID 一会儿 `<spine>` 中会用于 ref 这个文件。

下面是 `<spine>`，和 `<guide>`。在 `<spine>` 中只需要把文件按希望出现的顺序，用 `<manifest>` 设置的 ID 列一下就可以了。`<guide>` 里列了目录和正文开头的位置。

```
<spine toc="ncx">
  <itemref idref="content"/>
  <itemref idref="t1"/>
  <itemref idref="t2"/>
  <itemref idref="t3"/>
</spine>
<guide>
  <reference type="toc" title="目录" href="toc.html"/>
  <reference type="text" title="正文" href="62257.htm"/>
</guide>
```

#### 4. 生成

这一步非常简单。执行

```
kindlegen your.opf
```

就可以了。过程中如果出现 warning，最好一一解决，以免显示效果出错。生成出来的 `.mobi` 文件可以直接拷贝进 Kindle 里看。
