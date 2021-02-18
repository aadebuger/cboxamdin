  <template>
    <el-table
      :data="tableData" border fit highlight-current-row
      style="width: 100%">
      <el-table-column
        prop="name"
        label="名字"
        width="180">
      </el-table-column>
  <el-table-column label="头像" width="100">>
　　<template slot-scope="scope">
　　　<img :src="scope.row.imageurl" width="40" height="40" class="head_pic"/>
　　</template>
  </el-table-column>
      <el-table-column
        prop="androidid"
        label="手机标示"
        width="180">
      </el-table-column>
      <el-table-column
        prop="department"
        label="部门" width="60">
      </el-table-column>
      <el-table-column
      fixed="right"
      label="操作"
      width="150">
      <template slot-scope="scope">
        <el-button @click="handleAsyncClick(scope.row)" type="text" size="small">更新箱子配置</el-button>
        <el-button  @click="handlerunClick(scope.row)" type="text" size="small">修改</el-button>
         <el-button type="text" size="small"></el-button>
      </template>
    </el-table-column>
    </el-table>
  </template>

<script>
import AV from 'leancloud-storage'
async function querySync(dataid)
    {
      for ( var i = 0; i < 10; i++) {
    console.log(i);
    await new Promise(resolve => setTimeout(resolve, 500));
     var count =i ;
    const query = new AV.Query('Student');
      query.get(dataid).then((todo) => {
        console.log("todo",todo)
        console.log("i=",count)
      
      })

      }
        return "同步成功"
    }
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
    var query = new AV.Query('Student')
    query.find().then(function (students) {
      // students 是包含满足条件的 Student 对象的数组
      console.log(`students=`, students)
      p.tableData = students.map((item) => {
        console.log(`time=`, Date.parse(item.createdAt))
        var date1 = Date.parse(item.createdAt)
        console.log('date1=', date1)
        console.log('id=', item.id)
        return {
          name: item.get('name'),
          androidid: item.get('androidid'),
          photo: item.get('photo'),
          department: item.get('department'),
          imageurl: item.get('imageurl'),
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
    async querySync1(dataid)
    {
      for (i = 0; i < 10; i++) {
    console.log(i);
    await new Promise(resolve => setTimeout(resolve, 10000));

    const query = new AV.Query('Student');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        console.log("i=",i)
      
      })

      }
      return "同步成功"
    },
    async handleAsyncClick(row) {
       await new Promise(resolve => setTimeout(resolve, 10000));
       console.log('id=', row.id)
        var p = this
      const query = new AV.Query('Student');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        todo.set("syncing",1)
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

      });
      var msg = await querySync(row.id)
      console.log("msg=",msg)
       p.$message({
          message: msg,
          type: 'success'
        })
        
    },
    handleClick: function (row) {
       console.log('id=', row.id)
        var p = this
      const query = new AV.Query('Student');
      query.get(row.id).then((todo) => {
        console.log("todo",todo)
        todo.set("syncing",1)
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

    },
    handlerunClick: function (row) {
      console.log('row=', row)
      console.log('row=', row.name)
      console.log('id=', row.id)
      console.log(this.tableData)
      // this.$router.push({name: `run`, params: {name: row.name}})
      // window.open(`http://localhost:9081/view/pytests/job/pytestlr/`, '_blank')
      this.$router.push({name: `EditStudent`, params: {id: row.id}})
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
