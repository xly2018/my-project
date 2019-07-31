---
title: jquery contextmenu 右击菜单插件  
date: 2019-07-23 
tags:
- jquery         
---
### 添加插件 
```html
 <link rel="stylesheet" href="${ctxStatic}/components/contextmenu/jquery.contextMenu.min.css">
 <script src="${ctxStatic}/components/contextmenu/jquery.contextMenu.js"></script>
 <script src="${ctxStatic}/components/contextmenu/jquery.ui.position.min.js"></script>
```
### 使用 
```js
$.contextMenu({
        selector: '#selector',//选择器
        items: {
            copy: {
                name:"特殊标记",//名称
                icon: "fa-heart",//图标
                callback: function (key, opt) {//回调
                    
                }
            }
        }
    });
```
### 效果
![](https://xly2018.github.io/long/img/bfbb8309.gif)
### 结合jqgrid
```js
//添加onRightClickRow事件
onRightClickRow: function (rowid, iRow, iCol, e) {
    jseasyui.sys.student.main.initContextmenu(rowid);
},

/*初始化右击菜单 为表格每一行绑定右击菜单*/
jseasyui.sys.student.main.initContextmenu = function (id) {
    var rowData = $("#table_list").jqGrid('getRowData', id);
    var selector = '#' + id;
    //重新注册右击菜单插件
    $.contextMenu('destroy', selector);
    $.contextMenu({
        selector: selector,
        items: {
            copy: {
                name: rowData.specialMarkers == 0 ? "特殊标记" : "取消特殊标记",
                icon: rowData.specialMarkers == 0 ? "fa-heart" : "fa-heart-o",
                callback: function (key, opt) {
                    jseasyui.ajaxJson('student/markers', {id: id}, function (data) {
                        jseasyui.sys.student.main.query();
                    })
                }
            }
        }
    });
}

```