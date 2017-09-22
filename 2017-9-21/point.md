## 前言
好久没写canvas了，准备用canvas做个logo，但是突然发现对DOM节点的宽度、高度的计算有点忘记了，重新复习下吧。

---
## 盒子模型
在讨论宽高之前，先说下CSS的盒子模型：
每一个盒子都可以分为：content、padding、border、margin四个部分。
![box](http://img.blog.csdn.net/20170921195843863?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3dsOTc5NjIzMDc0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
盒子模型分为两种：标准盒子模型和IE盒子模型
**区别：**
 1.标准盒子模型中content中包含content
 2.IE盒子模型中content中包含content、padding、border

---
## 类型
前端宽高计算总共分为对屏幕、窗口、文档（DOM节点）的计算

---
## 屏幕
这里的屏幕是指的电脑显示器，每个屏幕的宽高都是固定值

```
let getScreenAttr = () => {
	return {
		width:screen.width,		
		height:screen.height,	//电脑屏幕的高度，包含任务栏的高度
		availWidth:screen.availWidth,
		availHeight:screen.availHeight,	//电脑屏幕的高度，不包含任务栏的高度
		availTop:screen.availTop,		//未被系统部件占用的最上方的像素值（只读）（不懂）
		availLeft:screen.availLeft,		//	未被系统部件占用的最左侧的像素值（只读）（不懂）
	}
}
```
其中需要注意的height与availHeight的区别：height的高度包括屏幕下端的任务栏的高度，availHeight不包含任务栏的高度。

---
## 窗口
这里的窗口指的是浏览器页面，窗口的大小就是窗口实际缩放的大小，单位为像素
```
let getWindowAttr = () => {
	return {
		innerWidth:window.innerWidth,		//窗口可视区域（能够看的见的屏幕，比如缩小之后，宽度高度都变化）的宽度
		innerHeight:window.innerHeight,		//窗口可视区域的高度
		outerWidth:window.outerWidth,		//包括工具条与滚动条
		outerHeight:window.outerHeight,		//包括工具条与滚动条
		screenTop:window.screenTop,			//窗口左上角距离电脑屏幕顶端的距离
		screenLeft:window.screenLeft,		//窗口左上角距离电脑屏幕左端的距离
	}
}
```
计算窗口我们常用的是innerHeight（innerWidth）和outerHeight（outerWidth），区别是他们是否包含工具条和滚动条。

---
## 文档（DOM节点）
这里的文档（body）是指整个的HTML页面，当然也可以用去计算DOM节点的宽高。
```
let getDomAttr = ele => {
	if(!ele){
		ele = document.body;
	}
	let bound = ele.getBoundingClientRect();
	let str = `width: ${bound.width},height: ${bound.height},top: ${bound.top},bottom: ${bound.bottom},left: ${bound.left},right: ${bound.right},`;
	return {
		clientWidth:ele.clientWidth,		//body可视区域（即body在页面上能看的见部分）的宽度(包括box-content,padding)
		clientHeight:ele.clientHeight,	//body可视区域（即body在页面上能看的见部分）的高度
		offsetWidth:ele.offsetWidth,		//同于clientWidth，但是包含边框的宽度(包括padding,box-content,border)
		offsetHeight:ele.offsetHeight,	//同于clientHeight，但是包含边框的宽度
		clientTop:ele.clientTop,		//boder的宽度
		clientLeft:ele.clientLeft,	//boder的宽度
		getBoundingClientRect:str,	//盒子的宽度、高度(不包含margin)、top、left、right、botom(注意：left、right计算时，以元素可视区（包含padding，包含border）的左上角为原点计算，top、bottom以左下脚为原点，其中width:left+right;height:bottom-top;)
		getClientRect:ele.getClientRects(),
	}
}
```
### clientWidth、clientHeight
需要注意的是这两个API在计算时，只计算content、padding的值。
### offsetWidth、offsetHeight
这两个API类似与**clientWidth、clientHeight**，区别在于它们计算时会将border的值加入进去
### clientTop、clientLeft
这个两个API是用与计算盒子的border的宽度的，虽然从名称上看它们并不像与border有任何关系
### getBoundingClientRect
不得不说的getBoundingClientRect，它会返回一个ClientRect对象，这个对象包含六个属性<br>
 * width	返回的是盒子的宽度，包括content、padding、border。需要注意的它不包含margin
 * heigh	返回的是盒子的高度，包括content、padding、border。需要注意的它不包含margin
 * top	返回的是盒子左上角距离浏览器工具条的垂直高度，**注意：**这里的左上角是border的左上角
 * bottom	返回的是盒子左下角距离浏览器工具条的垂直高度，**注意：**这里的左下角是border的左下角
 * left	返回的是盒子左上角距离浏览器最左侧的水平长度，**注意：**这里的左上角是border的左上角
 * right	返回的是盒子左上角距离浏览器最右侧的水平长度，**注意：**这里的左上角是border的左上角
### getClientRects
与getBoundingClientRect类似，但是它返回的是一个ClientRectList对象

---
# 应用
## 滚动
首先得先屡清楚一个概念，滚动与文档之间的计算关系，我们以body进行说明：
 > 可滚动的高度 = 文档的高度 - 窗口的高度（仅包含content的高度）
 <br>
当我们将文档替换成DOM元素时
 > 可滚动的高度 = DOM内文档的高度 - DOM的高度（仅包含content的高度）

### 滚动高度的获取
```
let getScrollAttr = () => {
	return {
		scrollTop:document.body.scrollTop,			//竖立方向的滚动条距离文档顶端的距离
		scrollLeft:document.body.scrollLeft,		//横立方向的滚动条距离文档左端的距离
		scrollWidth:document.body.scrollWidth,		//文档的宽度,包含box-content,padding
		scrollHeight:document.body.scrollHeight,	//文档的高度,包含box-content,padding
	}
}
```

**需的留意的是：** document.body.clientHeight与document.body.scrollHeight都是可以计算文档的高度，他们都是只包含content和padding，但是结果还是有细微的差别，原由未查到相关资料，理解的同道请给下解释

---
## demo
测试编写的一个demo
![这里写图片描述](http://img.blog.csdn.net/20170921200150389?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3dsOTc5NjIzMDc0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 源码：
```
<!DOCTYPE html>
<html>
<head>
	<title>document window screen DOM</title>
	<style type="text/css">
		html,body{
			padding: 0px;
			margin: 0px;
			height: 2000px;
		}
		
		#box{
			margin: 100px 400px;
			padding: 100px 0;
			border: 50px dashed #CCC;
			text-align:center;
			box-sizing: content-box;
		}
		
		table,td,th{
			border: 1px solid #CCC;
			border-collapse:collapse;
			text-align: center;
			padding: 5px;
		}
		
		.can{
			display: inline-block;
			margin: 0 20px;
		}
	</style>
</head>
<body>
	
	<div style="border: 1px solid blue;margin:10px;">
		<div id="box">
			<p style="width: 100px;height: 100px;background: blue;color: white;display: inline-block;line-height: 100px; ">box-content</p>
		</div>
	</div>
	<div id="rs"></div>
	<script type="text/javascript">
		//滚动条可滚动高度为，文档的高度（document.body.scrollHeight） - 屏幕可视区的高度（window.innerHeight）
		//所以，欲将滚动条置于低端的语句是
		//document.body.scrollTop = document.body.scrollHeight - window.innerHeight
		//window.scrollTo(0,document.body.scrollHeight - window.innerHeight)

		let getWindowAttr = () => {
			return {
				innerWidth:window.innerWidth,		//窗口可视区域（能够看的见的屏幕，比如缩小之后，宽度高度都变化）的宽度
				innerHeight:window.innerHeight,		//窗口可视区域的高度
				outerWidth:window.outerWidth,		//包括工具条与滚动条
				outerHeight:window.outerHeight,		//包括工具条与滚动条
				screenTop:window.screenTop,			//窗口左上角距离电脑屏幕顶端的距离
				screenLeft:window.screenLeft,		//窗口左上角距离电脑屏幕左端的距离
			}
		}

		let getDomAttr = ele => {
			if(!ele){
				ele = document.body;
			}
			let bound = ele.getBoundingClientRect();
			let str = `width: ${bound.width},height: ${bound.height},top: ${bound.top},bottom: ${bound.bottom},left: ${bound.left},right: ${bound.right},`;
			return {
				clientWidth:ele.clientWidth,		//body可视区域（即body在页面上能看的见部分）的宽度(包括box-content,padding)
				clientHeight:ele.clientHeight,	//body可视区域（即body在页面上能看的见部分）的高度
				offsetWidth:ele.offsetWidth,		//同于clientWidth，但是包含边框的宽度(包括padding,box-content,border)
				offsetHeight:ele.offsetHeight,	//同于clientHeight，但是包含边框的宽度
				clientTop:ele.clientTop,		//boder的宽度
				clientLeft:ele.clientLeft,	//boder的宽度
				getBoundingClientRect:str,	//盒子的宽度、高度(不包含margin)、top、left、right、botom(注意：left、right计算时，以元素可视区（包含padding，包含border）的左上角为原点计算，top、bottom以左下脚为原点，其中width:left+right;height:bottom-top;)
				getClientRect:ele.getClientRects(),
			}
		}

		let getScreenAttr = () => {
			return {
				width:screen.width,		
				height:screen.height,	//电脑屏幕的高度，包含任务栏的高度
				availWidth:screen.availWidth,
				availHeight:screen.availHeight,	//电脑屏幕的高度，不包含任务栏的高度
				availTop:screen.availTop,		//未被系统部件占用的最上方的像素值（只读）（不懂）
				availLeft:screen.availLeft,		//	未被系统部件占用的最左侧的像素值（只读）（不懂）
			}
		}

		//滚动条滚动的高度 = 文档的高度 - 可视区（box）的高度
		let getScrollAttr = () => {
			return {
				scrollTop:document.body.scrollTop,			//竖立方向的滚动条距离文档顶端的距离
				scrollLeft:document.body.scrollLeft,		//横立方向的滚动条距离文档左端的距离
				scrollWidth:document.body.scrollWidth,		//文档的宽度,包含box-content,padding
				scrollHeight:document.body.scrollHeight,	//文档的高度,包含box-content,padding
			}
		}
		
		let createTable = (type,data) => {
			let header = ['Attribute','Value'];
			let fragment = document.createDocumentFragment();
			
			let box = document.createElement('div');
			box.className = 'can';
			let div = document.createElement('h3');
			let title = document.createTextNode(type);
			div.appendChild(title);
			box.appendChild(div)

			let table = document.createElement('table');
			let tr = document.createElement('tr');
			for(let val of header){
				
				let th = document.createElement('th');
				let nodeText = document.createTextNode(val);
				th.appendChild(nodeText);
				tr.appendChild(th);
				
			}
			table.appendChild(tr);

			for(let attr in data){
				let tr = document.createElement('tr');
				let td1 = document.createElement('td');
				let text1 = document.createTextNode(attr);
				td1.appendChild(text1);
				tr.appendChild(td1);

				let td2 = document.createElement('td');
				td2.style.width = '150px';
				let text2 = document.createTextNode(data[attr]);
				td2.appendChild(text2);
				tr.appendChild(td2);

				table.appendChild(tr);
			}
			box.appendChild(table);
			fragment.appendChild(box);
			document.getElementById('rs').appendChild(fragment);
		}
		
		let init = () => {
			createTable('window',getWindowAttr());
			createTable('document',getDomAttr());
			createTable('DOM-#box',getDomAttr(document.getElementById('box')));
			createTable('screen',getScreenAttr());
			createTable('scroll',getScrollAttr());
		}

		(()=>{
			let rs = document.getElementById('rs');
			let box = document.getElementById('box');
			rs.style.position = 'fixed';
			rs.style.left = '0px';
			rs.style.bottom = '20px';

			window.onload = init;
			window.onscroll = function(e){
				let docHeight = document.body.scrollHeight;
				let winHeight = window.innerHeight;
				let scrollTop = document.body.scrollTop;
				if(scrollTop >= docHeight - winHeight){
					document.body.scrollTop -= 1;
					return;
				}
				let eles = document.querySelectorAll('.can')
				for(let i = 0; i<eles.length;i++){
					let ele = eles[i];
					ele.parentNode.removeChild(ele);
				}
				init();

			};				
		})();
	</script>
</body>
</html>
```
