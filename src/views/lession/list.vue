<template>
  <div class="app-container">
    <el-form ref="queryForm" :model="queryParam" :inline="true">
      <el-form-item label="课程名：">
        <el-input v-model="queryParam.userName" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm">查询</el-button>
        <router-link :to="{path:'/edu/lession/edit'}" class="link-left">
          <el-button type="primary">添加</el-button>
        </router-link>
      </el-form-item>
    </el-form>

    <el-table v-loading="listLoading" :data="tableData" border fit highlight-current-row style="width: 100%">
      <el-table-column prop="id" label="Id" />
      <el-table-column prop="name" label="课程名字" />
      <el-table-column prop="lm" label="时间" />
      <el-table-column prop="startTime" label="开始时间" />
      <el-table-column prop="endTime" label="结束时间" />
      <!--
        <el-table-column prop="members"
          label="学员名" />
      -->

      <el-table-column prop="group" label="班级" :formatter="levelFormatter" />
      <el-table-column width="270px" label="操作" align="center">
        <template slot-scope="{row}">

          <router-link :to="{path:'/edu/lession/edit', query:{id:row.id}}" class="link-left">
            <el-button size="mini">编辑</el-button>
          </router-link>

          <el-button size="mini" type="danger" class="link-left" @click="deleteUser(row)">删除</el-button>
        </template>
      </el-table-column>
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
    search() {
      this.listLoading = true
      if (this.$avinit.value === false) {
        var APP_ID = 'eAQGWOHouG1eTjsMkbAdlUD8-gzGzoHsz'
        var APP_KEY = 'R7yHLgfevCPbj8axml1CN49N'
        AV.init({
          appId: APP_ID,
          appKey: APP_KEY,
          serverURLs: this.$avhost.value
        })
        this.$avinit.value = true
      }
      var p = this
      var query = new AV.Query('Lesson')
      query.find().then(function(students) {
        // students 是包含满足条件的 Student 对象的数组
        p.queryParam.pageIndex = 0
        p.listLoading = false
        p.tableData = students.map((item) => {
          var moment1 = moment(item.get('lm'))
          return {
            name: item.get('name'),
            lm: moment1.format('YYYY-MM-DD'),
            startTime: item.get('startTime'),
            endTime: item.get('endTime'),
            id: item.id,
            group: item.get('group')
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
      const todo = AV.Object.createWithoutData('Lesson', row.id)
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
