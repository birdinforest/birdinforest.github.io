---
layout:     post
title:      First Entry
category: blog
description: My first entry on GitHub blog
---

##My blog on GitHub
Generally, this is a test entry.

##Test code hightlight
<pre class="prettyprint">
$('#disqus_container .comment').on('click',function(){
        $(this).html('加载中...');
        var disqus_shortname = 'beiyuu';
        var that = this;
        BYB.includeScript('http://' + disqus_shortname + '.disqus.com/embed.js',function(){$(that).remove()}); //这是一个加载js的函数
});
</pre> 

<pre class="prettyprint">
// sort list from less to greater
// a_sortList: the list waiting to be sorted
// a_max: numbers of element in the list
int& bubbleSort(int a_sortList[], int a_max)
{
	int temp = 0;

	// i is the counter, start from a_max - 1, minus by 1 every loop
	for (int i = 0; i < a_max - 1; ++i)
	{
		// compare every adjacent pair, move the greater one to the right side
		for (int j = 0; j < a_max - 1 - i; ++j)
		{
			if (a_sortList[j] > a_sortList[j + 1])
			{
				temp = a_sortList[j];
				a_sortList[j] = a_sortList[j + 1];
				a_sortList[j + 1] = temp;
			}
		}
	}
	return *a_sortList;
}
</pre> 

##Insert image
My lovely Baybay.
![test](/images/test/avatar.jpg)
![test](/images/test/baybay.jpg)

##Insert video from youtube
<!--
<iframe width="560" height="315" src="//www.youtube.com/embed/x4prTw5kE40" frameborder="0" allowfullscreen></iframe>
-->
<div style="text-align:center">
<embed src="//www.youtube.com/embed/x4prTw5kE40" frameborder="0" allowfullscreen width="560" height="315"></embed>
</div>

##Insert video from youku
<div style="text-align:center">
<embed src="http://player.youku.com/player.php/sid/XNTUxNDY1NDY4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" 
align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>
</div>
