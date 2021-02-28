<template>
  <div class="app-container">

    <el-table v-loading="listLoading" :data="tableData" border fit highlight-current-row style="width: 100%">
      <el-table-column         prop="boxNumber" label="箱号" />
      <el-table-column         prop="seriial" label="设备号" />
      <el-table-column  prop="name"
        label='绑定人' />
         <el-table-column  prop="status" :formatter="setStatus"
        label='状态' />     
    <!--
      <el-table-column prop="members"
        label="学员名" />
    -->
      <el-table-column width="270px" label="操作" align="center">
        <template slot-scope="{row}">

          <el-button  size="mini" type="danger" @click="handlerunClick(row)" class="link-left">开箱</el-button>
        </template>
      </el-table-column>
    </el-table>
<!--
    <pagination v-show="total>0" :total="total" :page.sync="queryParam.pageIndex" :limit.sync="queryParam.pageSize"
                @pagination="search"/>
                -->
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
  data () {
    return {
      queryParam: {
        userName: '',
        role: 1,
        pageIndex: 1,
        pageSize: 80
      },
      listLoading: true,
      tableData: [],
      total: 0
    }
  },
  created () {
    this.search()
  },
  methods: {
      handlerunClick1: function (row) {
  console.log('id=', row.id)
        var p = this
      const query = new AV.Query('Facebox');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        todo.set("task",24)
        todo.set("boxNumber",row.boxNumber)
        todo.save().then((todo) => {
        // 成功保存之后，执行其他逻辑
      
        p.$message({
          message: '打开请求成功',
          type: 'success'
        })
      }, (error) => {
        // 异常处理
        console.log('save error', error)
      })

      });

    },
     handlerunClick1: function (row) {
  console.log('id=', row.id)
        var p = this
      const query = new AV.Query('Box');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        todo.set("action","open")
        todo.save().then((todo) => {
        // 成功保存之后，执行其他逻辑
      
        p.$message({
          message: '打开请求成功',
          type: 'success'
        })
      }, (error) => {
        // 异常处理
        console.log('save error', error)
      })

      });

    },
    setStatus(row, column) {
      return this.getSatus(row.status)
    },
    getSatus(status){
     switch (status) {
        case 0:
          return '空闲';
        case 1:
          return "占用";
        case 2:
          return "锁定";
        default:
          return "已撤销";
      }
    },
    search () {
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
    var query = new AV.Query('Box')
    query.find().then(function (students) {
      // students 是包含满足条件的 Student 对象的数组
      console.log(`students=`, students)
      
      p.queryParam.pageIndex = 0
      p.listLoading = false
      p.tableData = students.map((item) => {
        console.log(`time=`, Date.parse(item.createdAt))
        var date1 = Date.parse(item.createdAt)
        console.log('date1=', date1)
        console.log("item.get('lm'=", item.get('lm'))
        var moment1 = moment(item.get('lm'))
        console.log('moment1', moment1)
        console.log('moment1 format', moment1.format('YYYY-MM-DD'))
        return {
          boxNumber: item.get('boxNumber'),
          id: item.id,
          serial: item.get('serial'),
          model: item.get('model'),
          device: item.get('device'),
          status: item.get('state')
        }
      })
      p.total =  p.tableData.length
      
    })



    },
    search1 () {
      this.listLoading = true
      userApi.getUserPageList(this.queryParam).then(data => {
        const re = data.response
        this.tableData = re.list
        this.total = re.total
        this.queryParam.pageIndex = re.pageNum
        this.listLoading = false
      })
    },
    changeStatus (row) {
      let _this = this
      userApi.changeStatus(row.id).then(re => {
        if (re.code === 1) {
          row.status = re.response
          _this.$message.success(re.message)
        } else {
          _this.$message.error(re.message)
        }
      })
    },
    deleteUser (row) {
      let _this = this
      console.log("deleteUser row id ",row.id)
      const todo = AV.Object.createWithoutData('Lesson', row.id);
      todo.destroy().then( (student)=>
      {
        _this.search()
           _this.$message.success(re.message)

      },(error) =>
      {
      _this.$message.error(re.message)
      })

    },
    deleteUser1 (row) {
      let _this = this
      userApi.deleteUser(row.id).then(re => {
        if (re.code === 1) {
          _this.search()
          _this.$message.success(re.message)
        } else {
          _this.$message.error(re.message)
        }
      })
    },
    submitForm () {
      this.queryParam.pageIndex = 1
      this.search()
    },
    levelFormatter  (row, column, cellValue, index) {
      return this.enumFormat(this.levelEnum, cellValue)
    },
    sexFormatter  (row, column, cellValue, index) {
      return this.enumFormat(this.sexEnum, cellValue)
    },
    statusFormatter (status) {
      return this.enumFormat(this.statusEnum, status)
    },
    statusTagFormatter (status) {
      return this.enumFormat(this.statusTag, status)
    },
    statusBtnFormatter (status) {
      return this.enumFormat(this.statusBtn, status)
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
  }
}
</script>
