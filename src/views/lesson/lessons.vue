  <template>
    <el-table
      :data="tableData" border fit highlight-current-row
      style="width: 100%">
      <el-table-column
        prop="name"
        label="课程名字"
        width="180">
      </el-table-column>
      <el-table-column
        prop="lm"
        label="时间"
        width="180">
      </el-table-column>
      <el-table-column
        prop="startTime"
        label='开始时间' width="200">
      </el-table-column>
      <el-table-column
        prop="endTime"
        label='结束时间' width="200">
      </el-table-column>
      <el-table-column
        prop="members"
        label="学员名" width="200">
      </el-table-column>
      <el-table-column
      fixed="right"
      label="操作"
      width="150">
      <template slot-scope="scope">
        
        <el-button  @click="handlerunClick(scope.row)" type="text" size="small" icon="el-icon-edit" >修改</el-button>
      </template>
    </el-table-column>
    </el-table>
  </template>

<script>
import AV from 'leancloud-storage'
import moment from 'moment'
export default {
  data () {
    console.log('data')
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
    var query = new AV.Query('Lesson')
    query.find().then(function (students) {
      // students 是包含满足条件的 Student 对象的数组
      console.log(`students=`, students)
      p.tableData = students.map((item) => {
        console.log(`time=`, Date.parse(item.createdAt))
        var date1 = Date.parse(item.createdAt)
        console.log('date1=', date1)
        console.log("item.get('lm'=", item.get('lm'))
        var moment1 = moment(item.get('lm'))
        console.log('moment1', moment1)
        console.log('moment1 format', moment1.format('YYYY-MM-DD'))
        return {
          name: item.get('name'),
          lm: moment1.format('YYYY-MM-DD'),
          startTime: item.get('startTime'),
          endTime: item.get('endTime'),
          address: 'home_tests',
          id: item.id
        }
      })

      return {
        tableData: [{
          name: 'test',
          android: 'HHH',
          address: 'home_tests'
        }]
      }
    })
    return {
      tableData: [{
        name: 'default',
        android: 'default',
        address: 'home_tests'
      }]
    }
  },
  methods: {
    handleClick: function (row) {
      var query = new AV.Query('pytest')
      query.equalTo('lastName', 'Smith')
      query.find().then(function (students) {
      // students 是包含满足条件的 Student 对象的数组
      })
      console.log('row=', row.name)

      //      this.$router.push({ path: `/unittest/` + row.name })
      this.$router.push({name: `unittest`, params: {name: row.name}})
    },
    handlerunClick: function (row) {
      console.log('row id=', row.id)
       this.$router.push({name: `EditLesson`, params: {id: row.id}})
       }

  },
  mounted () {
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
  }
}
</script>
