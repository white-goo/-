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
          prop="date"
          label="开课日期"
          :formatter="dateForm"
          width="180">
        </el-table-column>
        <el-table-column
          sortable
          prop="form"
          label="上课形式"
          width="180">
        </el-table-column>
        <el-table-column
          sortable
          prop="teacher.name"
          width="180"
          label="教师名称">
        </el-table-column>
        <el-table-column
          sortable
          prop="place"
          label="上课地点">
        </el-table-column>
        <el-table-column
          sortable
          prop="courseName"
          label="课程名称">
        </el-table-column>
        <el-table-column
          align="right">
          <template slot="header" slot-scope="scope">
            <el-input
              v-model="search"
              size="mini"
              @change="select"
              placeholder="输入课程名称搜索">
              <i slot="prefix" class="el-input__icon el-icon-search"></i>
            </el-input>
          </template>
          <template slot-scope="scope">
            <el-button
              size="mini"
              slot="reference"
              type="danger"
              @click.stop="handleDelete(scope.$index, scope.row)"
            >Delete
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div>
      <el-dialog title="课程详情" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="课程名称" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.courseName" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="上课地点" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.place" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="上课时间" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.date" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="教师名称" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.teacher.name" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="上课形式" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.form" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
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
        <el-button class="refresh-button" icon="el-icon-refresh" title="刷新" @click="refresh" />
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
          courseName: '',
          place: '',
          form: '',
          date: '',
          teacher: {
            name: ''
          },
          date1: '',
          date2: '',
          delivery: false,
          type: [],
          resource: '',
          desc: '',
          disabled: true
        },
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
          this.$http.post(this.url + "/course/delete", {
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
        this.$http.post(this.url + "/course/list", {
          current: this.currentPage,
          total: this.total,
          size: this.pageSize,
          conditions
        }).then(data => {
          console.log(data);
          data.data.courseList.map((item, index) => {
            item.form = item.form === "ONLINE" ? "网课" : "线下";
          })
          this.tableData = data.data.courseList;
          this.loading = false;
        })
      },

      rowClick(row, column, event) {
        console.log(row, column, event);
        this.$http.post(this.url + "/course/load", {
          id: row.id
        }).then(date => {
          this.form.date = date.data.course.date;
          this.form.form = date.data.course.form;
          this.form.teacher = date.data.course.teacher;
          this.form.courseName = date.data.course.courseName;
          this.form.place = date.data.course.place;
          this.dialogFormVisible = true
        })
      },

      select(string) {
        this.list({"course_name": string});
      },
      dateForm(row, column) {
        return formatDate(row[column.property], "yyyy-MM-dd");
      },
      refresh(event){
        let target = event.target;
        if(target.nodeName === "I" || target.nodeName === "SPAN"){
          target = event.target.parentNode;
        }
        target.blur();
        this.list();
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
