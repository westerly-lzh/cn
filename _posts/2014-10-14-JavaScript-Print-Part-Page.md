---
layout: post
title:  JavaScript打印局部页面
categories:
- js
tags:
- 打印
- print
- 局部页面
- js
- javascript



---

虽然浏览器提供了打印页面功能，但有时候需要打印页面上的局部内容。有些浏览器也提供了打印页面选中部分的功能，但仍会在打印出来的页面上把未选中部分所占的空白留出。所以如果有一种方式可以只打印选中的部分，而且以占满纸张的方式打印，那将会非常符合需求。下面的代码就完成了这样的功能。

	<script type="text/javascript">
		function Printpart(idstr){
   			var el = document.getElementById(idstr);
   			var iframe = document.createElement('IFRAME');
   			var doc = null;
   			iframe.setAttribute('style', 'position:absolute;width:0px;height:0px;left:-500px;top:-500px;');
   			document.body.appendChild(iframe);
   			doc = iframe.contentWindow.document;
   			doc.write('<div>' + el.innerHTML + '</div>');
   			doc.close();
   			iframe.contentWindow.focus();
   			iframe.contentWindow.print();
   			if (navigator.userAgent.indexOf("MSIE") > 0){
        		document.body.removeChild(iframe);
   			}
		}
		Printpart("div_needed_to_print")
	</script>
	
只需要把页面保存到本地，然后使用文本编辑器打开保存的页面，把这部分代码粘贴到页面的最后面，并把 **div_needed_to_print**  替换为需要打印部分的div的id即可。



