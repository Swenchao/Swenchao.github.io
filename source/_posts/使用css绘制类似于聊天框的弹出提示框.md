---
title: 使用css绘制类似于聊天框的弹出提示框
top: false
cover: false
toc: true
mathjax: true
date: 2020-05-17 11:09:22
password:
summary: css绘制图形
tags:
- 前端
categories:
- 前端
---

# 使用css画出类似于聊天框的弹出提示框

由于项目需要，要做一个在点击保存按钮，保存成功后，在右边显示保存成功的弹出提示框，然后一秒后消失。于是查阅网络，经过修改改成了自己需要的。

## 制作效果

![](1.jpg)

## 代码

```html
	<head>
        <style type="text/css">
            #saveSuccess {
                position: relative;
                width: 100px;
                min-height: 30px;
                background: #00da88;
                /*圆角*/
                border-radius: 5px;  
                margin-left: 10px;
                font-weight: bold;
                color: white;
                padding: 10px 20px 10px;
                line-height: 18px;
            }
            #saveSuccess::after {
                content: "";
                display: block;
                position: absolute;
                width: 0;
                height: 0;
                border: 8px solid transparent;
                border-right-color: #00da88;
                top: 10px;
                left: -14px;
            }
        </style>
	</head>
	<body>
		<button type="button" class="btn btn-sm btn-white" onclick="submitHandler()"><i class="fa fa-check"></i>保 存</button>
		<p id="saveSuccess" style="visibility: hidden; display: inline-block;">保存成功</p>
	</body>
	<script>
		function submitHandler() {
			// 其他代码
			
			// 保存成功
			success : function(result) {
				...
document.getElementById("saveSuccess").style.visibility="visible";  // 设置显示出来
				// 设置属性隐藏
				if ($("#saveSuccess").attr("style.visibility", "hidden")) {
					// 1s后隐藏
					setTimeout("$('#saveSuccess').hide();", 1000);
						// 隐藏后显示，由于属性是隐藏，所以就算是显示出来也看不到
						$('#saveSuccess').show();
                    }
                }
            });
        }
	</script>
```

