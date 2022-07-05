# SpreadJS_AdaptiveRowHeight
在纯前端在线表格中实现自适应行高功能
# SpreadJS_AdaptiveRowHeight
在纯前端在线表格中实现自适应行高功能

### SpreadJS 示例，自适应行高
该示例包括使用 SpreadJS API 的演示脚本，可用于实现自适应行高
有关 SpreadJS API 的更多信息，请参阅[SpreadJS API指南]( https://demo.grapecity.com.cn/spreadjs/help/api/) 和[帮助手册]( https://help.grapecity.com.cn/pages/viewpage.action?pageId=5963808)。



### 运行步骤
1、在开始之前，请确保您已满足以下先决条件：
要运行 SpreadJS，浏览器必须支持 HTML5，客户端导入和导出 Excel 需要 IE10及以上。
请先了解 [SpreadJS 的产品使用环境]( https://www.grapecity.com.cn/developer/spreadjs/selection-guide/product-use-environment)，并申请临时部署授权激活
安装并更新NodeJS和NPM
2、克隆或下载此代码库
3、初始化控件，并运行示例脚本
#### 控件初始化
首先，创建一个新页面，并在页面上输入以下代码：
```
<!DOCTYPE html>
    <html>
    <head>
        <title>SpreadJS HTML Test Page</title>
```
2、在页面中添加对 SpreadJS 的引用。代码如下。需要注意的是，SpreadJS 提供压缩过
```
//（minified）的 JavaScript 文件和和用于调试的文件：
<script src="[Your_Scripts_Path]/gc.spread.sheets.all.xxxx.min.js" type="text/javascript"></script>
```
3、添加 CSS 文件以改变Spread.JS 的外观。默认的CSS文件名为： 
gc.spread.sheets.xxxx.css，里面包含了所有的默认样式。该 CSS 文件将会影响滚动条，筛选框及其子元素，单元格和下方标签栏的样式。引入 CSS 的代码如下：
```
//<link href="[Your_CSS_Path]/gc.spread.sheets.xxxx.css" rel="stylesheet" type="text/css"/>
```
4、添加产品授权，代码为（本地测试可以不添加）：
```
GC.Spread.Sheets.LicenseKey = "xxx";
```
5. 添加控件初始化代码。本例会在一个 id 为 “ss” 的 DOM 元素上初始化 SpreadJS：
```
<script type="text/javascript">
// Add your license
// If run this in local for testing, remove or comment below code
 GC.Spread.Sheets.LicenseKey = "xxx";

// Add your code
 window.onload = function(){
var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"),{sheetCount:3});
var sheet = spread.getActiveSheet();
 }
</script>
</head>
<body>
```
6、 创建一个 id 为 “ss” 的元素，SpreadJS 将在该 DOM 中初始化：
```
<div id="ss" style="height: 500px; width: 800px"></div>
</body>
</html>
```
#### 示例代码
```
HTML：
<p>自适应行高</p>
<div id='ss'></div>

CSS：
#ss {
    height: 400px;
    width: 100%
}
p{
    color: #336699;
    text-align: center;
}

JavaScript：
// Title：首行缩进&自适应行高
// Description：实现首行缩进并根据编辑内容自适应行高表
// Tag:自定义单元格类型、首行缩进、自适应行高
GC.Spread.Common.CultureManager.culture('zh-cn');

var spreadNS = GC.Spread.Sheets;
$(document).ready(function() {
    var spread = new GC.Spread.Sheets.Workbook(document.getElementById("ss"));
    initSpread(spread);
});
// 声明用户自定义单元格类型，并支持序列化
function EnterNewlineCellType() {
    GC.Spread.Sheets.CellTypes.Text.apply(this, arguments);
    this.typeName = "EnterNewlineCellType";
}
EnterNewlineCellType.prototype = new spreadNS.CellTypes.Text();

// 重写paint方法
EnterNewlineCellType.prototype.paint = function(ctx, value, x, y, w, h, style, options) {

    // 这里需要加一个判断，当value前端有空格时，换成\t
    var val = "";
    if (value && (typeof value === 'string') && value.constructor === String) {
        for (var i = 0; i < value.length; i++) {
            if (value[i] === " ") {
                val += "\t";
            } else {
                val += value[i];
            }
        }
    }
    spreadNS.CellTypes.Text.prototype.paint.apply(this, [ctx, val, x, y, w, h, style, options]);
};

// 动态获取当前单元格编辑框的高度
EnterNewlineCellType.prototype.getEditorValue = function(editorContext, context) {
    var editHeight = $(editorContext).height();
    var sheet = context.sheet;
    var row = context.row;
    var col = context.col;
    sheet.setTag(row, col, editHeight);
    return spreadNS.CellTypes.Text.prototype.getEditorValue.apply(this, arguments);
};

// 设置不响应SpreadJS的Enter事件
EnterNewlineCellType.prototype.isReservedKey = function(e) {
    //这个方法目的是将enter事件注销，改为DOM自己的事件。
    return (e.keyCode === GC.Spread.Commands.Key.enter && !e.ctrlKey && !e.shiftKey && !e.altKey);
};

function initSpread(spread) {
    var sheet = spread.getSheet(0);
    sheet.suspendPaint();
    sheet.setValue(1, 1, "这是很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长很长的一段话");
    sheet.getCell(1, 1).wordWrap(true);
    
    // 给第2列设置自定义单元格
        sheet.setCellType(-1, 1, EnterNewlineCellType());
    sheet.setColumnWidth(1, 200);
        sheet.autoFitRow(1);

// 设置ValueChanged事件，动态调整行高
        spread.bind(GC.Spread.Sheets.Events.ValueChanged, function (s, e) {
            var newValue = e.newValue;
            var oldValue = e.oldValue;

            if(newValue !== oldValue){
                var sheet = spread.getActiveSheet();
                var row = e.row, col = e.col;
                var span = sheet.getSpan(row, col);
                // 处理含有合并单元格的情况
                if(span){
                    var tag = sheet.getTag(row, col);
                    if(tag){
                        var editHeight = parseFloat(tag);
                        var rowCount = span.rowCount;
                        var heightAll = 0;
                        for(let i=row; i<row + rowCount; i++){
                            heightAll += sheet.getRowHeight(i);
                        }
                        if(heightAll < editHeight){
                            sheet.setRowHeight(row, editHeight - heightAll + sheet.getRowHeight(row));
                        }
                    }
                }else{
                    sheet.autoFitRow(row);
                }
            }
        });
    sheet.resumePaint();
};
```

#### 关于 SpreadJS
[SpreadJS]( https://www.grapecity.com.cn/developer/spreadjs) 是一款基于 HTML5 的纯前端表格控件，兼容 450 多种 Excel 公式，具备“高性能、跨平台、与 Excel 高度兼容”的产品特性。使用 SpreadJS，可直接在 Angular、 React、 Vue 等前端框架中实现高效的模板设计、在线编辑和数据绑定等功能，为最终用户提供高度类似 Excel 的使用体验。

