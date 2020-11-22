---
title: layui表格及工作流二次开发实践1
categories:
- 后端
tags: activity
date:
---
> 学而时习之不亦说乎

### layui官网

<a href="https://www.layui.com" target="_blank">layui官网</a>

### 安装包下载
进入官网点击立即下载  下载后的文件如下：
![NTZlB4.png](https://s1.ax1x.com/2020/07/01/NTZlB4.png)

注：mock.js 是我自己加的，模拟后台接口数据用的

### chooseRoleDialog.html列表弹窗页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="layui/css/layui.css">
    <script src="layui/layui.js"></script>
    <script src="editor-app/libs/jquery_1.11.0/jquery.min.js"></script>
    <script src="layui/mock.js"></script>
</head>
<style>
.user-dialog{
    width: 900px;
    display: block;
    margin: auto;
}
.user-dialog .tree-area{
    width: 100%;
    display: block;
    float: left
}
.user-dialog .table-area{
    display: block;
    float: left
}
.defined-area{
    padding-left: 80%;
    padding-top: 3%;
}
.layui-form-label{
    width: 95px;
}
</style>
<body>
<div class="user-dialog">
<!--    <div class="tree-area">-->
<!--        <div id="tree_id" class="demo-tree-more"></div>-->
<!--    </div>-->
    <div class="table-area">
        <div class="layui-form-item">
            <div class="layui-inline">
                <label class="layui-form-label">角色分类编码</label>
                <div class="layui-input-inline">
                    <input type="tel" id="roleSortCode" name="roleSortCode"  autocomplete="off" class="layui-input">
                </div>
            </div>
            <div class="layui-inline">
                <label class="layui-form-label">角色分类名称</label>
                <div class="layui-input-inline">
                    <input type="text" id="roleSortName" name="roleSortName" autocomplete="off" class="layui-input">
                </div>
            </div>
            <button type="button" class="layui-btn" onclick="queryTable()">查询</button>
            <button type="button" class="layui-btn" onclick="resetQuery()">重置</button>
        </div>
        <table class="layui-hide" id="user_table" lay-filter="test"></table>
        <div id="page"></div>
<!--        <div class="layui-btn-container defined-area">-->
<!--            <button type="button" class="layui-btn" onclick="canSave()">确定</button>-->
<!--            <button type="button" class="layui-btn" onclick="closeDialog()">取消</button>-->
<!--        </div>-->
    </div>
</div>

<script>
    layui.config({
        version: '1591161919724' //为了更新 js 缓存,可忽略
    });
    var userStr = "";
    var selectUser = [];
    var selectUserId = [];
    var requestUrl = "http://xx.xx.xx.xx:8890/upms/roleSort/pageRoleSort";
    var getTableData;
    var requestParams = {pageNum:1,pageSize:10,roleSortCode:"",roleSortName:""}
    var response = {};
    var tree,layer,util,laypage,table;
    layui.use(['tree', 'util','laypage', 'layer', 'table','element', 'slider'], function() {
        tree = layui.tree
        layer = layui.layer
        util = layui.util
        laypage = layui.laypage //分页
        table = layui.table //表格

        //页面第一次请求 默认 1页 10条
        getTableData = function getTableDatas(params,response) {
            $.ajax({
                url: requestUrl+'?pageNum='+params.pageNum+'&pageSize='+params.pageSize
                    +'&roleCode='+params.roleSortCode+'&roleName='+params.roleSortName,
                type:'GET',
                async: false,
                dataType:'json',
                success:function(res){
                    response = res.data;
                    rendTable(response);
                    page(response);
                    // 表格监听
                    table.on('checkbox(test)', function(obj){
                        if(obj.checked && obj.type=='all'){// 全选
                            for(var i in response.records){
                                var rowData = response.records[i];
                                if(selectUserId.indexOf(parseInt(rowData.id)) == -1){
                                    selectUser.push(rowData);
                                }
                            }
                        }else if(!obj.checked && obj.type=='all'){// 全不选
                            for(var i = 0;i< selectUser.length;i++){
                                for(var j = 0;j< response.records.length;j++){
                                    var rowData = response.records[j];
                                    if(parseInt(selectUser[i].id) == parseInt(rowData.id)){
                                       selectUser.splice(i,1);
                                    }
                                }
                            }
                        }else if (obj.checked){//单个选
                             selectUser.push(obj.data);
                        }else if (!obj.checked){//单个取消选择
                            for(var i in selectUser){
                                if(selectUser[i].id == obj.data.id){
                                    selectUser.splice(i,1);
                                }
                            }
                        }
                        selectUserId = [];
                        for(var i in selectUser){
                            selectUserId.push(selectUser[i].id);
                        }
                    });
                },
                error:function(status){
                    response = {};
                }
            });
        }

        // 重构数据 增加选中属性
        function rebuildTableData(response){
            var role_ids= window.parent.document.getElementById("roles_id").value;
            if(role_ids){
                var select_list = role_ids.split(";");
                // 数据回显选中  这部分逻辑可以放到后台实现
                for(var r in select_list){
                    var id = select_list[r].substring(0,select_list[r].indexOf(","));
                    var roleSortName = select_list[r].substring(select_list[r].indexOf(",")+1,select_list[r].lastIndexOf(","));
                    var roleSortCode = select_list[r].substring(select_list[r].lastIndexOf(",")+1,select_list[r].length);
                    var map = {};
                    map.id = id;
                    map.roleSortName = roleSortName;
                    map.roleSortCode = roleSortCode;
                    if(id){
                        selectUser.push(map);
                        selectUserId.push(parseInt(id));
                    }
                }
                for(var i in response.records){
                    var rowData = response.records[i];
                    if(selectUserId.indexOf(rowData.id)!=-1){
                        rowData.LAY_CHECKED = true;
                    } else {
                        rowData.LAY_CHECKED = false;
                    }
                }
            }
            console.log(response.records);
        }

        // 表格渲染
        function rendTable(response) {
            rebuildTableData(response);
            table.render({
                elem: '#user_table',
                cols:  [[ //表头
                    {type: 'checkbox', fixed: 'left'}
                    ,{field: 'id', title: 'id', hide:true}
                    ,{field: 'roleSortCode', title: '角色分类编码', width:'30%'}
                    ,{field: 'roleSortName', title: '角色分类名称', width: '30%', sort: true, totalRow: true}
                    ,{field: 'remark', title: '备注', width:'40%', sort: true}
                ]],
                data: response.records, // 数据
                limit: response.total, // 显示的条数
                //page: true, // 开启分页
                done: function(res, curr, count){

                }
            });
        }
        // 分页
        function page(response) {
            laypage.render({
                elem: 'page',
                count: response.total,
		curr: response.current,
                limit: requestParams.pageSize,
                limits: [10, 20, 30, 40, 50],
                layout: ['count', 'prev', 'page', 'next', 'limit', 'skip'],
                jump: function (obj, first) {
                    //首次不执行
                    if (!first) {
                        requestParams.pageNum = obj.curr;
                        requestParams.pageSize = obj.limit;
                        getTableData(requestParams,response)
                    }
                },
                yes:function(index, layero){
                    layer.close(index);//需手动关闭 弹出层
                }
            });
        }
        //getTableData(requestParams, response);
	queryTable();
    });

    // 查询
    function queryTable(){
        requestParams.roleSortName = $("#roleSortName").val()==""?'':$("#roleSortName").val().trim();
        requestParams.roleSortCode = $("#roleSortCode").val()==""?'':$("#roleSortCode").val().trim();
        getTableData(requestParams, response);
    }

    // 重置
    function resetQuery(){
        $("#roleSortCode").val("");
        $("#roleSortName").val("");
        requestParams = {pageNum:1,pageSize:10,roleSortCode:"",roleSortName:""};
        getTableData(requestParams, response);
    }

    // 确定
    function canSave(){
        userStr = '';
        for(var i in selectUser){
            if(i == selectUser.length-1){
                userStr += selectUser[i].id+","+selectUser[i].roleSortName+","+selectUser[i].roleSortCode;
            }else{
                userStr += selectUser[i].id+","+selectUser[i].roleSortName+","+selectUser[i].roleSortCode+";";
            }
        }
        console.log(userStr);
        return userStr;
    }

    //function closeDialog(){
    //    var index = parent.layer.getFrameIndex(window.name); //先得到当前iframe层的索引
    //    parent.layer.close(index);
    //}
    // --树形结构
    // tree.render({
    //     elem: '#tree_id'
    //     ,data: data1
    //     ,showLine: true,
    //     click: function(obj){
    //         layer.msg(JSON.stringify(obj.data));
    //         var currTreeId = obj.data.id;
    //         //上一次点击的树节点id
    //         var beforeTreeId =  $('#tree_id').val();
    //         if (currTreeId !== beforeTreeId){
    //             $('div [data-id="'+beforeTreeId+'"] div .layui-tree-txt').first().css('color','');
    //             $('div [data-id="'+currTreeId+'"] div .layui-tree-txt').first().css('color','#009688');
    //             $('#tree_id').val(obj.data.id);
    //         }
    //     }
    // });
</script>
</body>
</html>
```
### 说明
至此layui关于表格的查询展示及勾选数据的回显都已全部处理完毕；目前做这个功能是activity工作流的二次开发，本页面实现的功能是工作流节点增加一个角色，点击弹出此页面选择角色。然后下次点击上次选中的数据需要被回显即被勾选上了！坑点：layui的表格复选框不支持jquery模拟点击事件，且如果用样式控制选中下次点击时复选框会异常！关于选中与否，使用表格字段属性LAY_CHECKED控制！

关于工作流绘制的二次开发"com-activity-demo"详见本人<a href="https://gitee.com/viEcho/com-activity-demo" target="_blank">gitee地址</a>.下载项目后需要在对应的库生成对应的25张表。

注意：此项目工作流前端用的是angular.js，需要遵循其模板语法；此处layui这个表格弹窗，我采用的是iframe标签引入到对应的模板页面中的，所以涉及到iframe 标签及其父子页面传值调用的方法，参考博客<a href="https://www.cnblogs.com/llljpf/p/7435526.html" target="_blank">iframe父子页面通信</a>
此外，接口若有跨域需要在服务端添加允许跨域配置，且改写了前端代码后浏览器会有本地缓存，crtl+f5刷新后浏览或直接采用无痕浏览模式查看！

### 小结
6月初回了趟老家，家中有事耽误一个星期转户口武汉看房买房又近一个星期；也许后面有了想法写篇买房日记啥的，哈哈哈。后面继续写写activity之类的实践篇，及补上springboot的学习总结篇！
