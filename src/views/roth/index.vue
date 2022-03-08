<template>
  <div>
    <div>
      <el-table
        :data="tableData"
        stripe
        v-loading="loading"
        row-key="id"
        :default-sort="{prop: 'date', order: 'ascending'}"
        @row-click="rowClick"
        style="width: 100%">
        <el-pagination
          background
          layout="prev, pager, next"
          :total="1000">
        </el-pagination>
        <el-table-column
          sortable
          prop="name"
          label="用户名">
        </el-table-column>
        <el-table-column
          sortable
          prop="createTime"
          :formatter="dateForm"
          label="创建时间">
        </el-table-column>
        <el-table-column
          sortable
          prop="editTime"
          :formatter="dateForm"
          label="修改时间">
        </el-table-column>
        <el-table-column
          align="right">
          <template slot="header" slot-scope="scope">
            <el-input
              v-model="search"
              size="mini"
              @change="select"
              placeholder="输入账号名称搜索">
              <i slot="prefix" class="el-input__icon el-icon-search"></i>
            </el-input>
          </template>
          <template slot-scope="scope">
            <el-button
              size="mini"
              slot="reference"
              type="danger"
              @click.stop="handleDelete(scope.$index, scope.row)"
            >删除
            </el-button>
          </template>
        </el-table-column>
        <el-table-column
          :render-header="renderHeader"
          width="120"
        >
          <template slot="operation" slot-scope="{col}">
            <el-table-column
              :key="col.index"
              :label="col.name"
              :prop="col.index"
              :width="col.width">
            </el-table-column>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div>
      <el-dialog title="用户详情" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="登录账号" :label-width="form.username">
            <el-input :disabled="this.disabled" v-model="form.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="用户名" :label-width="form.name">
            <el-input :disabled="this.disabled" v-model="form.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码" :label-width="form.password">
            <el-input placeholder="请输入密码" :disabled="this.disabled" v-model="form.password" show-password
                      autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button :style="{display:newDisplay}" type="primary" @click="dialogFormVisible = false">确 定
          </el-button>
          <el-button :style="{display:saveDisplay}" type="primary" @click="save">保 存</el-button>
          <el-button :style="{display:updateDisplay}" type="primary" @click="update">更新</el-button>
        </div>
      </el-dialog>
    </div>
    <div>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[1,5,10,20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper, slot"
        :total="tableData.length">
        <el-button class="refresh-button" icon="el-icon-refresh" title="刷新" @click="refresh"/>
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import {mapGetters} from 'vuex'
  import {formatDate} from 'element-ui/lib/utils/date-util';

  export default {
    name: 'Dashboard',
    data() {
      return {
        loading: true,
        tableData: [],
        search: '',
        currentPage: 1, // 当前页码
        total: 20, // 总条数
        pageSize: 10, // 每页的数据条数
        url: process.env.VUE_APP_BASE_API,
        dialogTableVisible: false,
        dialogFormVisible: false,
        form: {
          username: '',
          password: '',
          name: '',
          id: ''
        },
        date1: '',
        date2: '',
        delivery: false,
        type: [],
        resource: '',
        desc: '',
        disabled: true,
        options: [],
        authOptions: [],
        saveDisplay: 'none',
        updateDisplay: 'none',
        newDisplay: '',
        credit: 0,
        formLabelWidth: '120px'
      };
    },

    created() {
      this.list();
    },
    computed: {
      ...mapGetters([
        'name'
      ])
    },
    methods: {
      handleDelete(index, row) {
        console.log(index, row);
        this.$confirm("是否要删除该数据?", "提示", {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          customClass: 'del-model',
          type: 'warning'
        }).then(() => {
          this.$http.post(this.url + "/auth/user/delete", {
            id: row.id
          }).then(() => {
            this.list()
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
          })
        })
      },
      //每页条数改变时触发 选择一页显示多少行
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.currentPage = 1;
        this.pageSize = val;
      },
      //当前页改变时触发 跳转其他页
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.currentPage = val;
      },
      list(conditions) {
        console.log(this.currentPage, this.total, this.size, this.conditions)
        this.loading = true;
        this.$http.post(this.url + "/auth/user/list", {
          current: this.currentPage,
          total: this.total,
          size: this.pageSize,
          conditions
        }).then(data => {
          console.log(data);
          this.tableData = data.data.data;
          this.loading = false;
        })
      },

      rowClick(row, column, event) {
        console.log(row, column, event);
        this.saveDisplay = 'none';
        this.$http.post(this.url + "/auth/authCheck", [
          {
            key: "auth1",
            params: {
              id: row.id
            },
            path: "/auth/user/update",
            module: "auth"
          }
        ]).then(date => {
          if (date.data.data.auth1) {
            this.updateDisplay = '';
            this.newDisplay = 'none';
            this.disabled = false;
          } else {
            this.updateDisplay = 'none';
            this.newDisplay = '';
            this.disabled = true;
          }
        })

        this.$http.post(this.url + "/auth/user/load", {
          id: row.id
        }).then(date => {
          this.form = date.data.data;
          this.dialogFormVisible = true;
        })
      },

      select(string) {
        this.list({"username": string});
      },
      dateForm(row, column) {
        return formatDate(row[column.property], "yyyy-MM-dd");
      },
      refresh(event) {
        let target = event.target;
        if (target.nodeName === "I" || target.nodeName === "SPAN") {
          target = event.target.parentNode;
        }
        target.blur();
        this.list();
      },
      renderHeader() {
        return (
          <div>< el-button type = "primary" size = 'small' on-click = {()=>this.newRole()}> 新增 < /el-button></div>
      )
      },
      newRole() {
        this.form.id = '';
        this.form.name = '';
        this.form.username = '';
        this.saveDisplay = '';
        this.updateDisplay = 'none';
        this.newDisplay = 'none';
        this.disabled = false;
        this.dialogFormVisible = true;
      },
      save() {
        this.$http.post(this.url + "/auth/user/save", {
          username: this.form.username,
          password: this.form.password,
          name: this.form.name
        }).then(date => {
          if (date.data.code === 20000) {
            this.$message({
              message: '保存成功',
              type: 'success'
            });
            this.dialogFormVisible = false;
            this.list();
          }
        })
      },
      update() {
        this.$http.post(this.url + "/auth/user/update", this.form).then(date => {
          if (date.data.code === 20000) {
            this.$message({
              message: '保存成功',
              type: 'success'
            });
            this.dialogFormVisible = false;
            this.list();
          }
        })
      }
    }
  }
</script>

<style>
  #box ul {
    display: flex;
    flex-wrap: wrap;
  }

  #box li {
    padding: 3px;
    list-style: none;
    margin-right: 15px;
    border: 1px solid #eee;
  }

  #box img {
    width: 200px;
    height: 150px;
  }
</style>
