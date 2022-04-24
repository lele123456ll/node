# wxml

## 模板

```html
定义
    <template name="print"> 
      <view>
        <text> {{index}}: {{msg}} </text>
        <text> Time: {{time}} </text>
      </view>
    <template>
js定义数据:
item: {
   index: 0,
   msg: 'this is a template',
   time: '2016-06-18'
}
使用模板
    <template is = "print" data = "{{...item}}">
```

## 引用

```html
导入模板
<!-- item.wxml中 -->
<template name="item">
  <text>{{text}}</text>
</template>
        
<!--.wxml文件-->
<import src="item.wxml"/>
<template is="item" data="{{text: 'forbar'}}"/>
        
include 可以将目标文件中除了 <template/> <wxs/> 外的整个代码引入，相当于是拷贝到 include 位置

<!-- index.wxml -->
<include src="header.wxml"/>
<view> body </view>
<include src="footer.wxml"/>

<!-- header.wxml -->
<view> header </view>

<!-- footer.wxml -->
<view> footer </view>
```

