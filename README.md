今天应同学要求，需要写一个像Excel那样框选高亮，并且实现框选区域实现反选功能。要我用原生js写，由于没什么经验翻阅了很多资料，第一次写文章希望各位指出不足！！

**上来先建表**
***
```
<div class="table-container" >
    <table class="table" id="dataGrid" align="center">
        <tr id="title">
            <th>&nbsp;</th>
            <th>水果</th>
            <th>蔬菜</th>
        </tr>
        <tr id="tb1" class="fileDiv">
            <td><input type="checkbox" name="idarray" value="1"/></td>
            <td>苹果</td>
            <td>菠菜</td>
        </tr>
        <tr id="tb2" class="fileDiv">
            <td><input type="checkbox" name="idarray" value="2"/></td>
            <td>梨</td>
            <td>白菜</td>
        </tr>
        <tr id="tb3" class="fileDiv">
            <td><input type="checkbox" name="idarray" value="3"/></td>
            <td>葡萄</td>
            <td>萝卜</td>
        </tr>
        <tr id="tb4" class="fileDiv">
            <td><input type="checkbox" name="idarray" value="4"/></td>
            <td>桃子</td>
            <td>土豆</td>
        </tr>
        <tr id="tb5" class="fileDiv">
            <td><input type="checkbox" name="idarray" value="5"/></td>
            <td>苹果</td>
            <td>菠菜</td>
        </tr>
        以下省略十行……
    </table>
</div>
</body>
```
***样式***
```
<style>
        body {  padding: 0;  margin: 0;  overflow-y: hidden;  }
        .table {  width: 990px;  height: 850px;  }
        .table tr {  height: 50px;  overflow: scroll  }
        .table-container {
            position: absolute;
            top: 0;
            left: 200px;
            height: 750px;
            width: 990px;
            overflow-y: scroll;
            overflow-x: hidden;
        }
         #title {  background-color: #BDE4F4;  }
        .fileDiv {  background-color:#FEFF89; }
        .seled {  border: 1px solid red;  background-color: #D6DFF7;  }
    </style>
```
***
***效果***
![](http://upload-images.jianshu.io/upload_images/9736114-54e640e19dbdf6ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***
重头戏js部分开始
---
```
<script type="text/javascript">
        var allpro2 = [];
        var index = 0;

        window.onload = function () {
            document.onmousedown = function () {
                var evt = window.event || arguments[0];

                var startX = (evt.x || evt.clientX);
                var startY = (evt.y || evt.clientY);

                if (startX > 220 && startX < 1210) {
                    var selList = [];
                    var fileNodes = document.getElementsByTagName("tr");
                    for (var i = 0; i < fileNodes.length; i++) {
                        if (fileNodes[i].className.indexOf("fileDiv") != -1) {
                            fileNodes[i].className = "fileDiv";
                            selList.push(fileNodes[i]);
                        }
                    }
                    allpro2 = [];       //选错区域后重选则清空数组
                    var yheight = document.getElementById("tablewrapper");
                    if (yheight.scrollTop > 0) {      //判断初始滚动条是否为0 为后面是否加滚动条高度设置判断值
                        var istrue = true;
                    }
                    paint(yheight, selList, startX, startY, istrue);

                }
            }
        };


        function paint(yheight, selList, startX, startY, istrue) {
            var isSelect = true;
            var selDiv = document.createElement("div");
            selDiv.style.cssText = "position:absolute;width:0px;height:0px;font-size:0px;margin:0px;padding:0px;border:1px dashed #0099FF;background-color:#C3D5ED;z-index:1000;filter:alpha(opacity:60);opacity:0.6;display:none;";
            selDiv.id = "selectDiv";
            document.body.appendChild(selDiv);

            selDiv.style.left = startX + "px";
            selDiv.style.top = startY + "px";

            var _x = null;
            var _y = null;


            document.onmousemove = function () {
                evt = window.event || arguments[0];
                if (isSelect) {
                    if (selDiv.style.display == "none") {
                        selDiv.style.display = "";
                    }

                    _x = (evt.x || evt.clientX);
                    _y = (evt.y || evt.clientY);

                    selDiv.style.left = Math.min(_x, startX) + "px";
                    selDiv.style.top = Math.min(_y, startY) + "px";
                    selDiv.style.width = Math.abs(_x - startX) + "px";
                    selDiv.style.height = Math.abs(_y - startY) + "px";
                }
            };


            document.onmouseup = function () {
                if (selDiv) {
                    if (istrue) {
                        selDiv.style.height = Math.abs(_y - startY) + "px";
                        selDiv.style.top = Math.min(_y, startY) + yheight.scrollTop + "px";
                    } else {
                        selDiv.style.height = Math.abs(_y - startY) + yheight.scrollTop + "px";
                    }
                    var _l = selDiv.offsetLeft, _t = selDiv.offsetTop;
                    var _w = selDiv.offsetWidth, _h = selDiv.offsetHeight;
                    for (var i = 0; i < selList.length; i++) {
                        var sl = selList[i].offsetWidth + selList[i].offsetParent.offsetParent.offsetLeft; //获取父元素table距离左边距离
                        var st = selList[i].offsetHeight + selList[i].offsetTop;

                        if (sl > _l && st > _t && selList[i].offsetLeft < _l + _w && selList[i].offsetTop < _t + _h) {
                            if (selList[i].className.indexOf("seled") == -1) {
                                selList[i].className = selList[i].className + " seled";
                            }
                        } else {
                            if (selList[i].className.indexOf("seled") != -1) {
                                selList[i].className = "fileDiv";
                            }
                        }
                    }
                }

                isSelect = false;
                if (selDiv) {
                    document.body.removeChild(selDiv);
                    showSelDiv(selList);
                }
                revs1();
                selList = null, _x = null, _y = null, selDiv = null, startX = null, startY = null, evt = null;
            }
        }

        function showSelDiv(arr) {         //确定选中范围更新数组allpro2
            for (var i = 0; i < arr.length; i++) {
                if (arr[i].className.indexOf("seled") != -1) {
                    var allpro1 = document.getElementById(arr[i].id).getElementsByTagName("td")[0].getElementsByTagName("input")[0];
                    allpro2.push(allpro1);
                }
            }
        }


        function revs1() {        //反选函数
            if (allpro2.length > 0) {
                for (var i = 0; i < allpro2.length; i++) {
                    if (allpro2[i].type == "checkbox" && allpro2[i].checked == true) {
                        allpro2[i].checked = false;
                    } else if (allpro2[i].type == "checkbox" && allpro2[i].checked == false) {
                        allpro2[i].checked = true;
                    }
                }
            }
            allpro2 = [];
        }
    </script>
```

![效果动态图.gif](http://upload-images.jianshu.io/upload_images/9736114-6a75cd59a1bedce9.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




