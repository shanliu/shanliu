# cssflex

```
display: -moz-box;
display: -ms-flexbox;
display: -webkit-box;
display: -webkit-flex;
display: box; 
display: flexbox; 
display: flex;
```

> 以react-native使用

容器样式[容器本身也可是上级元素]

	flexDirection 内部元素排放方式
	row 横向排[注意:子元素宽度无效]
    column 竖向排[注意:子元素高度无效]
	row-reverse[横反] column-reverse[竖反]
	flexWrap 内部元素排放时是否换行[一般为元素过多] nowrap 换行 wrap 不换行 wrap-reverse 底部向上换行
	justifyContent 内部元素排放主轴[默认横向]间隔和靠边方式[一般为元素不足] flex-start 开始放,不足尾部留空 flex-end 结尾放,不足开头留空 center 放中间 space-between 两边放,中间间隙大 space-around 两边不放,边间隙跟元素间隙一样大
	alignItems 内部元素排放竖向靠边方式[主轴为横向,单排] flex-start 顶部对其 flex-end 底部对其 center 竖向居中 baseline 子元素内文字为标准对其 stretch 子元素高度100%
	alignContent 内部元素排放竖向靠边方式[主轴为横向,多排] flex-start 顶部对其 flex-end 底部对其 center 多个竖向居中 space-between 头低放 中间间隙大 space-around 头低不放,间隙同大 stretch 多行元素合成100%

元素样式

	order 元素本身的顺序位置,从小到大排 
	flexGrow 放大比例 
	flexShrink 缩小比例 
	flexBasis 空间有空余时主轴设置为此值 
	
	flex :flex-grow [flex-shrink] [flex-basis] alignSelf 元素本身的位置方式,参考:alignItems