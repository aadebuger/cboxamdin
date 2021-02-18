<template>
<div>
<el-form  :model="lessonForm" :rules="rules" ref="lessonForm" >
  <el-form-item label="课程名称" prop="name">
  <el-input
  type="textarea"
  :rows="2"
  placeholder="名称"
  v-model="lessonForm.name">
</el-input>
  </el-form-item>
 <el-form-item label="课程日期">
<el-date-picker
      v-model="lessonForm.lm"
      type="date"
      placeholder="选择日期">
    </el-date-picker>
  </el-form-item>
  <el-form-item label="时间">
<el-time-select
    placeholder="起始时间"
    v-model="lessonForm.startTime"
    :picker-options="{
      start: '08:30',
      step: '00:15',
      end: '18:30'
    }">
  </el-time-select>
  <el-time-select
    placeholder="结束时间"
    v-model="lessonForm.endTime"
    :picker-options="{
      start: '08:30',
      step: '00:15',
      end: '18:30',
      minTime: lessonForm.startTime
    }">
  </el-time-select>
  </el-form-item>
  <el-form-item label="学员">
  <el-transfer v-model="lessonForm.members" :data="memberdata"
  :titles="['学员', '学生']"
  ></el-transfer>
  </el-form-item>
  <el-form-item>
    <el-button type="primary" @click="onSubmit">提交</el-button>
  </el-form-item>
</el-form>
</div>
</template>
<script>
import moment from 'moment'
import AV from 'leancloud-storage'
export default {
  data () {
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
      p.memberdata = students.map((item) => {
        console.log(`time=`, Date.parse(item.createdAt))
        var date1 = Date.parse(item.createdAt)
        console.log('date1=', date1)
        return {
          label: item.get('name'),
          key: item.get('name'),
          disabled: false
        }
      })
    })
    const generateData = _ => {
      const data = []
      for (let i = 1; i <= 0; i++) {
        data.push({
          key: i,
          label: `学员 ${i}`,
          disabled: i % 4 === 0
        })
      }
      return data
    }
    return {
      memberdata: generateData(),
      lessonForm: {
      startTime: '09:00',
      endTime: '11:00',
      user: '',
      input: '',
      textarea: '',
      name: '',
      lm: '',
      members: []
      },
      rules: { name: [
            { required: true, message: '请输入名字', trigger: 'blur' },
            { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
          ]
      }
    }
  },
  methods: {
    onSubmit () {
      var p = this
      console.log(this.textarea)
      console.log('submit!')
      console.log('lm=', this.lm)
      console.log(this.members)

      var moment1 = moment(this.lm)
      console.log(this.startTime)
      console.log('moment1', moment1)
      console.log('moment1 format', moment1.format('YYYY-MM-DD'))
      var newdatestr = moment1.format('YYYY MM DD') + ' ' + this.startTime
      var moment2 = moment(newdatestr)
      console.log('moment2', moment2)
      console.log('moment2 date', moment2.toDate())
      //      const memberv = (this.members).split(',')

      const Todo = AV.Object.extend('Lesson')

      // 构建对象
      const todo = new Todo()

      // 为属性赋值
      todo.set('name', this.name)
      todo.set('members', this.members)
      todo.set('lm', this.lm)
      todo.set('startTime', this.startTime)
      todo.set('endTime', this.endTime)

      // 将对象保存到云端
      todo.save().then((todo) => {
        // 成功保存之后，执行其他逻辑
        console.log(`保存成功。objectId：${todo.id}`)
        p.$message({
          message: '成功',
          type: 'success'
        })
      }, (error) => {
        // 异常处理
        console.log('save error', error)
      })
    },
    handleRemove (file, fileList) {
      console.log(file, fileList)
    },
    handlePreview (file) {
      console.log(file)
    },
    handleExceed (files, fileList) {
      this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    beforeRemove (file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
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

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
