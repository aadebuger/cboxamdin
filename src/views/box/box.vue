  <template>
    <el-table
      :data="tableData" border fit highlight-current-row
      style="width: 100%">
      <el-table-column
        prop="boxNumber"
        label="名字"
        width="180">
      </el-table-column>
 <el-table-column
        prop="serial"
        label="设备号"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="名字" width="200">
      </el-table-column>
      <el-table-column
        prop="status"
        label="状态"
        width="180">
      </el-table-column>
      <el-table-column
        prop="device"
        label="device" width="200">
      </el-table-column>
      <el-table-column
      fixed="right"
      label="操作"
      width="150">
      <template slot-scope="scope">
         <el-button  @click="handlerunClick(scope.row)" type="text" size="small">开箱</el-button>
      </template>
    </el-table-column>
    </el-table>
  </template>

<script>
import AV from 'leancloud-storage'
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
    var query = new AV.Query('Box')
    query.ascending('boxNumber')
    query.find().then(function (students) {
      // students 是包含满足条件的 Student 对象的数组
      console.log(`students=`, students)
      p.tableData = students.map((item) => {
        console.log(`time=`, Date.parse(item.createdAt))
        var date1 = Date.parse(item.createdAt)
        console.log('date1=', date1)
        console.log('id=', item.id)
        return {
          boxNumber: item.get('boxNumber'),
          id: item.id,
          serial: item.get('serial'),
          model: item.get('model'),
          device: item.get('device')
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
  console.log('id=', row.id)
        var p = this
      const query = new AV.Query('Box');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        todo.set("action","open")
        todo.save().then((todo) => {
        // 成功保存之后，执行其他逻辑
      
        p.$message({
          message: '同步请求成功',
          type: 'success'
        })
      }, (error) => {
        // 异常处理
        console.log('save error', error)
      })

      });

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
        serverURLs:  this.$avhost.value
      })
      this.$avinit.value = true
      console.log(`Vue.prototype.$avinit`, this.$avinit)
    }
  }
}
</script>
