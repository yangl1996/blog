---
date: "2012-01-17T09:30:32Z"
title: Arduino+LCD1602：显示
---

<blockquote>连接，写代码，显示

&nbsp;</blockquote>
<ul>
	<li>引言：Arduino Cookbook，好书</li>
</ul>
这不是题外话。我之所以会试试看LCD Shield，一是因为我本来就有这块板，但一直不会用，第二个原因则是<em>Arduino Cookbook</em>这本书。O'Reilly的这本手册对你在Arduino实践中的各种问题（几乎是每一种问题）提供了对应的解答。"Problem"描述了出现的问题，"Recipe"给出了解决办法，"Discuss"则进行了更进一步的讲解。参考了这些资料后，我开始试验我的LCD Shield。

<strong><em>Arduino Cookbook</em>可以在“<a title="皮皮书屋" href="http://www.ppurl.com/" target="_blank">皮皮书屋</a>”下载，这是一个非常好的电子书网站，收录的大批关于计算机及其衍生学科的电子书，大部分是英文原版PDF。</strong>
<ul>
	<li>基础知识</li>
</ul>
本篇文章，我会改变一下以前“实验笔记”的形式，介绍一些基础性的知识，方便你自己研究、试验。

LCD Keypad Shield：我使用的LCD模块。接口编号（括号内为对应接口编号）：DB4(PIN4),DB5(PIN5),DB6(PIN6),DB7(PIN7),RS(PIN8),Enable(PIN9),背光控制(PIN10)

LiquidCrystal库：这个库包含了你要让LCD工作的一切函数，Arduino IDE自带。你可以在<a title="这里" href="http://arduino.cc/en/Reference/LiquidCrystal?from=Tutorial.LCDLibrary" target="_blank">这里</a>找到相关信息，当然，下面一会提到一些。

函数（全部由我人工翻译，若出错请见谅并指正）：
<ol>
	<li>LiquidCrystal()——定义你的LCD的接口：各个引脚连接的I/O口编号，格式为LiquidCrystal(rs, enable, d4, d5, d6, d7)
LiquidCrystal(rs, rw, enable, d4, d5, d6, d7)
LiquidCrystal(rs, enable, d0, d1, d2, d3, d4, d5, d6, d7)
LiquidCrystal(rs, rw, enable, d0, d1, d2, d3, d4, d5, d6, d7)</li>
	<li>begin()——定义LCD的长宽（n列×n行），格式<em>lcd</em>.begin(cols, rows)</li>
	<li>clear()——清空LCD，格式<em>lcd</em>.clear()</li>
	<li>home()——把光标移回左上角，即从头开始输出，格式<em>lcd</em>.home()</li>
	<li>setCursor()——移动光标到特定位置，格式<em>lcd</em>.setCursor(col, row)</li>
	<li>write()——在屏幕上显示内容（必须是一个变量，如"Serial.read()"），格式<em>lcd</em>.write(data)</li>
	<li>print()——在屏幕上显示内容（字母、字符串，等等），格式<em>lcd</em>.print(data)
<em>lcd</em>.print(data, BASE)</li>
	<li>cursor()——显示光标（一条下划线），格式<em>lcd</em>.cursor()</li>
	<li>noCursor()——隐藏光标，格式<em>lcd</em>.noCursor()</li>
	<li>blink()——闪烁光标，格式<em>lcd</em>.blink()</li>
	<li>noBlink()——光标停止闪烁，格式<em>lcd</em>.noBlink()</li>
	<li>display()——（在使用noDisplay()函数关闭显示后）打开显示（并恢复原来内容），格式<em>lcd</em>.display()</li>
	<li>noDisplay()——关闭显示，但不会丢失原来显示的内容，格式为<em>lcd</em>.noDisplay()</li>
	<li>scrollDisplayLeft()——把显示的内容向左滚动一格，格式<em>lcd</em>.scrollDisplayLeft()</li>
	<li>scrollDisplayRight()——把显示的内容向右滚动一格，格式为<em>lcd</em>.scrollDisplayRight()</li>
	<li>autoscroll()——打开自动滚动，这使每个新的字符出现后，原有的字符都移动一格：如果字符一开始从左到右（默认），那么就往左移动一格，否则就向右移动，格式<em>lcd</em>.autoscroll()</li>
	<li>noAutoscroll()——关闭自动滚动，格式<em>lcd</em>.noAutoscroll()</li>
	<li>leftToRight()——从左往右显示，也就是说显示的字符会从左往右排列（默认），但屏幕上已经有的字符不受影响，格式<em>lcd</em>.leftToRight()</li>
	<li>rightToLeft()——从右往左显示，格式<em>lcd</em>.rightToLeft()</li>
	<li>createChar()——自造字符，最多5×8像素，编号0-7，字符的每个像素显示与否由数组里的数（0-不显示，1-显示）决定，格式<em>lcd</em>.createChar(num, data)，有点难理解，可以看一个例子</li>
</ol>
<code>#include &lt;LiquidCrystal.h&gt;

LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

byte smiley[8] = {
  B00000,
  B10001,
  B00000,
  B00000,
  B10001,
  B01110,
  B00000,
};

void setup() {
  lcd.createChar(0, smiley);
  lcd.begin(16, 2);
  lcd.write(0);
}

void loop() {}</code>
基本知识就介绍到这里，需要更多信息，可以看看<em>Arduino Cookbook</em>，访问<a href="http://arduino.cc/en/Reference/LiquidCrystal" target="_blank">Arduino网站</a>与我联系（如果我能提供帮助的话）。