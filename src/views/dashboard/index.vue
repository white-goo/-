<template>
  <div>
    <div>
      <el-table
        :data="tableData"
        stripe
        row-key="id"
        default-sort = "{prop: 'date', order: 'ascending'}"
        @row-click="this.rowClick"
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
              placeholder="输入课程名称搜索"/>
          </template>
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.$index, scope.row)">Edit</el-button>
            <el-button
              size="mini"
              type="danger"
              @click="handleDelete(scope.$index, scope.row)">Delete</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[1,5,10,20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="tableData.length">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import {mapGetters} from 'vuex'

  export default {
    name: 'Dashboard',
    data() {
      return {
        tableData: [],
        search: '',
        currentPage: 1, // 当前页码
        total: 20, // 总条数
        pageSize: 10, // 每页的数据条数
        url: process.env.VUE_APP_BASE_API
      };
    },
    created() {
      this.list(this.currentPage, this.total, this.pageSize);
    },
    computed: {
      ...mapGetters([
        'name'
      ])
    },
    methods: {
      handleEdit(index, row) {
        console.log(index, row);
      },
      handleDelete(index, row) {
        console.log(index, row);
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
      list(current,total,size){
        this.$http.post(this.url + "/course/list",{
          "current": current,
          "total": total,
          "size": size,
        }).then(data=>{
          console.log(data);
          this.tableData = data.data.courseList;
        })
      },

      rowClick(row, column, event){
        console.log(row.id);
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
