<template>
  <div class="views-container">
      <div class="views-search">
          <span> 别名：
            <el-input v-model="runUrl" placeholder="请输入别名"></el-input>
          </span>
          <span> 名称：
            <el-input v-model="runContent" placeholder="请输入名称"></el-input>
          </span>
          <el-button @click="fetchDatesourcetemp()" type="primary">查询</el-button>
          <el-button @click="resetRUNtemp()">重置</el-button>
      </div>
      <div class="views-content">
          <h2>
              <el-button @click="addSysAttribuManagemDialog()">新增</el-button>
              <div>
                  <a href="#"></a>
              </div>
          </h2>
          <el-table
            :data="scriptlistData"
            ref="multipleTable"
            @selection-change="handleSelectionChange"
            @row-click="fn_rowClick"
            style="width: 100%"
            border>
            <el-table-column type="selection" width="40"></el-table-column>
            <el-table-column align="center" label="别名">
              <template slot-scope="scope">
                  {{scope.row.key}}
              </template>
            </el-table-column>
            <el-table-column align="center" label="名称">
              <template slot-scope="scope">
                  {{scope.row.name}}
              </template>
            </el-table-column>
            <el-table-column align="center" label="数据库" width="300">
              <template slot-scope="scope">
                  {{scope.row.dbType}}
              </template>
            </el-table-column>
            <el-table-column label="管理" align="center" width="200">
              <template slot-scope="scope">
                  <el-button type="text" @click="editScriptDialog(scope.row)">编辑</el-button>
                  <el-button type="text" @click="delDatasourceInfo(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
      </div>
      <div class="pagesNum">
        <!-- <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          @prev-click="handleCurrentChange"
          @next-click="handleCurrentChange"
          :current-page.sync="page.currentPage"
          :page-size="page.pageSize"
          layout="total, prev, pager, next"
          :total="page.total">
        </el-pagination> -->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          @prev-click="handleCurrentChange"
          @next-click="handleCurrentChange"
          :current-page.sync="page.currentPage"
          :page-sizes="page.pageSizes"
          :page-size="page.pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="page.total">
        </el-pagination>
      </div>

      <!--弹出框-新增编辑-->
      <newSysDatasources-dialog
          v-bind:visible.sync="dialog_show.newSysDatasourcesDialog"
          @closed="closeDialog"
          :edit_data="to_edit_data"
          :title="add_OR_editor">
      </newSysDatasources-dialog>

  </div>
</template>

<script>
import {
  getSysDatasourceListData,  //获取列表数据
  geDatasourcefullInfoById,  //根据ID返回该脚本的完整信息
  removeDatasource,  //根据ID删除该行信息
} from "@/api/api.js";

import newSysDatasourcesDialog from "./dialogs/new-sysDatasources-dialog"; // 新增编辑
// import { param } from '../../utils';
export default {
  name: "commonscript",
  components: {
    newSysDatasourcesDialog,
  },
  data() {
    return {
      runUrl: "",
      runContent:"",
      environment:"",
      environments:[{
        value: '选项1',
        label: '开发-默认'
      }, {
        value: '选项2',
        label: '测试'
      }, {
        value: '选项3',
        label: '用户测试'
      }, {
        value: '选项4',
        label: '灰度'
      }, {
        value: '选项5',
        label: '生产'
      }],
      scriptlistData:[],
      to_edit_data:{},
      add_OR_editor: "",
      dialog_show: {
        newSysDatasourcesDialog: false,
      },
      temp: 1,
      page: {
        currentPage: 1,
        pageSizes: [10, 25, 50, 100],
        pageSize: 10,
        total: 0
      },

    }
  },
  computed: {},
  methods: {
    handleSelectionChange(val) {
      //val是选中选的数组集合，包含每一个选中项的数据
      this.multipleSelection = val; //选择项发生变化时，选中数组变量跟着改变
    },
    //表格单选
    fn_rowClick(row, column, event) {
      //当某一行被点击时，改行多选框被选中
      this.$refs.multipleTable.toggleRowSelection(row);
    },

    /*查询列表信息*/
    fetchDatesourcetemp(pageFlag){
      /* 初始化当前页 */
      let currPageBefore = this.page.currentPage;
      /* 判断是否是分页  如果当前页是第二页  查询的时候从第一页开始 */
      if(pageFlag!=='page'){
        this.page.currentPage = 1;
      }
      let param = {
        offset: (this.temp-1)*this.page.pageSize,
        limit: this.page.pageSize,
        url: this.runUrl,
        status: this.abnormalstate,
        content: this.runContent
      };
      getSysDatasourceListData(param)
        .then(response => {
          this.scriptlistData = response.rows;
          this.page.total = parseInt(response.total);
        })
        .catch(error => {
          console.error(error);
          this.page.currentPage =currPageBefore;
        });
    },
    /*重置*/
    resetRUNtemp(){
      this.runUrl ="";
      this.sortScript ="";
      this.page.currentPage=1;
      this.fetchDatesourcetemp();

    },
    /*新增*/
    addSysAttribuManagemDialog(){
      this.add_OR_editor = "新增";
      this.dialog_show.newSysDatasourcesDialog = true;
    },

    /*编辑*/
    editScriptDialog(row) {
      this.add_OR_editor = "编辑";
      let param = {
        id: row.id
      };
      geDatasourcefullInfoById(param)
        .then(response => {
          this.to_edit_data = response.data;
          this.dialog_show.newSysDatasourcesDialog = true;
          this.$refs.multipleTable.clearSelection();
      })
    },

    /*删除*/
    delDatasourceInfo(row) {
      this.$confirm('确定要删除, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }).then(() => {
      let param = {
        id: row.id
      };
      removeDatasource(param)
        .then(response => {
          if (response.isOk === true) {
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
            this.fetchDatesourcetemp();
          } else {
            this.$message.error("删除失败");
          }
        })
        .catch(error => {
          console.error(error);
        });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          });
        });
    },

    /*关闭弹窗操作*/
    closeDialog(){
      this.fetchDatesourcetemp();
    },

    //分页
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.temp = val;
      this.fetchDatesourcetemp("page");
      // alert('现在是第'+val+'页');
      // this.currentDBRow = "-1";
    },
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`);
      this.page.pageSize = val;
      this.fetchDatesourcetemp("page");
      // alert('现在是每页'+val+'条');
    },

  },
  // mounted--钩子函数，一来到该页面即执行
  mounted: function() {
    this.fetchDatesourcetemp();
  }
};
</script>

<style lang="scss" scoped>
.views-container {
  padding: 10px;
}
.views-search{
  width: auto;
  padding: 6px 0;
  clear: both;
  margin-bottom: 10px;
}
.views-search span{
  padding: 0px 10px;
}
.views-search span /deep/ .el-input{
  width: 30%;
}
.views-search span /deep/ .el-select .el-input{
  width: 90%;
}
.views-content{
  height: 96%;
}
.pagesNum {
  height: 50px;
  margin-top: 15px;
  text-align: right;
}
</style>