# 简单的实现列表的分页插件

由于现在公司内部写的分页功能实在太乱了，所以，自己实现一个简单的分页插件，这样在其他同学在使用的时候，只需要在页面上添加相应的标签就可以了。(由于本人CSS能力实在是渣，所以就写的很不专业了......)

## 首先做一个分析
我们做一个分页，首先需要哪些数据? 大致应该有如下一个数据
* 总数
* 总的页码数
* 当前页码
* 每页显示的数量(这里我默认使用了每页显示20条数据)
......

有这些数据，那必要需要有一个地方能够存储这些数据值。因为要和后端进行交互，所以，这些数据值还是依赖于后端功能。我将这些数据值存在HTML代码中:

```html
<form action="", method="post">
	<input hidden id="pageResultNo" name="pageNo" value="当前页的页码值" />
</form>
<div id="pageDiv" style="padding-left: 40%; padding-bottom: 5%;"
         data-total="后端返回的总的数据的数量"
         </div>
```

总数有了，当前页码也有了，接下来就是页码数啦!

```javascript
var pageNum = parseInt(pageNum)
if (pageNum == "" || pageNum == 0 || pageNum == void 0) {
	pageNum = 0;
}
var totalPages = Math.round(total / 20);
```
totalPages 就是我们总的页码数量了。

好了，该有的数据，我们都有了。那么，剩下的就是实现的逻辑思路了......

我们先来想一想一般分页插件的一般样式大致为如下:

首页 + [上一页] 页码 ... 页码 [下一页] 尾页

### 首页
首页这里好办，因为首页的页码始终是1，所以，代码大致就是这样:

```javascript
<a data-id="1">首页</a>
```
我们将页码存在data-id属性中。

### 尾页
既然首页出来了，那么，尾页就差不多了，只不过数据不一样而已:

```javascript
<a data-id="'+totalPages+'">尾页</a>
```
我们只要将我们总的页码数放入就可以了。

### 中间的页码部分

这一部分相对来说比较复杂一点。这里最重要的一点就是要依据当前页作为一个相对目标，从而做页码的移动展示。

```javascript
for (var i=1; i<2; i++) {
	if (pageNum-i > 1) {
		str = '<a class="pageNo" style="font-size: 20px" data-id="'+(pageNum - i) +'"><span>' + (pageNum - i) + '</span>&nbsp;</a>' + str;
            }
  if (pageNum + i < totalPages) {
  		str = str + '<a class="pageNo" style="font-size: 20px" data-id="'+(pageNum + i) +'"><span>' + (pageNum + i) + '</span>&nbsp;</a>';
  }
}
```
上面这部分代码的意思就是当前页之前和之后个显示2个页码数量。

当前页大于第一页的时候，需要显示上一页:

```javascript
if (pageNo > 1) {
	str = '<a data-id='+pageNo - 1+'>上一页</a>' + str;
}
```

如果页码数超过我们需要显示的页码数量，我们就要用'...'来替换

```javascript
if (pageNo + 3 < totalPages) {
	str = str + ' ...';
}
```

当前页小于总页数显示下一页:

```javascript
if (pageNo < totalPages) {
	str = str + '<a data-id='+pageNo + 1+'>下一页</a>'
}
```

剩下的就是想页面填充数据了:

```javascript
$('#pageDiv').append(str);
```


### 下面的就是通过点击触发sumbit

```javascript
/**
 * 用于控制是否是表单按钮所触发还是链接按钮触发
 * @type {boolean}
 */
var TRIGGER_FLAG = false;

 /**
 * 跳转到指定页面
 */
 $(document).on('click', '.pageNo', function () {
 	$('#pageResultNo').val();
 	var pageNo = $(this).data().id;
	$('#pageResultNo').val(pageNo);
 	TRIGGER_FLAG = true;
  	$('#J_Search').click();
});
 /**
 * 跳转到下一页
 */
 $(document).on('click', '#nextPage', function () {
 	var pageNo = $('#pageResultNo').val();
 	pageNo = parseInt(pageNo) + 1;
 	$('#pageResultNo').val(pageNo);
 	TRIGGER_FLAG = true;
 	$('#J_Search').click();
 });
 /**
  * 跳转到上一页
  */
  $(document).on('click', '#prevPage', function () {
 		var pageNo = $('#pageResultNo').val();
		pageNo = parseInt(pageNo) - 1;
		$('#pageResultNo').val(pageNo);$('#pageResultNo').val();
			TRIGGER_FLAG = true;
			$('#J_Search').click();
 });

 $('#J_Search').on('click', function () {
 	var pageNo = $('#pageResultNo').val();
 	/**
 	 * 当时表单按钮触发的时候,需要将页码设置为第一页
 	 */
 	if (pageNo == "" || pageNo == 0 || pageNo == void 0 || !TRIGGER_FLAG) {
 		$('#pageResultNo').val(1);
 	}
 });
```

这里要注意的就是，由于我们form提交的时候，会触发submit的click事件，但是当我们通过页码跳转的时候，需要传递页码pageNo。但是，当我们直接通过submit按钮去操作的话，就要把当前的页码设置为1做一个初始化的操作了。

---
### 最后上总的代码
```html
<form action="", method="post">
	<input hidden id="pageResultNo" name="pageNo" value="当前页的页码值" />
</form>
<div id="pageDiv" style="padding-left: 40%; padding-bottom: 5%;"
         data-total="后端返回的总的数据的数量"
         </div>
```

我只要在HTML中添加这几部分即可。填写后端返回的页码，和中的数据量。

---
javascript 部分的功能代码

```javascript
function showPage() {
        var total = $('#pageDiv').data().total; // 总共多少条数据
        var pageNo = $('#pageResultNo').val();

        pageNo = parseInt(pageNo);
        if (pageNo == "" || pageNo == 0 || pageNo == void 0) {
            pageNo = 0;
        }
        var totalPages = Math.round(total / 20);
        var str = '<span style="color: #008A03; font-size: 20px">' + pageNo + '</span>&nbsp;';
        if (total == "" || total == 0 || total == void 0) {
            str = "暂无数据"
        }
        /**
         * 首页链接
         * @type {string}
         */
        var firstPage = '';
        if (totalPages > 2 && pageNo != 1) {
            firstPage = '<a class="pageNo" style="font-size: 20px" data-id="1">首页<a/>&nbsp;&nbsp;';
        }
        /**
         * 尾页链接
         */
        var lastPage = '';
        if (totalPages > 2 && pageNo != totalPages) {
            lastPage = '&nbsp;&nbsp;<a class="pageNo" style="font-size: 20px" data-id="'+totalPages+'">尾页<a/>';
        }

        for (var i=1; i<3; i++) {
            if (pageNo-i > 1) {
                str = '<a class="pageNo" style="font-size: 20px" data-id="'+(pageNo - i) +'"><span>' + (pageNo - i) + '</span>&nbsp;</a>' + str;
            }
            if (pageNo + i < totalPages) {
                str = str + '<a class="pageNo" style="font-size: 20px" data-id="'+(pageNo + i) +'"><span>' + (pageNo + i) + '</span>&nbsp;</a>';
            }
        }
        if (pageNo > 1) {
            str = firstPage + '&nbsp;&nbsp;&nbsp;<a id="prevPage" style="font-size: 20px">&lt;&nbsp;<span>上一页</span></a>&nbsp;' + ' ' + str;
        }

        if (pageNo + 3 < totalPages) {
            str = str + ' ...';
        }

        if (pageNo < totalPages) {
            str = str + ' <a class="pageNo" style="font-size: 20px" data-id="'+totalPages+'"><span>' + totalPages + '</span></a>&nbsp;<a id="nextPage" style="font-size: 20px">下一页&nbsp;&gt;</a>' + lastPage;
        }

        $('#pageDiv').append(str);

        /**
         * 用于控制是否是表单按钮所触发还是链接按钮触发
         * @type {boolean}
         */
        var TRIGGER_FLAG = false;

        /**
         * 跳转到指定页面
         */
        $(document).on('click', '.pageNo', function () {
            $('#pageResultNo').val();
            var pageNo = $(this).data().id;
            $('#pageResultNo').val(pageNo);
            TRIGGER_FLAG = true;
            $('#J_Search').click();
        });
        /**
         * 跳转到下一页
         */
        $(document).on('click', '#nextPage', function () {
            var pageNo = $('#pageResultNo').val();
            pageNo = parseInt(pageNo) + 1;
            $('#pageResultNo').val(pageNo);
            TRIGGER_FLAG = true;
            $('#J_Search').click();
        });
        /**
         * 跳转到上一页
         */
        $(document).on('click', '#prevPage', function () {
            var pageNo = $('#pageResultNo').val();
            pageNo = parseInt(pageNo) - 1;
            $('#pageResultNo').val(pageNo);$('#pageResultNo').val();
            TRIGGER_FLAG = true;
            $('#J_Search').click();
        });

        $('#J_Search').on('click', function () {
            var pageNo = $('#pageResultNo').val();
            /**
             * 当时表单按钮触发的时候,需要将页码设置为第一页
             */
            if (pageNo == "" || pageNo == 0 || pageNo == void 0 || !TRIGGER_FLAG) {
                $('#pageResultNo').val(1);
            }
        });
```
---
## 总结

能力有限，代码写的确实不怎么样......
