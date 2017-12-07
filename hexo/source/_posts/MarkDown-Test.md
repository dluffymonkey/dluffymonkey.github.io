---
title: MarkDown
date: 2016-11-21 16:21
tags: Hexo
---
><font color="#4590a3" size = "4px">文字</font>



# This is an H1

## This is an H2 

### This is an H3 

>哈哈我就是这个热线


	- (UIView *)tableView:(UITableView *)tableView viewForFooterInSection:(NSInteger)section
	{
		UIView *footV = [[PurchaseSectionFooterView alloc] init];
		return footV;
	}	


> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.
> > > My name is Leo

这是为啥呢

<!--more -->

> ## 这是一个标题。
>
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
>
> 给出一些例子代码：
>
>     return shell_exec("echo $input | $markdown_script");


*   Red
*   Green
*   Blue

+   Red
+   Green
+   Blue

-   Red
-   Green
-   Blue

1.  Bird
2.  McHale
3.  Parish


<ol>
<li>Bird</li>
<li>McHale</li>
<li>Parish</li>
</ol>

1.  Bird
2.  McHale
3.  Parish




3. Bird
4. McHale
5. Parish

<ul>
<li>Bird</li>
<li>Magic</li>
</ul>

1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.


*   This is a list item with two paragraphs.

    This is the second paragraph in the list item. You're
    only required to indent the first line. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit.

*   Another item in the same list.


*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.


*   一列表项包含一个列表区块：

        <代码写在这>
   

1986\. What a great season.

* * *

***

*****

- - -

---------------------------------------

This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

See my [About](/about/) page for details.

This is [an example][id] reference-style link.

This is [an example] [id] reference-style link.

[link text][a]
[link text][A]

[id]: http://example.com/  "Optional Title Here"

[A]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  "Optional Title Here"

[Google][]
[Google]: http://google.com/

Visit [Daring Fireball][] for more information.
[Daring Fireball]: http://daringfireball.net/

I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].

[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"


<p>I get 10 times more traffic from <a href="http://google.com/"
title="Google">Google</a> than from
<a href="http://search.yahoo.com/" title="Yahoo Search">Yahoo</a>
or <a href="http://search.msn.com/" title="MSN Search">MSN</a>.</p>

I get 10 times more traffic from [Google](http://google.com/ "Google")
than from [Yahoo](http://search.yahoo.com/ "Yahoo Search") or
[MSN](http://search.msn.com/ "MSN Search").


*single asterisks*

_ single underscores_

**double asterisks**

__double underscores__

un*frigging*believable

\*this text is surrounded by literal asterisks\*

Use the `printf()` function.


``There is a literal backtick (`) here.``

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

Please don't use any `<blink>` tags.

&#1254; is the decimal-encoded equivalent of `&mdash;`.

![Alt text](http://img5q.duitang.com/uploads/item/201306/26/20130626164439_MRL8t.thumb.700_0.jpeg) 
<center>luffy</center>
![Alt text](http://img5q.duitang.com/uploads/item/201504/10/20150410H5845_3BHyr.jpeg)

<address@example.com>



插入外部链接图片
![“图片描述”](“图片地址”)  

添加本地图片
在\hexo\source目录下新建文件夹，命名为images或者其他你喜欢的名字，然后编辑你的md博文，插入下面的代码样式：
![“图片描述”](/images/你的图片名字.JPG)  

插入音乐
比如网易云音乐，找到喜欢的歌曲，点击分享按钮，把里面的代码复制下来，直接粘贴到博文中即可

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86>
    src="http://music.163.com/outchain/player?type=2&id=25706282&auto=0&height=66">  
</iframe>  


		\   反斜线
		`   反引号
		*   星号
		_   底线
		{}  花括号
		[]  方括号
		()  括弧
		#   井字号
		+   加号
		-   减号
		.   英文句点
		!   惊叹号