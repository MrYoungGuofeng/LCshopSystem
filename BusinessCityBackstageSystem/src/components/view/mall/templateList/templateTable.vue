<template>
<div>
    <el-table
    :data="datalist"
    :border="isBorder"
    @selection-change='showextra'
    @cell-click='showMemberInfo'
    :default-sort = "{prop: 'date', order: 'descending'}"
    v-loading="this.listLoading"
    :stripe='true'
    style="width: 100%"
    height='500'>
        <el-table-column
        fixed
        type="index"
        label="N"
        :index="indexMethod">
        </el-table-column>
        <el-table-column
        fixed
        type="selection" 
        width="55">
        </el-table-column>
        <el-table-column class='borderRight' 
        fixed 
        prop="templateName" 
        label="模板名称" 
        width='260'>
        </el-table-column>
        <el-table-column
        label="模板分类">
        <template slot-scope="scope">
            <span>{{ scope.row.templateType == 1?'首页模板':scope.row.templateType == 2?'活动模板':scope.row.templateType == 3?'详情模板':'自定义'}}</span>
        </template>
        </el-table-column>
        <el-table-column
        prop="createTime"
        label="创建时间"
        width='170'>
        </el-table-column>
        <el-table-column
        label="模板地址"
        width='260'>
        <template slot-scope="scope">
            <span>{{ apis+'navBottom?id='+scope.row.templateID }}</span> 
            <!-- +scope.row.templateID -->
        </template>
        </el-table-column>
        <el-table-column
        prop="description"
        label="模板描述"
        width='200'
        >
        </el-table-column>
        <el-table-column
        prop="types"
        width='220'
        label="操作">
         <template slot-scope="scope">
              <!-- <el-button type="text"  size="small" @click="qRcodeSee(scope.$index, scope.row,$event)">二维码</el-button> -->
             <el-button type="text"  size="small" @click="handleSee(scope.$index, scope.row,$event)">浏览</el-button>
            <el-button type="text"  size="small" @click="handleEdit(scope.$index, scope.row,$event)">编辑</el-button>
            <el-button type="text"  size="small" @click="handleSwicth(scope.$index, scope.row,$event)">
                <span :class="scope.row.isEnabled == true?'templateStausColorGreen':scope.row.isEnabled == false?'templateStausColorRed':''">{{ scope.row.isEnabled == true?'启用中':scope.row.isEnabled == false?'停用中':''}}</span>
                </el-button>
            <el-button type="text" size="small" @click="handleDelete(scope.$index, scope.row,$event)">删除</el-button>
        </template>
        </el-table-column> 
    </el-table>
     <div class="dialog" id="dialog">
        <div class="dialog_head" id="move_part" @mouseover="phoneDialog()">可拖拽部分</div>
        <div class="dialog_content">
            <iframe :src="iframeLink" frameborder="0" name="ifrmname" class="template_iframe"></iframe>
        </div>
        <div class="button close_button" id="close" @click="closeBower"><a href="#" style="font-size:23px;">&times;</a>
        </div>
    </div>
    </div>
</template>
<script>
/* eslint-disable */
//@row-click="showMemberInfo()"
export default {
    prop:['listLoading'],
    data(){
        return {
            isBorder:false,
            datalist:[],
            showLeft:0,
            pageIndex:1,
            iframeLink:'',
            apis:'http://www.shhongzhiyun.com/'
        }
    },
    created:function(){
        this.$root.$on('pageIndex',(data) => {
            this.pageIndex = data.value
            this.getDate(this.pageIndex)
        })
        this.getDate(1)
        this.$root.$on('getDatezdy',(data)=>{
             this.getDate(data);
        })
        this.$root.$on('dataListBox',(data)=>{
            this.datalist = data
        })
        this.phoneDragFn()
    },
    methods:{
        changeURLArg(url,arg,arg_val){
            var pattern=arg+'=([^&]*)';
            var replaceText=arg+'='+arg_val; 
            if(url.match(pattern)){
                var tmp='/('+ arg+'=)([^&]*)/gi';
                tmp=url.replace(eval(tmp),replaceText);
                return tmp;
            }else{ 
                if(url.match('[\?]')){ 
                        return url+'&'+replaceText; 
                }else{ 
                    return url+'?'+replaceText; 
                } 
            }
        },
        handleSee(index, row,event){ // 浏览某个模板
          let templateId = row.templateID
          //给后台设置浏览的ID
         history.pushState(null,null,this.changeURLArg(window.location.href,'id',templateId));
         this.iframeLink = this.apis+"navBottom?id="+templateId
          document.getElementById('dialog').style.display = 'block';
          this.autoCenter(document.getElementById('dialog'));
        },
        closeBower(){
             document.getElementById('dialog').style.display = 'none';
        },
        autoCenter(el){
            //获取可见窗口大小
            var bodyW = document.documentElement.clientWidth;
            var bodyH = document.documentElement.clientHeight;
            //获取对话框宽、高
            var elW = el.offsetWidth;
            var elH = el.offsetHeight;

            el.style.left = (bodyW - elW)/8 + 'px';
            el.style.top = (bodyH - elH)/10 + 'px';
        },
        phoneDragFn(){
            //禁止选中对话框内容
            if(document.attachEvent) {//ie的事件监听，拖拽div时禁止选中内容，firefox与chrome已在css中设置过-moz-user-select: none; -webkit-user-select: none;
                document.getElementById('dialog').attachEvent('onselectstart', function() {
                return false;
                });
            }
            function autoCenter(el){
                //获取可见窗口大小
                var bodyW = document.documentElement.clientWidth;
                var bodyH = document.documentElement.clientHeight;
                //获取对话框宽、高
                var elW = el.offsetWidth;
                var elH = el.offsetHeight;
                el.style.left = (bodyW - elW)/8 + 'px';
                el.style.top = (bodyH - elH)/10 + 'px';
            };
            //窗口大小改变时，对话框始终居中
            window.onresize = function(){
                autoCenter( document.getElementById('dialog'));
            };
        },
        phoneDialog(){
                //alert(111)
                //声明需要用到的变量
                let mx = 0,my = 0;      //鼠标x、y轴坐标（相对于left，top）
                let dx = 0,dy = 0;      //对话框坐标（同上）
                let isDraging = false;      //不可拖动
                //鼠标按下
                document.getElementById('move_part').addEventListener('mousedown',function(e){
                    var e = e || window.event;
                    mx = e.pageX;      //点击时鼠标X坐标
                    my = e.pageY;      //点击时鼠标Y坐标
                    dx = document.getElementById('dialog').offsetLeft;
                    dy = document.getElementById('dialog').offsetTop;
                    isDraging = true;      //标记对话框可拖动
                });
                document.onmousemove = function(e){
                    var e = e || window.event;
                    var x = e.pageX;      //移动时鼠标X坐标
                    var y = e.pageY;      //移动时鼠标Y坐标
                    if(isDraging){        //判断对话框能否拖动
                        var moveX = dx + x - mx;      //移动后对话框新的left值
                        var moveY = dy + y - my;      //移动后对话框新的top值
                        document.getElementById('dialog').style.left = moveX +'px';       //重新设置对话框的left
                        document.getElementById('dialog').style.top =  moveY +'px';     //重新设置对话框的top

                    };
                };
                document.getElementById('move_part').addEventListener('mouseup',function(){
                            isDraging = false;
                        });
                //设置拖动范围
                var pageW = document.documentElement.clientWidth;
                var pageH = document.documentElement.clientHeight;
                var dialogW = document.getElementById('dialog').offsetWidth;
                var dialogH = document.getElementById('dialog').offsetHeight;
                var maxX = pageW - dialogW;       //X轴可拖动最大值
                var maxY = pageH - dialogH;       //Y轴可拖动最大值
                //moveX = Math.min(Math.max(0,moveX),maxX);     //X轴可拖动范围
                //moveY = Math.min(Math.max(0,moveY),maxY);     //Y轴可拖动范围

               // document.getElementById('dialog').style.left = moveX +'px';       //重新设置对话框的left
                //document.getElementById('dialog').style.top =  moveY +'px';        //重新设置对话框的top
            },
            handleDelete(index, row,event) { //  删除某一个模板
                let that = this;
                console.log(row);
                this.$confirm('确定删除 "'+row.templateName+'"吗?', '提示', 
                    {confirmButtonText: '确定',cancelButtonText: '取消',type: 'warning'})
                .then(() => {
                    that.$message({
                        type: 'success',
                        message: '操作成功!',
                        duration:800,
                        onClose:that.$http.post('/api/product/mall/template/remove',
                            [row.templateID]
                        ).then(res => {
                            console.log(res.data.msg);
                            if(res.data.msg == '删除失败'){
                                that.$message({
                                type: 'info',
                                message: '请先停用该模板',
                                duration:800
                            }); 
                            }
                            that.getDate(1);
                        }).catch(err => {console.log(err)})
                    });
                }).catch(() => {
                    that.$message({
                        type: 'info',
                        message: '已取消删除',
                        duration:800
                    });          
                });   
            },
            handleSwicth(index,row,event){
                let that = this;
            // console.log(row);
            // console.log(row.isEnabled)
                let stateBl = !row.isEnabled
                this.$confirm('确定更改 "'+row.templateName+'"的状态吗?', '提示', 
                    {confirmButtonText: '确定',cancelButtonText: '取消',type: 'warning'})
                .then(() => {
                    that.$message({
                        type: 'success',
                        message: '更改成功!',
                        duration:800,
                        onClose:that.$http.post('/api/product/mall/template/setEnabledByIds?value='+stateBl,
                            [row.templateID]
                        ).then(res => {
                            //console.log(res.data.msg);
                            that.getDate(1);
                        }).catch(err => {console.log(err)})
                    });
                }).catch(() => {
                    that.$message({
                        type: 'info',
                        message: '已取消更改',
                        duration:800
                    });          
                });
            },
            handleEdit(index, row,event) {
            // this.$root.$emit('showWindowss',{type:'yes',rowData:row});
            // console.log(row)
                let datas = JSON.stringify(row)
                window.sessionStorage.setItem ("Template_AllData",datas);
                window.sessionStorage.setItem('Template_Type',1)
                //this.$store.dispatch('editTemplate',row)
                this.$router.push({ path: '/mallSet' })
            },
            getDate(pageIndex) {
                this.listLoading =  true;
                let url = '/api/product/mall/template/query?page='+pageIndex+'&pageSize=10';
                this.$http({
                    url: url,
                    method: 'POST',
                    // 请求体重发送的数据
                    // headers: { 'Content-Type': 'application/json' },
                    data:{
                        'templateType':1
                    },
                })
                .then(response => {
                    this.listLoading =  false;
                    this.datalist=(response.data.info.list);
                    console.log(response.data.msg)
                    this.$root.$emit('pages',response.data.info.pages)
                    this.$root.$emit('total',response.data.info.total)
            })
            .catch(error=>{
                console.log(error);
                // //         alert('网络错误，不能访问');
            })
            },
            showMemberInfo(row,column,cell,event){//  点击显示侧滑
                //console.log(row,column,cell,event)
                //  let classNum = cell.className.split('n_')[1] //  获取单元格的类名
                let labelValue = column.label
                if(labelValue == 'ID'){
                    this.showLeft = 16
                    this.$root.$emit('infoCoverShow',this.showLeft)
                    this.$root.$emit('searchPersonnelInfo',row.id)
                }
            },      
            showextra(val){
                let show=false;
                let editcan=true;
                this.multipleSelection = val
                if(this.multipleSelection.length>0){
                    show=true;
                }
                if(this.multipleSelection.length>1){
                    editcan=false;
                }
                this.$root.$emit('showlttip',{show,editcan,num:this.multipleSelection.length,datas:this.multipleSelection});
            },
            indexMethod(index) {
                return index + 1
            },
        },
        beforeDestroy(){
            this.$root.$off('pageIndex');
            this.$root.$off('getDatezdy');
            this.$root.$off('dataListBox')
        }
}
</script>
<style lang="" scoped>
.templateStausColorGreen{
    color:#50c380;
}
.templateStausColorRed{
    color:#ff3b30
}
a{text-decoration: none; color: #eee; display: block;}
    .button{       
         width: 35px;
        height: 35px;}
    .callout_button{background:#FF5B5B;margin:0 auto; }
    .callout_button:hover{background: red;}
    .close_button{ 
        width: 35px;
        height: 35px;
        border-radius: 50px;
        text-align: center;
        line-height: 32px;
        position: absolute;
        top: 20px;
        left: 264px;   
        background: #272424;
        margin: 0 auto;
    }
    .close_button:hover{background: black;}
    /* .mask{width: 100%;height: 100%;background:#000;position: absolute;top: 0px;left:0px;opacity: 0.4;z-index: 8000; display: none;-moz-user-select: none; -webkit-user-select: none;} */
    .dialog{ position: absolute;z-index: 666; display: none;-moz-user-select: none; -webkit-user-select: none;}
    .dialog_head{
        width: 325px;
        height: 695px;
        line-height: 50px;
        color: #eee;
        cursor: move;
        padding: 100px 16px;
        background-image: url(./phone.5909f66.png);
        background-repeat: no-repeat;
        background-size: 100%;
        box-sizing: border-box;
        border-radius: 45px;
    background-color: black;
    }
    .dialog_content{
        width: 92%;
        margin-left: 12px;
        height: 518px;
        position: absolute;
        background-color: #ffffff;
        top: 82px;
    }
    .template_iframe{
        width: 100%;
        height: 518px;
        background-color: #fff;
    }
</style>