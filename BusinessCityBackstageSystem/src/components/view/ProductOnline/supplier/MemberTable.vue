<template>
    <el-table
    :data="datalist"
    @selection-change='showextra'
    @cell-click='showMemberInfo'
    :default-sort = "{prop: 'date', order: 'descending'}"
    v-loading="listLoading"
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
        width="55" >
        </el-table-column>
        <!-- <el-table-column class='borderRight' fixed prop="id" label="ID" width='360' height='100'>
        </el-table-column> -->
        <el-table-column
        prop="name"
        label="公司"
        fixed
        width='200'
        >
        </el-table-column>
        <el-table-column
        prop="contact"
        label="联系人">
        </el-table-column>
        <el-table-column
        prop="contactMobile"
        label="电话"
        >
        </el-table-column>
        <el-table-column
        prop="categoryName"
        label="类型"
        >
        </el-table-column>
        <el-table-column
        prop="levelName"
        label="级别"
        sortable>
        </el-table-column>
        <el-table-column
        label="供应商标签"
        width='500'
        sortable>
         <template slot-scope="scope">
            <el-button v-for='(item,index) in scope.row.supplierLabel' :style="{background: item.color  }" style='color:#fff;' :key='index'>{{item.name}}</el-button>
        </template>
        </el-table-column>
        <el-table-column
        prop="source"
        fixed="right"
        width='120'
        label="操作">
        <template slot-scope="scope">
            <el-button type="text" @click='handleWindow(scope.row)' size="small">编辑</el-button>
            <el-button type="text" size="small" @click='handleClick(scope.row)'>删除</el-button>
        </template>
        </el-table-column>

                
            </el-table>
</template>
<script>
/ eslint-disable /
//@row-click="showMemberInfo()"

export default {
    prop:['listLoading'],
    data(){
        return {
            datalist:[],
            showLeft:0,
            pageIndex:1,
            data:{
                categoryId:null,
                contactMobile:null,
                labelId:null,
                levelId:null,
                name:null,
                pageSize:10
            }
        }
    },
    created:function(){
        
        this.$root.$on('pageIndex',(data) => {
            this.pageIndex = data.value
            this.getDate(this.pageIndex)
        })
        this.getDate(1)
        this.$root.$on('getDatezdy',(data)=>{
             this.getDate( data)
        })
        this.$root.$on('dataListBox',(data)=>{
            this.datalist = data
        })
        this.$root.$on('search',(datas)=>{
            console.log(datas);
            this.data.categoryId=datas.supplier.categoryId===''?null:datas.supplier.categoryId;
            this.data.contactMobile=datas.supplier.contactMobile===''?null:datas.supplier.contactMobile;
            this.data.labelId=datas.supplier.labelId===''?null:datas.supplier.labelId;
            this.data.levelId=datas.supplier.levelId===''?null:datas.supplier.levelId;
            this.data.name=datas.supplier.name===''?null:datas.supplier.name;
            this.getDate(1);
            // console.log(datas.supplier);
            // this.datalist = data
        })
        
    },
    methods:{
      getDate(pageIndex) {
            this.listLoading =  true;
            this.data.pageNum=pageIndex;
            let url = '/api/product/supplierInfo/queryPageList';
            this.$http({
                url: url,
                method: 'POST',
                // 请求体重发送的数据
                //headers: { 'Content-Type': 'application/json' },
                data:this.data,
            })
            .then(response => {
                console.log(response);
                this.listLoading =  false;
                if(response.data.info==null){
                    this.datalist=[];
                }
                else{
                    this.datalist=(response.data.info.list);
                    this.$root.$emit('pages',response.data.info.pages)
                    this.$root.$emit('total',response.data.info.total)
                }
                
          })
          .catch(error=>{
              console.log(error);
              //         alert('网络错误，不能访问');
          })
        },
        showMemberInfo(row,column,cell,event){//  点击显示侧滑
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
        handleClick(row) {
            this.delBox(row)
       },
       delBox(id){
           console.log(id)
            this.$root.$emit("delBox",[id])
        },
       handleWindow(row){
           this.showWindow([row])
       },
       showWindow(id) {
            this.$root.$emit("showWindow",id);
        },
    },
    beforeDestroy(){
        this.$root.$off('search');
    }

}
</script>
