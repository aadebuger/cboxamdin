<template>
  <div class="app-container">
    <el-form ref="queryForm" :model="queryParam" :inline="true">
      <el-form-item label="用户名：">
        <el-input v-model="queryParam.userName" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm">查询</el-button>
        <router-link :to="{path:'/user/student/edit'}" class="link-left">
          <el-button type="primary">导出</el-button>
        </router-link>
      </el-form-item>
    </el-form>

    <el-table v-loading="listLoading" :data="tableData" border fit highlight-current-row style="width: 100%">
      <el-table-column prop="id" label="Id" />
      <el-table-column prop="userName" label="用户名" />
      <el-table-column prop="lessonname" label="课程" />
      <el-table-column prop="status" label="报警内容" />

      <el-table-column prop="createTime" label="报警时间" width="160px" />

      <!--      <el-table-column width="270px" label="操作" align="center">
        <template slot-scope="{row}" />
      </el-table-column> -->
    </el-table>
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParam.pageIndex"
      :limit.sync="queryParam.pageSize"
      @pagination="search"
    />
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex'
import Pagination from '@/components/Pagination'
import userApi from '@/api/user'
import AV from 'leancloud-storage'
import moment from 'moment'
async function querySync(dataid) {
  var bQueryok = 0
  var msg = '同步成功'
  for (var i = 0; i < 10; i++) {
    console.log(i)
    await new Promise(resolve => setTimeout(resolve, 5000))
    var count = i
    const query = new AV.Query('Student')
    query.get(dataid).then((todo) => {
      console.log('todo', todo)
      console.log('i=', count)
      var syncing = todo.get('syncing')
      console.log('synciing=', syncing)
      if (syncing === 0) {
        bQueryok = 1
        msg = todo.get('msg')
        return todo.get('msg')
      }
    })
    if (bQueryok === 1) { break }
  }
  return msg
}

export default {
  components: { Pagination },
  data() {
    return {
      queryParam: {
        userName: '',
        role: 1,
        pageIndex: 1,
        pageSize: 10
      },
      listLoading: true,
      tableData: [],
      total: 0
    }
  },
  computed: {
    ...mapGetters('enumItem', [
      'enumFormat'
    ]),
    ...mapState('enumItem', {
      sexEnum: state => state.user.sexEnum,
      statusEnum: state => state.user.statusEnum,
      statusTag: state => state.user.statusTag,
      statusBtn: state => state.user.statusBtn,
      levelEnum: state => state.user.levelEnum
    })
  },
  created() {
    this.search()
  },
  methods: {

    async handleAsyncClick(row) {
      console.log('id=', row.id)
      var p = this
      const query = new AV.Query('Student')
      query.get(row.id).then((todo) => {
        console.log('todo', todo)
        todo.set('syncing', 1)
        todo.save().then((todo) => {
        // 成功保存之后，执行其他逻辑

          p.$message({
            message: '发送同步请求',
            type: 'success'
          })
        }, (error) => {
        // 异常处理
          console.log('save error', error)
        })
      })
      var msg = await querySync(row.id)
      console.log('msg=', msg)
      p.$message({
        message: msg,
        type: 'success'
      })
    },
    search() {
      this.listLoading = true
      console.log(`Vue.prototype.$avinit`, this.$avinit.value)
      if (this.$avinit.value === false) {
        var APP_ID = 'eAQGWOHouG1eTjsMkbAdlUD8-gzGzoHsz'
        var APP_KEY = 'R7yHLgfevCPbj8axml1CN49N'
        AV.init({
          appId: APP_ID,
          appKey: APP_KEY,
          serverURLs: this.$avhost.value
        })
        this.$avinit.value = true
        console.log(`Vue.prototype.$avinit`, this.$avinit)
      }
      var p = this
      var query = new AV.Query('Alert')
      query.find().then(function(students) {
      // students 是包含满足条件的 Student 对象的数组
        console.log(`students=`, students)

        p.queryParam.pageIndex = 0
        p.listLoading = false
        p.tableData = students.map((item) => {
          console.log(`time=`, Date.parse(item.createdAt))
          console.log(`time=`, Date.parse(item.createdAt))
          var moment1 = moment(item.get('lm'))
          console.log('moment1', moment1)
          console.log('moment1 format', moment1.format('YYYY-MM-DD'))

          var ms1 = Date.parse(item.createdAt)
          console.log('id=', item.id)
          console.log('status=', item.get('status'))

          return {
            userName: item.get('student'),

            lessonname: item.get('lession'),
            status: item.get('status'),
            id: item.id,
            createTime: moment(ms1).format('YYYY-MM-DD')
          }
        })
        p.total = p.tableData.length
      })
    },
    search1() {
      this.listLoading = true
      userApi.getUserPageList(this.queryParam).then(data => {
        const re = data.response
        this.tableData = re.list
        this.total = re.total
        this.queryParam.pageIndex = re.pageNum
        this.listLoading = false
      })
    },
    changeStatus(row) {
      const _this = this
      userApi.changeStatus(row.id).then(re => {
        if (re.code === 1) {
          row.status = re.response
          _this.$message.success(re.message)
        } else {
          _this.$message.error(re.message)
        }
      })
    },
    deleteUser(row) {
      const _this = this
      console.log('deleteUser row id ', row.id)
      const todo = AV.Object.createWithoutData('Student', row.id)
      todo.destroy().then(re => {
        if (re.code === 1) {
          _this.search()
          _this.$message.success(re.message)
        } else {
          _this.$message.error(re.message)
        }
      })
    },

    deleteUser1(row) {
      const _this = this
      userApi.deleteUser(row.id).then(re => {
        if (re.code === 1) {
          _this.search()
          _this.$message.success(re.message)
        } else {
          _this.$message.error(re.message)
        }
      })
    },
    submitForm() {
      this.queryParam.pageIndex = 1
      this.search()
    },
    levelFormatter(row, column, cellValue, index) {
      return this.enumFormat(this.levelEnum, cellValue)
    },
    sexFormatter(row, column, cellValue, index) {
      return this.enumFormat(this.sexEnum, cellValue)
    },
    statusFormatter(status) {
      return this.enumFormat(this.statusEnum, status)
    },
    statusTagFormatter(status) {
      return this.enumFormat(this.statusTag, status)
    },
    statusBtnFormatter(status) {
      return this.enumFormat(this.statusBtn, status)
    }
  }
}
</script>
