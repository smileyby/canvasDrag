实现canvas里图形的拖拽
====================

在前端交互中，拖拽是比较常见的。比如页面里的弹出登录框，或者一些个人主页的自定义布局，这种布局允许用户自由的拖拽不同模块到不同位置，当然这种自定义布局涉及到后台数据记录，再或者我们曾经玩过的拼图游戏，这些都是拖拽应用。

倘若说道拖拽的表现形式，有通过js结合mouse事件（mousedown、mousemove、mouseup）来改变DOM元素的left、top值得，也有[html5里拖放API](http://www.yi-jy.com/2013/08/24/html5%E6%8B%96%E6%94%BEapi%E7%AE%80%E4%BB%8B%E5%8F%8A%E5%BA%94%E7%94%A8/)，而今天讲的则是canvas里的图形拖拽。

首先来看实例演示：[canvas里图形的拖拽](https://smileyby.github.io/canvasDrag/)

那么类似这种交互式如何实现的？我们知道，canvas没有DOM的概念，因此就更没有什么给元素绑定事件的做法。我们换个思路，要实现canvas里图形的拖拽，就相当于图形的绘制坐标不断改变，并且这个会制作表是随着鼠标移动而改变的。但如何将鼠标坐标与图形绘制坐标建立关系呢？其实原理和DOM里是一样的。

在DOM中拖拽元素原理大概是这样的：mousedown的时候记录鼠标与元素的距离，这里用offset对象来表示：

```js

var offset = {
	x: mouse.x - shape.x,
	y: mouse.y - shape.y
};

```

接下来是mousemove。试想一下，元素随着鼠标移动，鼠标的坐标不断变化，但拖拽过程中鼠标与元素的距离始终保持不变，因此我们可以通过这个距离得出元素的坐标（left/top值）：

```js

mouse = {
    x : e.clientX ,
    y : e.clientY
};
...
element.left = mouse.x - offset.x;
element.top = mouse.y - offset.y;

```

对于canvas的各种图形，虽然没有left、top值，但是有绘制坐标（类似元素的left、top值）。虽然不能绑定事件，但我们可以将事件作用于canvas元素上。鼠标在canvas上移动，我们就清楚整个canvas画布，在绘制图形（shape）即可。一些都看起来很顺利：

```js

canvas.addEventListener( "mousedown" , function(e){
    var mouse = {
            x : e.clientX - canvas.getBoundingClientRect().left,
            y : e.clientY - canvas.getBoundingClientRect().top
        },
        offset = {
            x : mouse.x - shape.x,
            y : mouse.y - shape.y
        };
 
    canvas.addEventListener("mousemove" , function(e){
        mouse = {
            x : e.clientX - canvas.getBoundingClientRect().left,
            y : e.clientY - canvas.getBoundingClientRect().top
        };
 
        shape.x = mouse.x - offset.x;
        shape.y = mouse.y - offset.y;
 
        canvas.clearRect(0,0,canvas.width,canvas.height);
        drawGraph(); // 重新绘制图形
 
    } , false);
 
    canvas.addEventListener("mouseup" , function(){
        ...
    } , false);
})

```

但我们忽略了一个很重要的问题：如何确定鼠标是在某个图形中呢？

这看起来的确是一个很头疼的问题，因为只有鼠标在入行中，才能更新坐标，重绘图形，产生拖拽效果。幸运的是，在canvas中提供了一个方法能够检测某个点是否在图形中，那就是`isPointPath`，它的参数是坐标的x和y值，于是就有：

```js

function isMouseInGraph(mouse) {
	context.beginPath();
    context.rect( shape.x , shape.y , shape.w , shape.h);
    return  context.isPointInPath(mouse.x , mouse.y);
}

```

当对canvas执行mousedown时，通过isMouseInGraph方法来判断当前鼠标点是否在图形中，然后根据鼠标点与图形的初始距离（上面提到的offset），在鼠标mousemove移动时，更新图形的绘制坐标，最终实现canvas图形拖拽。

对于一些特殊需求，比如canvas中有多个图形，当拖拽到多个图形重叠在一起时，可能会出现多个图形一起移动。但实际上我们想要的效果是，只移动最上面的图形。比如比如A先绘制，图形B后绘制。此时B覆盖在A的上面，在canvas按下鼠标，当鼠标点在A和B的重叠处时，图形A、B都满足`isPointInPath`方法。但我们需要移动的只是图形B，即后面绘制符合条件的图形。此处，我么就需要用一个临时数组来存储这些符合条件的图形对象，取临时数组的最后一项更新它的坐标即可。


