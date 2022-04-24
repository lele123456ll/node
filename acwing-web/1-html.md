# html

## 1. html文件结构 - 树形

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web应用课</title>
</head>
<body>
    <h1>第一讲</h1>
</body>
</html>
```

- <html>为根节点
- 引入资源 <link rel="类型" href="地址">

## [2. 文本标签](https://www.acwing.com/file_system/file/content/whole/index/content/4084683/)

## 3. 图片

```html
<img src="必须填写" alt="文本描述" width="" height=""/>
```

## 4. 音频与视频

```html
<audio src = "" type = "audio/mpeg"/>

如果有多个<source>, 就会一个个尝试加载..
<audio controls>
    <source src="/audios/sound1" type="audio/mpeg"/>
    <source src="/audios/sound2" type="audio/mpeg"/>
</audio>

<video src="" type = "video/mp4">
如果有多个<source>, 就会一个个尝试加载..
    
<video controls width="800">
    <source src="/videos/video1.mp4" type="video/mp4">
    <source src="/videos/video2.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>
```

## 5. 超链接

```html
<a href = "" target="_blank"> 跳转重新打开一个页面
```

## 6. 表单

```html
<form>

<input>填写内容: 
    <input type="text">
    <input type="number">
    <input type="email">
    <input type="password">
    <input type="radio">
    
例如: 
    <label for="username">用户名</label>
    <input type="text" name="username" id="username" required minlength="3" maxlength="15" placeholder="用户名"> 
    
    placeholder : 空白显示
    label : 将一个 <label> 和一个 <input> 元素匹配在一起，你需要给 <input> 一个 id 属性。而 <label> 需要一个 for 属性，其值和  <input> 的 id 一样。
    
```

- 使用label优点
    - 标签文本不仅与其相应的文本输入元素在视觉上相关联，程序中也是如此。 这意味着，当用户聚焦到这个表单输入元素时，屏幕阅读器可以读出标签，让使用辅助技术的用户更**容易理解**应输入什么数据。
    - 你可以点击**关联的标签来聚焦或者激活这个输入元素**，就像直接点击输入元素一样。这扩大了元素的可点击区域，让包括使用触屏设备在内的用户**更容易激活这个元素**。

## [7. 列表](https://www.acwing.com/blog/content/15784/)

## 8. 表格

```html
    <table>
    <caption> 标题
 	<thead> 定义了一组定义表格的列头的行。
    <tbody> 定义一组数据行。
    <tr> 定义了一个包含数据的表格单元格。
    <th> 元素定义表格内的表头单元格。
```

## [9. 语义标签](https://www.acwing.com/blog/content/15799/)

## [10 .特殊符号](https://www.acwing.com/blog/content/15812/)

