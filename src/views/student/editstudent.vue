<template>
<div>
<el-form :model="studentForm" :rules="rules"  ref="studentForm" >

  
   
      <el-form-item label="人名" prop="name">
         <el-col :span="8">
  <el-input
  placeholder="人名"
  v-model="studentForm.name">
</el-input>
</el-col>
  </el-form-item>



<el-row>
  
       <el-form-item label="设备" prop="deviceid">
         <el-col span="8">
  <el-input

  placeholder="设备id"
  v-model="studentForm.deviceid">
</el-input>

</el-col>
  </el-form-item>
  <el-form-item label="品牌">
<el-col span="8"> 
  <el-input
  placeholder=""
  v-model="studentForm.model">
</el-input>
  </el-col>
</el-form-item>
</el-row>
 <el-form-item label="照片" prop="imageUrl">
<el-upload
  class="avatar-uploader"
  action="http://localhost:7000/1.1/filesformminio/test.png"
  :show-file-list="false"
  :on-success="handleAvatarSuccess"
  :before-upload="beforeAvatarUpload">
  <img v-if="studentForm.imageUrl" :src="studentForm.imageUrl" class="avatar">
  <i v-else class="el-icon-plus avatar-uploader-icon"></i>
</el-upload>
  </el-form-item>
  <el-form-item label="部门">
 <el-select v-model="studentForm.depart" placeholder="部门">
    <el-option
      v-for="item in options"
      :key="item.value"
      :label="item.label"
      :value="item.value">
    </el-option>
  </el-select>
  </el-form-item>
  <el-form-item>
    <el-button type="info"  @click="onSearch">获取手机型号</el-button>  <el-button type="primary" @click="onSubmit">提交</el-button>
  </el-form-item>
</el-form>
</div>
</template>
<style>
  .el-select .el-input {
    width: 130px;
  }
  .input-with-select .el-input-group__prepend {
    background-color: #fff;
  }
  .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
<script>
import moment from 'moment'
import AV from 'leancloud-storage'
export default {
  data () {
      var id = this.$route.params.id
      console.log("id=",id)
    const query = new AV.Query('Student')
    var p = this
    query.get(id).then((todo) => {
      // todo 就是 objectId 为 582570f38ac247004f39c24b 的 Todo 实例
      console.log('Student', todo)
      console.log('department', todo.get('department'))
      p.studentForm.name = todo.get('name')
      p.studentForm.deviceid = todo.get('androidid')
      p.studentForm.imageUrl = todo.get('imageurl')
      p.studentForm.depart = todo.get('department')
    })
    return {
      options: [{
        value: `主管`,
        label: '主管'
      }, {
        value: `学员`,
        label: '学员'
      }
      ],
      studentForm: {
      depart: `学员`,
      user: '',
      input: '',
      textarea: '',
      name: '',
      deviceid: '',
      photo: '',
      imageUrl: '',
      model: '',
      lm: '',
      members: []
      },
      rules: { name: [
            { required: true, message: '请输入名字', trigger: 'blur' },
            { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
          ],
          deviceid: [
            { required: true, message: '请获取手机类型', trigger: 'blur' }
          ],
          imageUrl: [
            { type: 'url', required: true,   message: '请上传照片'}
          ]
      }
    }
  },
  methods: {
    onSearch () {
      console.log('onSearch')
    var p = this
    var query = new AV.Query('Androiddevicenew')
    query.first().then(function (device) {
      // students 是包含满足条件的 Student 对象的数组
     if (typeof device === 'undefined') 
     {
        p.$message({
          message: '未接手机,请连接手机后再试',
          type: 'fail'
        })
       return 
     }

      console.log(`students=`, device)
      console.log("name",device.get("name"))
      p.studentForm.deviceid = device.get('serial')
      p.studentForm.model = device.get("name")
      
      }).catch((error) => {
      console.error(error);
});
    },
    onSubmit () {
      console.log('this.$refs[formName]',this.$refs['studentForm'])

      this.$refs['studentForm'].validate((valid) => {
          if (valid) {
         
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

      const Todo = AV.Object.extend('Student')

      // 构建对象
      const todo = new Todo()

      // 为属性赋值
      todo.set('name', this.studentForm.name)
      todo.set('androidid', this.studentForm.deviceid)
      todo.set('photo', this.studentForm.photo)
      todo.set('imageurl',this.studentForm.imageUrl)
      todo.set('department', this.studentForm.depart)
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

          } else {
            console.log('error submit!!');
            return false;
          }
        });

      
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
    },
    handleAvatarSuccess(res, file) {
        // this.imageUrl = URL.createObjectURL(file.raw);
        console.log('res=', res)
        console.log('file=', file)
        // this.imageUrl='http://localhost:9000/boxhr/2060S.png'
        //this.imageUrl = URL.createObjectURL(file.raw);
        this.studentForm.imageUrl= res['url']
      },
    beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/png';
        const isLt2M = file.size / 1024 / 1024 < 2;

        if (!isJPG) {
          this.$message.error('上传头像图片只能是 PNG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      }
  },
  mounted () {
    console.log(`Vue.prototype.$avinit`, this.$avinit.value)
    var rlserver= this.$avhost.value
    if (this.$avinit.value === false) {
      var APP_ID = 'eAQGWOHouG1eTjsMkbAdlUD8-gzGzoHsz'
      var APP_KEY = 'R7yHLgfevCPbj8axml1CN49N'
      AV.init({
        appId: APP_ID,
        appKey: APP_KEY,
        serverURLs: rlserver
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
