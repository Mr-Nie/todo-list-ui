<template>
  <div class="home">
  	<el-tabs v-model="status" type="card" @tab-click="handleClick">
	    <el-tab-pane label="全部" name="0"></el-tab-pane>
	    <el-tab-pane label="待办" name="1"></el-tab-pane>
	    <el-tab-pane label="已完成" name="2"></el-tab-pane>
	    <el-tab-pane label="已删除" name="3"></el-tab-pane>
	</el-tabs>
    <el-button class="add-btn" type="primary" @click="addFn">新增任务</el-button>
    <el-table :data="tableData" style="width:100%" :stripe="true">
	    <el-table-column
	      prop="deadline"
	      label="日期"
	      width="150">
	      <template slot-scope="scope">
	      	{{scope.row.deadline.substr(0,10)}}
	      </template>
	    </el-table-column>
	    <el-table-column
	      prop="name"
	      label="名称"
	      width="120">
	    </el-table-column>
	    <el-table-column
	      prop="content"
	      label="内容">
	    </el-table-column>
	    <el-table-column
	      label="状态"
	      width="120">
	      <template slot-scope="scope">
	      	<span v-if="scope.row.status==1">待办</span>
	      	<span v-if="scope.row.status==2">已完成</span>
	      	<span v-if="scope.row.status==3">已删除</span>
	      </template>
	    </el-table-column>
	    <el-table-column
	      label="操作"
	      width="120">
	      <template slot-scope="scope">
	        <el-button
	          @click.native.prevent="editFn(scope.row)"
	          type="text"
	          size="small">
	          编辑
	        </el-button>
	        <el-button v-if="scope.row.status==1"
	          @click.native.prevent="deleteFn(scope.row.id,2)"
	          type="text"
	          size="small">
	          完成
	        </el-button>
	        <el-button v-if="scope.row.status==2"
	          @click.native.prevent="deleteFn(scope.row.id,1)"
	          type="text"
	          size="small">
	          待办
	        </el-button>
	        <el-button  v-if="scope.row.status!=3"
	          @click.native.prevent="deleteFn(scope.row.id,3)"
	          type="text"
	          size="small">
	          移除
	        </el-button>
	      </template>
	    </el-table-column>
	</el-table>
    <el-dialog :title="isEdit?'编辑任务':'添加任务'" :visible.sync="dialogFormVisible" :show-close="false" center>
	  <el-form ref="todoForm" :model="todoForm">
	    <el-form-item label="名称" prop="name" :label-width="formLabelWidth">
	      <el-input v-model.trim="todoForm.name" autocomplete="off"></el-input>
	    </el-form-item>
	    <el-form-item label="截止日期" prop="deadline">
	      <el-date-picker type="date" placeholder="选择日期" v-model="todoForm.deadline" value-format="yyyy-MM-dd" style="width: 100%;"></el-date-picker>
	    </el-form-item>
	    <el-form-item label="备注" prop="content">
	      <el-input type="textarea" v-model.trim="todoForm.content"></el-input>
	    </el-form-item>
	  </el-form>
	  <div slot="footer" class="dialog-footer">
	    <el-button @click="dialogFormVisible = false">取 消</el-button>
	    <el-button type="primary" @click="editTodoFn">确 定</el-button>
	  </div>
	</el-dialog>
	<el-pagination
	  background :hide-on-single-page="true" :page-size="limit"
	  layout="prev, pager, next" @current-change="changePage"
	  :total="total">
	</el-pagination>
  </div>
</template>

<script>
export default {
  name: 'home',
  data() {
    return {
      tableData: [],//列表数据
      dialogFormVisible: false,//是否显示弹框
      isEdit:false,
      formLabelWidth: "100",
      total:0,//列表总条数
      status:0,//状态标签，全部-0，待办-1，完成-2，删除-3
      page:1,//当前页码
      limit:8,//每页数据条数
      todoForm:{
      	name:'',
      	deadline:'',
      	content:''
      }
    }
  },
  mounted:function(){
  	this.getList();
  },
  methods:{
  	// 分页
  	changePage(e){
		this.page = e;
		this.getList();
  	},
  	// 切换状态
  	handleClick(tab){
  		this.status = tab.name;
      	this.page = 1;
  		this.getList();
  	},
  	// 编辑打开弹框
  	editFn(todo){
  		this.isEdit = true;
  		this.todoForm.id = todo.id;
  		this.todoForm.name = todo.name;
  		this.todoForm.deadline = todo.deadline;
  		this.todoForm.content = todo.content;
  		this.dialogFormVisible = true;
  	},
  	// 新增打开弹框
  	addFn(){
  		this.todoForm.name = '';
  		this.todoForm.deadline = '';
  		this.todoForm.content = '';
		this.dialogFormVisible = true;
		this.isEdit=false;
  	},
  	// 获取列表数据
  	getList(){
	    var that = this;
	    var status = that.status;
	    var page = that.page;
	    var limit = that.limit;
	    this.$http.get('/api/list?status='+status+'&limit='+limit+'&page='+page).then(res=>{
	    	let list = res.data.list
	        that.tableData = list.rows
	        that.total = list.count
	    })
  	},
  	// 新增或编辑任务
    editTodoFn(){
    	var that = this;
    	var postData = this.todoForm;
    	var url = '/api/update';
    	if(!that.isEdit){
    		url = '/api/create';
    		delete postData.id;
    	}
        this.$http.post(url,postData).then(res=>{
      	  that.page = 1;
          that.getList();
    	  that.dialogFormVisible = false;
        })
    },
    // 删除或更改任务状态
    deleteFn(id,status){
      var that = this;
      this.$http.post('/api/status',{id:id,status:status}).then(res=>{
      	that.page = 1;
        that.getList();
      })
    }
  }
}
</script>
<style>
.home{
	padding-top:10px;
}
.add-btn{
	position:fixed;
	right:10px;
	top:10px;
}
</style>