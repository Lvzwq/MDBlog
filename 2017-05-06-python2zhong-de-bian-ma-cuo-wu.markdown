---
layout: post
title: "Python2中的编码错误"
date: 2015-10-31 17:22:55 +0800
comments: true
categories: Dev
tags: [Python]
---

<!--more-->

<h3>Python2中常出现的编码问题</h3>

<pre><code>UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 4: ordinal not in range(128)
</code></pre>

对中文字符串解码出错
虽然可以在代码头部加入以下代码解决

<pre><code>import sys
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
</code></pre>

但是这种方式已经不推荐使用了<a href="http://stackoverflow.com/questions/3828723/why-we-need-sys-setdefaultencodingutf-8-in-a-py-script">[详细阅读]</a>。  
所以推荐使用以下方式(Python2默认的字符串编码是<code>str</code>)  
<code>xxx.decode("utf-8")</code>  
或者在你需要使用中文的地方前面加上<code>u</code>.

<blockquote>
  这种错误通常发生在以某种编码("<code>ascii</code>")解码一个<code>str</code>类型的字符串时.因为编码映射仅仅只能支持一部分<code>str</code>类型的字符串到<code>unicode</code>字符串, 一个非法的<code>str</code>类型的序列将会导致编码<code>decode()</code>失败。 <a href="https://wiki.python.org/moin/UnicodeDecodeError">[详细阅读]</a>
</blockquote>

如<code>str</code>字符串<code>\x81</code>就不能转化为<code>unicode</code>对应的字符串

<pre><code>&gt;&gt;&gt; "x81".decode("utf-8")
Traceback (most recent call last):
  File "&lt;stdin&gt;", line 1, in &lt;module&gt;
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/encodings/utf_8.py", line 16, in decode
    return codecs.utf_8_decode(input, errors, True)
UnicodeDecodeError: 'utf8' codec can't decode byte 0x81 in position 0: invalid start byte
</code></pre>

另一种情况的解码错误

<blockquote>
  自相矛盾的是，<code>UnicodeDecodeError</code>也可能发生在编码<code>__encoding__</code>时。
  原因是编码函数<code>encode()</code>通常情况下需要一个<code>unicode</code>类型的字符串作为参数。但是实际传过来的是一个<code>str</code>类型的参数。<code>encode()</code>函数将这个参数向上转换"<code>up-convert</code>"为<code>unicode</code>类型，然后再将转化为他们自己的编码。这也会出现这样的向上转换"<code>up-convertion</code>"的时候，系统默认选择一个<code>ascii</code>解码器, 解码器中没有这个<code>str</code>类型的<code>unicode</code>编码,  。因此这是在一个编码器<code>encoder</code>中出现解码失败的情况。
</blockquote>

如<code>str</code>类型的<code>\xd0\x91</code>在encode()时就会出现<code>UnicodeDecodeError</code>错误

<pre><code>&gt;&gt;&gt; "xd0x91".encode("utf-8")
Traceback (most recent call last):
  File "&lt;stdin&gt;", line 1, in &lt;module&gt;
UnicodeDecodeError: 'ascii' codec can't decode byte 0xd0 in position 0: ordinal not in range(128)
</code></pre>

在这个过程中发生了两件事情。首先<code>\xd0\x91</code>是python默认的<code>str</code>类型的字符串，而编码<code>encode</code>需要一个<code>unicode</code>类型的字符串，所以在编码<code>encode</code>之前，先转化为<code>unicode</code>,而执行的是<code>"\xd0\x91".decode("ascii")</code>, 所以会出现上面的错误。

<pre><code>&gt;&gt;&gt; "xd0x91".decode("ascii")
Traceback (most recent call last):
  File "&lt;stdin&gt;", line 1, in &lt;module&gt;
UnicodeDecodeError: 'ascii' codec can't decode byte 0xd0 in position 0: ordinal not in range(128)
&gt;&gt;&gt; # 而如果使用utf-8 转码的话就可以正常执行
&gt;&gt;&gt; "xd0x91".decode("utf-8")
u'u0411'
</code></pre>

<h3>总结</h3>

所以出现上面的<code>UnicodeDecodeError</code>错误时，可以不使用系统的<code>ascii</code>编码解码，自己使用<code>utf-8</code>解码就可以解决问题,即<code>xx.decode("utf-8")</code>。但是如果<code>str</code>类型的字符串非常特殊，如第一种例子中的<code>\x81</code>的话就直接无法转码了。所以最好的方法还是将一个字符串定义为<code>unicode</code>类型。