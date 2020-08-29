<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 基础表格
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
        <div class="handle-box">
            <el-button
                type="primary"
                icon="el-icon-delete"
                class="handle-del mr10"
                @click="delAllSelection"
            >批量删除</el-button>
            <el-select v-model="query.address" placeholder="地址" class="handle-select mr10">
                <el-option key="1" label="广东省" value="广东省"></el-option>
                <el-option key="2" label="湖南省" value="湖南省"></el-option>
            </el-select>
            <el-input v-model="query.name" placeholder="用户名" class="handle-input mr10"></el-input>
            <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            <el-button @click="handleInsert" style="float: right;margin-right:13px;" type="primary" icon="el-icon-search">新增员工</el-button>
        </div>
        <el-table
            :data="accs"
            border
            stripe
            class="table"
            ref="multipleTable"
            header-cell-class-name="table-header"
            @selection-change="handleSelectionChange"
        >
            <el-table-column type="selection" width="55" align="center"></el-table-column>
            <el-table-column fixed prop="id" label="ID" width="55" align="center"></el-table-column>
            <el-table-column fixed prop="empDetails.jobNum" label="工号" width="100" align="center"></el-table-column>
            <el-table-column fixed prop="name" label="姓名" width="100" align="center"></el-table-column>
            <el-table-column fixed prop="account.number" label="账户号码" width="100" align="center"></el-table-column>
            <el-table-column fixed prop="account.money" label="账户余额" width="100" align="center"></el-table-column>
            <el-table-column fixed prop="account.createTime" label="开户日期" width="100" align="center"></el-table-column>
            <el-table-column label="账户状态" width="100" align="center">
                <template slot-scope="scope">
                    <template v-if="scope.row.account">
                      <el-tag
                        :type="scope.row.account.state===true?'success':(scope.row.account.state===false?'danger':'')"
                      >
                      <span v-if="scope.row.account.state">启用</span>
                      <span v-else>冻结</span>
                      </el-tag>
                    </template>
                    <template v-else>
                      <el-tag :type="'danger'">
                      <span>暂无</span>
                      </el-tag>
                    </template>
                </template>
            </el-table-column>
            <el-table-column label="用户状态" width="100" align="center">
                <template slot-scope="scope">
                  <el-tag
                    :type="scope.row.enable===true?'success':(scope.row.enable===false?'danger':'')"
                  >
                  <span v-if="scope.row.enable">启用</span>
                  <span v-else>停用</span>
                  </el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="empDetails.hireDate" label="入职时间" width="100" align="center"></el-table-column>
            <el-table-column fixed="right" label="操作" width="180" align="center">
                <template slot-scope="scope">
                  <el-button
                    type="text"
                    icon="el-icon-edit"
                    @click="handleAddAcc(scope.$index, scope.row)"
                  >开户</el-button>
                  <el-button
                    style="margin-right:10px;color:red;"
                    type="text"
                    icon="el-icon-edit"
                    @click="handleDelAcc(scope.$index, scope.row)"
                  >销户</el-button>
                  <el-dropdown >
                    <span style="font-size:10px;" class="el-dropdown-link">
                      状态<i class="el-icon-arrow-down el-icon--right"></i>
                    </span>
                  <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item>
                      <el-button
                        type="text"
                        icon="el-icon-delete"
                        class="green"
                        @click="handlEnable(scope.$index, scope.row)"
                      >启用</el-button>
                    </el-dropdown-item>
                    <el-dropdown-item>
                      <el-button
                        type="text"
                        icon="el-icon-delete"
                        class="red"
                        @click="handleDisable(scope.$index, scope.row)"
                      >禁用</el-button>
                    </el-dropdown-item>
                  </el-dropdown-menu>
                </el-dropdown>  
                </template>
            </el-table-column>
        </el-table>
      <div class="pagination">
        <el-pagination
          background
          layout="total, prev, pager, next"
          :current-page="query.pageIndex"
          :page-size="query.pageSize"
          :total="pageTotal"
          @current-change="handlePageChange"
        ></el-pagination>
      </div>
      <!-- 开户弹出框 -->
      <el-dialog title="开户" :visible.sync="addVisible" width="30%">
        <el-form ref="addform" :model="addform" label-width="70px"
          style="text-align-last:justify;text-align:justify;">
          <el-form-item label="账 号">
            <el-input v-model="addform.username"></el-input>
          </el-form-item>
          <el-form-item label="账号状态"> 
            <el-switch
              v-model="addform.enable"
              active-text="启用"
              inactive-text="禁用"
            >
            </el-switch>
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="addform.remark"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addVisible = false">取 消</el-button>
          <el-button type="primary" @click="saveInsert">确 定</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>
<script>
export default {
  name: 'accInfo',
  data() {
    return {
      accs: [],
      hr: {},
      query: {
        address: '',
        name: '',
        pageIndex: 1,
        pageSize: 10
      },
      tableData: [],
      multipleSelection: [],
      delList: [],
      pageTotal: 0,
      addVisible: false,
      addform: {
        username: '',
        enable: true,
        remark: ''
      },
      idx: -1,
      id: -1
    };
  },
  created() {
    this.getData()
  },
  methods: {
    getData() {
      this.getRequest('/acc/info/page').then(resp => {
        this.accs = resp.data;
        this.pageTotal = resp.total
      })
    },
    // 触发搜索按钮
    handleSearch() {
      this.$set(this.query, 'pageIndex', 1);
      this.getData();
    },
    // 禁用操作
    handleDisable(index, row) {
      if (row.account == null || row.account.id == null) {
        this.$message.error("暂无账号，请先开户!")
        return
      }
      // 二次确认删除
      this.$confirm('确定要禁用吗？', '提示', {
        type: 'warning'
      })
      .then(() => {
        this.getRequest('/acc/info/disableAcc/'+row.account.id).then(resp => {
          this.getData()
          this.$message.success(e.msg)
        }).catch(()=> {})
      })
      .catch(() => {});
    },
    handlEnable(index,row){
      if (row.account == null || row.account.id == null) {
        this.$message.error("暂无账号，请先开户!")
        return
      }
      // 二次确认启用
      this.$confirm('确定要启用吗？', '提示', {
        type: 'warning'
      })
      .then(() => {
        this.getRequest('/acc/info/enableAcc/'+row.account.id).then(resp => {
          this.getData()
          this.$message.success(e.msg)
        }).catch(()=> {})
      })
      .catch(() => {});
    },
    // 多选操作
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    delAllSelection() {
      const length = this.multipleSelection.length;
      let str = '';
      this.delList = this.delList.concat(this.multipleSelection);
      for (let i = 0; i < length; i++) {
        str += this.multipleSelection[i].name + ' ';
      }
      this.$message.error(`删除了${str}`);
      this.multipleSelection = [];
    },
    // 编辑操作
    handleEdit(index, row) {
      this.idx = index;
      this.editform = row;
      this.editVisible = true;
    },
    handleAddAcc(index, row) {
      // 二次确认开户
      this.$confirm('确定要开户吗？', '提示', {
        type: 'warning'
      })
      .then(() => {
        this.getRequest('/acc/info/addAccount/'+row.id).then(resp => {
          this.getData()
          this.$message.success(e.msg)
        }).catch(()=> {
            
        })
      })
      .catch(() => {});
    },
    handleDelAcc(index, row){
      // 二次确认销户
      this.$confirm('确定要销户吗？', '提示', {
        type: 'warning'
      })
      .then(() => {
        this.getRequest('/acc/info/delAcc/'+row.id).then(resp => {
          this.getData()
          this.$message.success(e.msg)
        }).catch(()=> {})
      })
      .catch(() => {});
    },
    // 保存编辑
    saveEdit() {
      this.editVisible = false;
      this.putRequest('/emps/updateBaseInfo',this.editform).then(resp => {
        this.getData()
      })
    },
    //保存新增
    saveInsert() {            
      this.addVisible = false;
      this.postRequest('/emps/addBaseInfo',this.addform).then(resp => {
        this.getData()
      })
    },
    // 分页导航
    handlePageChange(val) {
      this.$set(this.query, 'pageIndex', val);
      this.getData();
    },
    handleInsert() {
      this.addVisible = true;
    }
  }
}
</script>
<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}
.handle-input {
  width: 300px;
  display: inline-block;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.green {
  color: green;
}
.mr10 {
  margin-right: 10px;
}
.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>
