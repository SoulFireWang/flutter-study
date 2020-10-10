简介
> 在邮件中块引用中很方便用来仿真文本的回复。
> 这行是同一个块的一部分。

> 当这行很长的文字被包裹的时候，它依然会被恰当的引用。让我们继续写下去来确保包裹它时对于每个人来说它足够长。你可以*在*块引用中使用其它
**Markdown**。

# H1
## H2
### H3
#### H4
##### H5
###### H6

Alt-H1
======

Alt-H2
------

~~划掉这个~~

__ 下划线 __
**星号**
_下划线_

1. 有序列表的第一项
2. 另外一个项

..* 无序子列表

...你可以适当的缩紧列表项中的段落。注意上面的空行和前导空格（至少一个，但是这里我们使用三个来对齐原始的Markdown内容）。

...换行而不形成段落，你需要使用两个尾部空格。..
...注意这行是分开的，但还在同一个段落中。..
...（这个违背了不需要尾部空格的典型的GFM换行行为）。

* 无序列表可以使用星号
- 或者减号
+ 或者加号

[内嵌式链接](https://www.google.com)

[带标题的内嵌式链接](https://www.google.com "谷歌的主页")

[引用式链接][arbitrary case-insensitive reference text]

我们以这行作为开始。

This line is separated from the one above by two newlines, so it will be a *separate paragraph*.

这行与上面那行被两个换行符分隔，所以它会成为一个 *单独的段落*。

This line is also a separate paragraph, but...
This line is only separated by a single newline, so it's a separate line in the *same paragraph*.

这行同样是一个单独的段落，但是...
这行仅仅被一个换行符分隔，所以它是一个在 *同一段落* 中的单独的行。


[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com

Text prior to footnote reference.[^2]
[^2]: Comment to include in footnote.

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

![alt text][logo]
[logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 2"

```javascript
var s = "JavaScript语法高亮";
alert(s);
```

```python
s = "Python语法高亮"
print s
```

```dart
class Widget{

}
print(123)
```

```
没有指明语言，所有没有语法高亮。
让我们随便写个标签试试 <b>tag</b>
```

Alt-H1
======

<a href="https://www.bilibili.com/video/BV1p441177uV?p=4" target="_blank">
<img src="http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg"
alt="IMAGE ALT TEXT HERE" width="750" height="500" border="10" /></a>

This line is also a separate paragraph, but...
This line is only separated by a single newline, so it's a separate line in the *same paragraph*.


<dl>
  <dt>Definition list</dt>
  <dd>Is something people use sometimes.</dd>

  <dt>Markdown in HTML</dt>
  <dd>Does *not* work **very** well. Use HTML <em>tags</em>.</dd>
</dl>

---

***

___

This line is separated from the one above by two newlines, so it will be a *separate paragraph*.