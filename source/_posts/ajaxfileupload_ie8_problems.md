title: 'AjaxFileUpload IE8 problems'
tags:
  - web
categories:
  - web
date: 2014-01-03 01:13:37
---

本不是做前端的，无奈公司缺少有经验的前端开发人员，只能偶尔客串一下。

今天被人拉着去解决浏览器兼容问题。是一个文件上传功能，这个功能在chrome上运行良好，但要求兼容IE8，文件上传功能无法正常运行了。功能用到了AjaxFileUpload这个插件。经过调试，主要有两个问题。

1. 因为IE安全限制问题，没有点击file的浏览按钮选择文件都不让上传。一定要手动点击，js触发click事件是无效的。

方案：由于页面样式是自定义的，不能使用默认的file控件。所以采用参考资料中的方案，在自定义的“选择文件”的按钮上放置file控件，并置file控件为透明。这样的话，用户以为自己点击的是“选择文件”按钮，而实际上是点击的file控件上的按钮。

2. 对于json类型的返回值，ie会提示下载文件，而不能由AjaxFileUpload的success handler捕获。

方案：把response的content-type由application/json改为text/html或text/plain。

参考：

[IE input file隐藏不能上传文件解决方法](http://www.qttc.net/201305334.html)
