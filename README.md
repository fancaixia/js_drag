# js_drag
基于jq 的js拖拽小插件<br/>
![https://github.com/fancaixia/js_drag/blob/master/pic/001.png](https://github.com/fancaixia/js_drag/blob/master/pic/001.png)

案例思路：
 > 监听每个li按下，移动，松开事件<br/>
     鼠标移动后修改当前元素坐标值 （鼠标跟随状态）

 >  鼠标松开时<br/>
     计算选中盒子是否移入展示区域<br/>
    没有移入展示区域则重置为初始位置（因为盒子是相对定位 所以初始位置为0，0）
   
> 展示区域的元素根据进入的先后顺序设置z-index  （层叠顺序）<br/>
    并且根据先后顺序设置left 位置<br/>
    
##### html
```
<!DOCTYPE html>   
<html lang="zh">   
<head>   
    <meta charset="UTF-8">   
    <title>js拖拽</title>  
    <link rel='stylesheet' type='text/css' href="./drag.css">
</head>   
<body>   
  <div id="select_box">

      <div id="content_box">
          快来挑选喜欢的水果吧，只能选一个哦
      </div>

  </div>

<script src="./jquery-3.3.1.js"></script>
<script src='./drag.js'></script>

<script>  
window.onload = function (){
  
  let dragObj = new dragFruit('#select_box','#content_box');
  dragObj.config = {
            fruit_height : 100,     // 水果盒子的高度
            container_width : 500,   // 最外层容器宽度设置 container
            content_padding : 10,  // 容器内边距   
            content_height : 120,  // 展示区域高度
        }
}

</script>   
</body>   
</html>  
```
##### drag.js
```

function dragFruit(container,content_box){

    // container  最外层容器组件
    // content_box  拖拽展示区域  
    if(!this.config){
        this.config = {
            fruit_height : 100,     // 水果盒子的高度
            container_width : 500,   // 最外层容器宽度设置 container
            content_padding : 10,  // 容器内边距   
            content_height : 120,  // 展示区域高度
        }
    }

    const container_box = document.querySelector(container); // 获取最外层容器组件
    let content_box_offset = $(content_box).offset();  // 计算展示区域相对于父元素偏移量
        content_box_width = $(content_box).width(),  //展示区域的宽
        content_boxX = content_box_offset.left,   //展示区域 X 坐标
        content_boxY = content_box_offset.top;    //展示区域 Y 坐标

    let	fruits = [
        {name:'苹果',color:'#f14010',id:0},
        {name:'梨子',color:'#c1d000',id:1},
        {name:'橘子',color:'#f19220',id:2},
        {name:'栗子',color:'#d1226D',id:3},
    ]
    // 遍历将fruits  生成html
    let htmlstr = $(container).html();
    htmlstr += '<ul>';

    fruits.forEach((item)=>{
        htmlstr+=`<li data-id=${item.id} style='background:${item.color};line-height:${this.config.fruit_height}px'>${item.name}</li>`; 
    })
    htmlstr+= '</ul>'
    container_box.innerHTML = htmlstr;

    $(container).css({width:this.config.container_width,padding:this.config.content_padding})
    // 动态设置展示区域的宽，高， 内边距
    $(content_box).css({height:this.config.content_height,padding:this.config.content_padding})

    let _that = this,  // this 指向全局调用者
        showFruit = []; // 展示区域水果盒子个数
    // 给每个li添加按下  移动事件
    $(`${container} li`).each(function(i){
        let isDrag = false;   //判断是否被拖拽

        let eleX = $(this).offset().left,      // 记录鼠标按下时元素的偏移量 X
            eleY = $(this).offset().top;       // 记录鼠标按下时元素偏移量 Y

		$(this).mousedown(function(ev){
            ev.preventDefault();  //阻止默认事件

            let _index = showFruit.indexOf($(this).attr('data-id'));  // 计算点击元素在数组中的索引
            if(_index !== -1){
                showFruit.splice(_index);  // 删除将要移除的元素
            }

			currentitem = this;
            isDrag = true;
            
            let $this = this;                      // 记录当前点击li
 			$(container).mousemove(function(e){    // 在文档移动时候(绑定事件)
              
               if(!isDrag){    // 如果是false，不移动
               	return false;  
			   }
               
                // 水果盒子跟随鼠标移动
                let currentpageX = e.pageX;
                let currentpageY = e.pageY;

                let posleft = currentpageX - eleX;
                let postop = currentpageY - eleY;
                $($this).css({left:posleft,top:postop});

            $(container).mouseup(function(e){  //在文档移动时候(解绑事件)
                if(!isDrag){     // 如果是false，不移动
               	    return false;  
                }

                isDrag = false;

                let currentpageX = e.pageX;
                let currentpageY = e.pageY;

                let li_width = Math.ceil($(`${container} li`).width());   // 计算每个水果盒子宽度
                let currenthand_right = currentpageX + li_width;     // 计算水果盒子右侧坐标位置

                // 水果盒子右侧或者左侧移入展示区域
                // 当水果盒子触及展示区域边缘自动存取水果

                if(content_boxX < currenthand_right && currentpageX < content_boxX+content_box_width && content_boxY < currentpageY && currentpageY < content_boxY+_that.config.content_height){

                    var posleft = (content_boxX - eleX) + _that.config.content_padding * 2;
                    var postop = content_boxY - eleY + _that.config.content_padding * 2;
                    // 移入水果展示区域时 判断个数 根据个数计算每个li 的 left 位置
                    // 个数 * 20  
                    posleft = posleft + showFruit.length * 20;
                    $($this).css({left:posleft,top:postop,zIndex:showFruit.length});
                    showFruit.push($($this).attr('data-id'))

                }else{
                    // 此处定位是相对与自己本身的位置  所以重置为0
                    $($this).css({left:0,top:0});
                }

               })  // mouseup 结束

			})  // mousemove 结束
		})   // mousedown 结束
    })  // each 结束
   

}
```
##### drag.css
```
  *{
    list-style: none;
    box-sizing: border-box;
    font-size:18px;
    padding: 0;
    margin:0;
    user-select: none;
  }
  #select_box{
    border: 1px solid #ddd;
    overflow: hidden;
    
  }
  ul{
    width:100%;
  }
  li{
    float: left;
    width: 33%;
    text-align: center;
    position: relative;
  }
  li:hover{
    cursor: pointer;
    box-shadow:0 0 3px 3px #666;
    z-index: 10000;
  }
  /* content_box */
  #content_box{
    margin: 0 50px 50px 50px;
    position: relative;
    overflow: hidden;
    border: 1px solid #0d0;
  }
```
