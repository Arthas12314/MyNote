## HTML概述
- HTML： Hyper Text Markup Language超文本标记语言
- 超文本：比普通文本功能更加强大，可以添加各种样式
- 标记语言：通过一组标签，来对内容进行描述<关键字>
## HTML语法规范
```HTML
	<!DOCTYPE html> --文档声明--
		<html> --根标签 html主要分头体两部分，头放置一些页面信息，体放置HTML内容--
			<head>
				<meta charset="utf-8">  --编码方式--
				<title></title>
			</head>
			<body>
			</body>
		</html>
```
## 标签
- 标题```<h1></h1>```取值范围1-6
- 段落```<p></p>```
- 分割线```<hr />```
- font标签
	* 常用属性
		+ color
		+ size
		+ face(字体)
	* 实现：
	```<font color="red" size="1-7" face="">文本</font>```
- 补充
	* ```<b>:加粗 <i>:斜体 <strong>:带语义标签的加粗 <em>:带语义标签的斜体```
- src标签
	* 常用属性
		+ src
		+ width
		+ height
		+ alt 文件加载失败时提示信息
	* 实现
	```<img src="../img/*.jpg" width="500px" alt="这张图片可能加载问题"/>```
		+ tips: ./当前路径 ../上一级路径
- 无序列表ul
	* type disc square 
	* li列表项
	* 实现
	```<ul type=""> 
			<li></li>
			<li></li>
			<li></li>
	   </ul>```
- 有序列表ol
 	* type disc square
 	* li列表项
 	* start 起始数字
	* 实现
	```<ol type=""> 
			<li></li>
			<li></li>
			<li></li>
	   </ol>```
- 超链接标签a
	* 常用属性：
		+ href：指定要跳转去的地址 需要加http协议
		+ target：以什么方式打开 _self:默认窗口打开 _blank：新标签页打开
- 表格标签table
	* 常用属性 
		+ 行tr
		+ 列td
		+ 边框border
		+ 宽度width
		+ 高度height
		+ 对齐方式align
	* 常用操作
		+ colspan跨列
		+ rowspan跨行
	* 实例
	``` <table border="1px" width="400px" bgcolor="yellow">
		<tr bgcolor="red">
			<td colspan="2"></td>
		</tr>
		</table>
	```
## 案例
