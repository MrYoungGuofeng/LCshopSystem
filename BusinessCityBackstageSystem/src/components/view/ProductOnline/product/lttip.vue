<template>
    <div class="grid-content">
        <div class="productDesignation">
            <h3 class="listName pull-left">{{listname}}
                <i class="icon-double-angle-right"></i>
            </h3>
            <el-button type="primary" :class="{'btn-search':true,'el-icon-arrow-down':searchtext=='展开搜索','el-icon-arrow-up':searchtext=='收起搜索'}"  size="mini" @click="switchsearch">{{searchtext}}</el-button>
            <ul class="emendation">
                <li>已选中<span class="nums">0</span>项</li>
                <li id="modificationBtn" class='other' @click="edit">
                    <i class='el-icon-edit-outline'></i> 编辑
                </li>
                <li class="other"  data-toggle="modal" data-target="#delModal" @click="delBox">
                    <i class='el-icon-delete'></i> 删除
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
export default {
    props:['name','showWindow'],
    data(){
        return{
            listname:'',
            canedit:true,
            dataInfo:'',
            searchtext:'展开搜索'
        }
    },
    created:function(){
        this.listname=this.name;
        this.$root.$on('showlttip',(data)=>{
            console.log(data)
            this.dataInfo = data.datas
            var dom=document.getElementsByClassName('emendation')[0];
            let dom_edit=document.getElementById('modificationBtn');
            document.getElementsByClassName('nums')[0].innerHTML=data.num;
            dom.style.left=data.show?'0px':'-500px';
            dom_edit.style.cursor=data.editcan?'':'not-allowed';
            this.canedit=data.editcan;
        });
        // this.$root.$on('search',()=>{
        //     this.searchtext='展开搜索';
        // });
    },
    methods:{
        edit(){
            if(this.canedit){
                this.$root.$emit('editdialog');
                this.$root.$emit("showWindow",this.dataInfo)
                
            }

        },
        delBox(){
            this.$root.$emit("delBox",this.dataInfo)
        },
        switchsearch(){
            // console.log(this.searchtext);
            let organtext=this.searchtext;
            let flag=organtext=='展开搜索';
            this.searchtext=organtext=='展开搜索'?'收起搜索':'展开搜索';
            this.$root.$emit('switch',flag);
        }
    }
}
</script>
<style scoped>
.productDesignation{
    height: 72px;
    position: relative;
    margin-left: 33px;
}
.productDesignation .btn-search{
    position: absolute;
    left: 115px;
    top:22px;
    background-color: #fff;
    color: #27a1f2;
    border:none;
    font-size: 13px;
}
.productDesignation>i {
	color: #3da4c3;
}
.productDesignation h3 {
	padding-top: 25px;
	font-size: 20px;
	color: #0D4156;
	padding-left: 20px;
}
.productDesignation h3:before {
	content: "";
	width: 4px;
	height: 22px;
	display: block;
	position: absolute;
	background: #253a4d;
	left: 0px;
	top:26px;
}
.productDesignation p {
	margin: 0;
	color: #7e8e9f;
	font-size: 12px;
	letter-spacing: 2px;
	margin-top: 0.5rem;
}
.userincrease {
	padding-top: 10px;
}

.emendation {
	margin-top: 26px;
	position: absolute;
	top: 0;
	left:-500px;
	height: 32px;
    background: #fff;
    color: #555;
    transition: all 1s;
    -moz-transition: all 1s;
    -webkit-transition: all 1s;
    -o-transition:all 1s;
}

.emendation li {
	float: left;
	font-size: 16px;
	width: 110px;
	padding: 0 15px;
	position: relative;
	cursor: pointer;
    text-align: center
}
.other{
    border-left: 1px solid
}
.emendation li:nth-child(1){
	width:auto;
}
.emendation .nums {
	padding: 0 10px;
	color: #70b2c5;
}
</style>

