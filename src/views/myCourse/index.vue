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
            >删除
            </el-button>
          </template>
        </el-table-column>
        <el-table-column
          width="120"
        ></el-table-column>
      </el-table>
    </div>
    <div>
      <el-dialog title="课程详情" :visible.sync="dialogFormVisible">
        <el-form :model="form">
          <el-form-item label="课程名称" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.courseName" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="课程地点" :label-width="formLabelWidth">
            <el-input :disabled="form.disabled" v-model="form.place" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="开始时间" :label-width="formLabelWidth">
            <el-date-picker
              :disabled="form.disabled"
              v-model="form.date"
              type="date"
              placeholder="选择日期">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="结束时间" :label-width="formLabelWidth">
            <el-date-picker
              :disabled="form.disabled"
              v-model="form.endDate"
              type="date"
              placeholder="选择日期">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="教师名称" :label-width="formLabelWidth">
            <el-select :disabled="form.disabled" v-model="form.teacher.id" placeholder="请选择">
              <el-option
                v-for="item in form.options"
                :key="item.id"
                :label="item.name"
                :value="item.id">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="上课形式" :label-width="formLabelWidth">
            <el-radio :disabled="form.disabled" v-model="form.form" label="ONLINE">网课</el-radio>
            <el-radio :disabled="form.disabled" v-model="form.form" label="OFFLINE">线下</el-radio>
          </el-form-item>
          <el-form-item label="课程数量" :label-width="formLabelWidth">
            <el-input-number :disabled="form.disabled" :min="0" :max="10" v-model="form.courseAmount"></el-input-number>
          </el-form-item>
          <el-form-item label="学分" :label-width="formLabelWidth">
            <el-input-number :disabled="form.disabled" :min="0" :max="10" v-model="form.credit"></el-input-number>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
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
    name: 'Index',
    //import引入的组件需要注入到对象中才能使用",
    components: {},
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
          endDate: '',
          courseAmount: '',
          teacher: {
            id: '',
            name: ''
          },
          date1: '',
          date2: '',
          delivery: false,
          type: [],
          resource: '',
          desc: '',
          disabled: true,
          options: [],
          saveDisplay: 'none',
          newDisplay: '',
          credit: 0
        },
        formLabelWidth: '120px'
      };
    },
    //监听属性 类似于data概念",
    computed: {
      ...mapGetters([
        'name'
      ])
    },
    //监控data中的数据变化",
    watch: {},
    //方法集合",
    methods: {
      handleDelete(index, row) {
        console.log(index, row);
        this.$confirm("是否要删除该数据?", "提示", {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          customClass: 'del-model',
          type: 'warning'
        }).then(() => {
          this.$http.post(this.url + "/course/deleteMyCourse", {
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
        this.$http.post(this.url + "/course/listMyCourse", {
          current: this.currentPage,
          total: this.total,
          size: this.pageSize,
          conditions
        }).then(data => {
          console.log(data);
          data.data.data.map((item, index) => {
            item.form = item.form === "ONLINE" ? "网课" : "线下";
          })
          this.tableData = data.data.data;
          this.loading = false;
        }).catch(error => {
          this.loading = false;
        })
      },

      rowClick(row, column, event) {
        console.log(row, column, event);
        this.$http.post(this.url + "/course/load", {
          id: row.id
        }).then(date => {
          this.form.date = date.data.course.date;
          this.form.endDate = date.data.course.endDate;
          this.form.courseAmount = date.data.course.courseAmount;
          this.form.form = date.data.course.form;
          this.form.teacher = date.data.course.teacher;
          this.form.courseName = date.data.course.courseName;
          this.form.place = date.data.course.place;
          this.form.saveDisplay = 'none';
          this.form.newDisplay = '';
          this.form.disabled = true;
          this.dialogFormVisible = true;
          this.form.credit = date.data.course.credit;
        })
      },

      select(string) {
        this.list({"course_name": string});
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
      }
    },
    //生命周期 - 创建之前",数据模型未加载,方法未加载,html模板未加载
    beforeCreate() {
    },

    //生命周期 - 创建完成（可以访问当前this实例）",数据模型已加载，方法已加载,html模板已加载,html模板未渲染
    created() {
      this.list();
    },
    //生命周期 - 挂载之前",html模板未渲染
    beforeMount() {

    },

    //生命周期 - 挂载完成（可以访问DOM元素）",html模板已渲染
    mounted() {

    },

    //生命周期 - 更新之前",数据模型已更新,html模板未更新
    beforeUpdate() {

    },
    //生命周期 - 更新之后",数据模型已更新,html模板已更新
    updated() {

    },
    //生命周期 - 销毁之前",
    beforeDestroy() {

    },
    destroyed() {

    },
    //生命周期 - 销毁完成",
    //如果页面有keep-alive缓存功能，这个函数会触发",
    activated() {

    },
  }
</script>

<style>

</style>
