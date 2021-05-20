---
title: summernote使用
top: false
cover: false
toc: true
mathjax: true
date: 2020-05-17 15:29:53
password:
summary: bootstrap富文本编辑器summernote使用
tags:
- 前端
categories:
- 前端
---

# Bootstrap —— summernote

官方文档地址：https://summernote.org/getting-started

## 使用步骤

1. 这里使用cdn引入
```html
	<!-- 插件引入 -->
	<link href="http://cdnjs.cloudflare.com/ajax/libs/summernote/0.8.9/summernote.css" rel="stylesheet">
	<script src="http://netdna.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.js"></script>
	<script src="http://cdnjs.cloudflare.com/ajax/libs/summernote/0.8.9/summernote.js"></script>
	
	<!--引入中文JS包-->
	<script src="https://cdn.bootcss.com/summernote/0.8.10/lang/summernote-zh-CN.js"></script>  //引入中文包，不然编辑器是默认的英文
```

当然也可以用文件引入

	1、将summernote 相应的文件放到工程中（webapp下面）
	2、建一个jsp文件，在文件中引入相应的js、css文件

2. 建一个div区域存放内容，注意写上id

```html
	<div id="summernote"></div>   //内容载体
```

3. js初始化

```js
    // 编辑器功能=====================================
	$("#summernote").summernote({
		lang : 'zh-CN',// 语言
        height : 496, // 高度
        minHeight : 300, // 最小高度
        placeholder : '请输入文章内容', // 提示       
        // summernote自定义配置(编辑器上方的粗体什么的)
        toolbar: [
        ['operate', ['undo','redo']],
        ['magic',['style']],
        ['style', ['bold', 'italic', 'underline', 'clear']],
        ['para', ['height','fontsize','ul', 'ol', 'paragraph']],
        ['font', ['strikethrough', 'superscript', 'subscript']],
        ['color', ['color']],
        ['insert',['picture','video','link','table','hr']],
        ['layout',['fullscreen','codeview']],
        ],
		callbacks : { // 回调函数
		// 上传图片时使用的回调函数   因为我们input选择的本地图片是二进制图片，需要把二进制图片上传服务器，服务器再返回图片url，就需要用到callback这个回调函数
            onImageUpload : function(files) { 
                var form=new FormData();
                form.append('myFileName',files[0])  //myFileName 是上传的参数名，一定不能写错
                $.ajax({
                    type:"post",
                    url:"${path}/Img/upload", //上传服务器地址
                    dataType:'json',
                    data:form,
                    processData : false,
                    contentType : false,
                    cache : false,
                    success:function(data){
                        console.log(data.data)                            
                        $('#summernote').summernote('editor.insertImage',data.data);
                    }
                })
            }
		}
	});
```

**注意：** $('#summernote').summernote('insertImage', url, filename); 官方文档提供的图片插入封装函数，在本地环境下服务器返回的url并不能展示在编辑框内，放到生产环境的时候自然会展示在编辑框内（网上搜索才知道，网址具体忘了，侵删）

## 最后样式

![](1.png)

## 其他使用

1. 获取编辑器内的HTML内容

	var markupStr = $('#summernote').summernote('code');

如果初始化了多个编辑器，可以通过jquery的eq方法获取某个编辑器的HTML内容。eg,获取第二个编辑器的。

	var markupStr = $('.summernote').eq(1).summernote('code');

2. 赋值

	// 这样也是赋成html格式
	$('.summernote').summernote('code','要赋的值');
