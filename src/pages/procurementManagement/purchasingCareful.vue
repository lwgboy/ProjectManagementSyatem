<template>
<el-row class='PendingOrder myContainer'>
  <el-col :span='24' class="title">
    <div class="procurment_title">已处理订单</div>
  </el-col>
      <el-col :span='24' class="PendingOrderheader">
       <el-form :inline="true"  class="demo-form-inline">
	       	 <el-form-item label="所有项目">
				 <el-select v-model="selectvalue" placeholder="请选择">
				    <el-option
				      v-for="item in options"
				      :key="item.value"
				      :label="item.label"
				      :value="item.value"
				      prop='selectvalue'>
				    </el-option>
				  </el-select>
	         </el-form-item>
	        <el-form-item label="订单名称">
			    <el-input v-model="condition" ></el-input>
			 </el-form-item>
			<el-form-item label="开始时间">
			        <el-date-picker
				       v-model="staDate"
				       type="date"
				       placeholder="选择日期"
				       value-format="yyyy-MM-dd">
				    </el-date-picker>
			 </el-form-item>
			 <el-form-item label="结束时间">
			         <el-date-picker
				       v-model="endDate"
				       type="date"
				       placeholder="选择日期"
				       value-format="yyyy-MM-dd">
				    </el-date-picker>
			 </el-form-item>
			  <el-button  size="small" type="primary" @click='GetUntreated'>查询</el-button>
        </el-form>  
       </el-col>
   <el-col :span='24' class="PendingOrdermain">
         <el-table
		    :data="tablepending"
		    border
		    stripe
		    style="width: 100%">
		    <el-table-column
		      prop="PurchaseCode"
		      label="编号"
		      width="180">
		    </el-table-column>
		    <el-table-column
		      prop="PurchaseTitle"
		      label="订单标题"
		      width="180">
		    </el-table-column>
            <el-table-column
		      prop="LoginName"
		      label="申请人"
		      width="180">
		    </el-table-column>
		     <el-table-column
		      prop="ProjectCode"
		      label="所属项目"
		      width="180">
		    </el-table-column>
		     <el-table-column
		      prop="ExpirationDate"
		      label="交货时间"
		      width="180">
		    </el-table-column>
		    <el-table-column
		      label="供货详说明">
			  <template slot-scope="scope">
		          <a  @click="Link(scope.$index)"  class="underline">
						  {{tablepending[scope.$index].FileName}}	 
				  </a>
		      </template>
		    </el-table-column>
		    <el-table-column
		      label="签收单">
			  <template slot-scope="scope">
		          <a  @click="SignLink(scope.$index)"  class="underline">
						  {{tablepending[scope.$index].SignInFileName}}	 
				  </a>
		      </template>
		    </el-table-column>
		    <el-table-column
		      prop="name"
		      label="操作"
		      width="180">
		 <template slot-scope="scope">
	       <el-button type="primary" size="small" @click='Editingpermissions(scope.$index)'>详情</el-button>
	      </template>
		    </el-table-column>
	    </el-table>
         <div class="myPagination"><!-- 组件翻页 -->
	        <el-pagination
	          background
	          layout="prev, pager, next"
	          :total="totalNumber" 
	          :page-size='pageSize'
	          @current-change='pageIndexChange'
	          >
	        </el-pagination>
	      </div>

   </el-col>
    
</el-row>

</template>
<script>
	import{ProjectManage,ChaseProcessed}from'@/api/api'//引进api
export default {
       data(){
          return{
          	  stripe:true,
              options:[],//请求过来项目列表!
           	  selectvalue:'',//下拉框选择的值
           	  startvalue:'',//日期开始的值,
           	  endvalue:'',  //日期结束的值,
              projectCode:'',
              condition:'',
              staDate:'',
              endDate:'',
              pageIndex:1,
              pageSize:10,
              totalNumber:null,
              tablepending: [],
              FileUrl:'',
	       }
	     },
       methods:{
	      getprojectmange(){
	          var parms={//传的参数,项目下拉框!
	               pageIndex: 10000,
	               pageSize:10000
	              }
	           ProjectManage(parms).then( res => {//项目列表
	              let options=[]; 
	              options=this.options
	              for(let i=0;i<res[0].TotalNumber;i++){   //遍历出来的数组放进去
	                options.push({
	                   value:res[0].DataList[i].ProjectCode,
		               label:res[0].DataList[i].ProjectName
	                  })   
	                }
	              })
	           },
	           GetUntreated(ev){
	           	var _this=this;
                 var parms={
                     projectCode:this.selectvalue,
                     condition:this.condition,
                     staDate:this.staDate,
                     endDate:this.endDate,
                     pageIndex:this.pageIndex,
                     pageSize:this.pageSize
                 }
	           	 ChaseProcessed(parms).then(res=>{
	           	 	this.tablepending=[];
	           	 	this.totalNumber=res[0].TotalNumber
	           	 	console.log(this.totalNumber) 
	           	 	for(let item of res[0].DataList){
	           	 		this.tablepending.push({
                           PurchaseCode:item.PurchaseCode, 
                           PurchaseTitle:item.PurchaseTitle,
                           LoginName:item.LoginName,
                           ProjectCode:item.ProjectCode,
                           ExpirationDate:item.ExpirationDate.replace("0:00:00",""),
                           FileName:item.FileName,
                           FileUrl:item.FileUrl,
                           SignInFileName:item.SignInFileName,
                           SignInFileUrl:item.SignInFileUrl

	           	 		})
	           	 	}
	           	 /*	console.log(this.tablepending)*/
	           	 })
	           },
             Editingpermissions(index) {//点击详情
					this.$router.push({name:'Details', params: {id:this.tablepending[index].PurchaseCode}})//传编号给路由
				},
	           Link(index){//点击下载时候下载相应的文档!
                        window.open(this.tablepending[index].FileUrl)//链接绑定
	           },
               SignLink(index){
                     window.open(this.tablepending[index].SignInFileUrl) 
               },
	           pageIndexChange(pageIndex){//翻页监控当前页面发生变化没有! 重新获取列表的页面!~
                 this.pageIndex = pageIndex;//传当前页面     
                 this. GetUntreated()//重新获取一边当前的
               }   
       },
       mounted(ev){
       	   var  _this=this
       	     this.getprojectmange();//调用方法.项目名称显示!
       	     this.GetUntreated();//得到待处理订单的列表!
               $(window).keyup(function(ev){//enetr键快捷搜
			        // console.log(ev);
			         if(ev.keyCode == 13){
			           _this.GetUntreated();
			        }
			      })
       }
}
</script>

<style scoped lang="scss">
 .PendingOrder{ 
   .PendingOrdermain{
	    margin:0px 20px 20px 20px;
		background: #fff;
	    border: 1px solid #ccc; 
	    width: calc(100% - 40px);
	    height: calc(100% - 90px);
	    text-align: center;
	    }

	    .PendingOrderheader{
	    	margin: 20px 20px 0px 20px;
	        /*min-width: 1474px;*/
	    }
	    .underline{
	    	text-decoration: underline;
	    	color: blue;
	    	cursor:Pointer;
	    }
	    .el-pagination{
	    	text-align: center;
	    }
    }
</style>