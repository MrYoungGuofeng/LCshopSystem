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
        <!--<el-table-column class='borderRight' fixed prop="id" label="ID" width='360' height='100'>
        </el-table-column>!-->
        <el-table-column
        prop="departmentName"
        label="部门"
        >
        </el-table-column>
        <el-table-column
        prop="object.adminName"
        label="姓名">
        </el-table-column>
        <el-table-column
        prop="content"
        label="敏感操作">
        </el-table-column>
        <el-table-column
        prop="model"
        label="操作详情">
        </el-table-column>
        <el-table-column
        prop="createTime"
        label="操作时间">
        </el-table-column>
        <el-table-column
        prop="ip"
        label="员工IP">
        </el-table-column>  
    </el-table>
</template>
<script>
/ eslint-disable /
//@row-click="showMemberInfo()"
export default {
    // prop:['listLoading'],
    data(){
        return {
            datalist:[],
            showLeft:0,
            pageIndex:1,
            listLoading:false,
            data:{
                userId:null,
                content:null,
                beginTime:null,
                endTime:null,
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
            this.data.userId=datas.log.userId===''?null:datas.log.userId;
            this.data.content=datas.log.content===''?null:datas.log.content;
            this.data.beginTime=datas.log.daterange?datas.log.daterange[0]+' 0:00:00':null;
            this.data.endTime=datas.log.daterange?datas.log.daterange[1]+' 0:00:00':null;
            // console.log(this.data);
            this.getDate(1)
        });
    },
    methods:{
      getDate(pageIndex,data) {
            this.listLoading =  true;
            let url = '/api/public/logger/findLog?pageNum='+pageIndex+'&pageSize=10';
            this.$http({
                url: url,
                method: 'POST',
                // 请求体重发送的数据
                headers: { 'Content-Type': 'application/json' },
                data:this.data,
            })
            .then(response => {
                this.listLoading =  false;
                console.log(response)
                //for(let i = 0 ;i<response.data.info.list.length;i++){
                            this.datalist = response.data.info.list
                //}
                console.log( this.datalist)
                this.$root.$emit('pages',response.data.info.pages)
                this.$root.$emit('total',response.data.info.total)
          })
          .catch(error=>{
              this.listLoading =  false;
              console.log(error);
              //         alert('网络错误，不能访问');
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
            // this.$root.$emit('showlttip',{show,editcan,num:this.multipleSelection.length,datas:this.multipleSelection});
        },
        indexMethod(index) {
            return index + 1
        },
    },
    beforeDestry(){
        this.$root.$off('search');
    }

}
</script>
