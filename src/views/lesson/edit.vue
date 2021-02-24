<template>
  <div class="app-container">

    <el-form :model="form" ref="form" label-width="100px" v-loading="formLoading" :rules="rules">
      <el-form-item label="课程名："  prop="name" required>
        <el-input v-model="form.name"></el-input>
      </el-form-item>
  
   <el-form-item label="课程日期" prop="lm">
<el-date-picker
      v-model="form.lm"
      type="date"
      placeholder="选择日期">
    </el-date-picker>
  </el-form-item>
  <el-form-item label="时间">
<el-time-select
    placeholder="起始时间"
    v-model="form.startTime"
    :picker-options="{
      start: '08:30',
      step: '00:15',
      end: '18:30'
    }">
  </el-time-select>
  <el-time-select
    placeholder="结束时间"
    v-model="form.endTime"
    :picker-options="{
      start: '08:30',
      step: '00:15',
      end: '18:30',
      minTime: form.startTime
    }">
  </el-time-select>
</el-form-item>

   
      <el-form-item label="年级：" prop="userLevel" required>
        <el-select v-model="form.userLevel" placeholder="年级">
          <el-option v-for="item in levelEnum" :key="item.key" :value="item.key" :label="item.value"></el-option>
        </el-select>
      </el-form-item>
      <!--
      <el-form-item label="状态：" required>
        <el-select v-model="form.status" placeholder="状态">
          <el-option v-for="item in statusEnum" :key="item.key" :value="item.key" :label="item.value"></el-option>
        </el-select>
      </el-form-item>
      -->
      <el-form-item>
        <el-button type="primary" @click="submitForm">提交</el-button>
        <el-button @click="resetForm">重置</el-button>
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
import { mapGetters, mapState, mapActions } from 'vuex'
import userApi from '@/api/user'
import AV from 'leancloud-storage'
var editstudent;
export default {
  data () {
    return {
      form: {
        id: null,
        name: '',
        password: '',
        realName: '',
        role: 1,
        status: 1,
        age: '',
        sex: '',
        birthDay: null,
        phone: null,
        userLevel: 1,
        imageUrl: '',
        startTime: '09:00',
      endTime: '11:00',
        deviceid: ''
      },
      formLoading: false,
      rules: {
        userName: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        realName: [
          { required: true, message: '请输入真实姓名', trigger: 'blur' }
        ],
        userLevel: [
          { required: true, message: '请选择年级', trigger: 'change' }
        ]
      }
    }
  },
  created () {
    let id = this.$route.query.id
    let _this = this
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true

      const query = new AV.Query('Lesson')
    var p = this
    query.get(id).then((todo) => {
      // todo 就是 objectId 为 582570f38ac247004f39c24b 的 Todo 实例
      console.log('Student', todo)
      console.log('department', todo.get('department'))
      p.form.name = todo.get('name')
      p.form.lm =todo.get('lm')
      p.form.startTime=todo.get('startTime')
      p.form.endTime=todo.get('endTime')
      p.form.userLevel = todo.get("userlevel")
       _this.formLoading = false
      editstudent=todo

    })
    /*
      userApi.selectUser(id).then(re => {
        _this.form = re.response
        _this.formLoading = false
      })
      */
    }
  },
  methods: {
    submitForm ()
    {
      let _this = this
      this.$refs.form.validate((valid) => {
        if (valid) {
            console.log("this.$route.query.id",this.$route.query.id)
            let id = this.$route.query.id
            const Todo = AV.Object.extend('Lesson')
    
            var todo = new Todo()

             if (id && parseInt(id) !== 0)
                {
                    todo= editstudent
                }
          // 为属性赋值
      todo.set('name', this.form.name)
      todo.set('userlevel', this.form.userLevel)
      todo.set('lm', this.form.lm)
      todo.set('startTime', this.form.startTime)
      todo.set('endTime', this.form.endTime)

          todo.save().then((todo) => {
            // 成功保存之后，执行其他逻辑
            console.log(`保存成功。objectId：${todo.id}`)
            _this.$message({
              message: '成功',
              type: 'success'
            })
              _this.delCurrentView(_this).then(() => {
              _this.$router.push('/newlesson/newlesson/list')
              })

          }, (error) => {
            // 异常处理
            _this.formLoading = false
            console.log('save error', error)
          })
        } else {
          console.log('error submit!!')
          _this.formLoading = false
          return false
        }
      })
    },
    submitForm1 () {
      let _this = this
      this.$refs.form.validate((valid) => {
        if (valid) {
          this.formLoading = true
          userApi.createUser(this.form).then(data => {
            if (data.code === 1) {
              _this.$message.success(data.message)
              _this.delCurrentView(_this).then(() => {
                _this.$router.push('/user/student/list')
              })
            } else {
              _this.$message.error(data.message)
              _this.formLoading = false
            }
          }).catch(e => {
            _this.formLoading = false
          })
        } else {
          return false
        }
      })
    },
    resetForm () {
      let lastId = this.form.id
      this.$refs['form'].resetFields()
      this.form = {
        id: null,
        name: '',
        password: '',
        realName: '',
        role: 1,
        status: 1,
        age: '',
        sex: '',
        birthDay: null,
        phone: null,
        userLevel: null,
        startTime: '09:00',
      endTime: '11:00',
      lm: ''
      }
      this.form.id = lastId
    },
    handleBeforeupload() {
      return this.$uploadhost.value
      //  return "http://localhost:7000/1.1/filesformminio/test.png"
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log(file)
    },
    handleExceed(files, fileList) {
      this.$message.warning(`当前限制选择 3 个文件，本次选择了 ${files.length} 个文件，共选择了 ${files.length + fileList.length} 个文件`)
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },
    handleAvatarSuccess(res, file) {
      // this.imageUrl = URL.createObjectURL(file.raw);
      console.log('res=', res)
      console.log('file=', file)
      // this.imageUrl='http://localhost:9000/boxhr/2060S.png'
      // this.imageUrl = URL.createObjectURL(file.raw);
      this.form.imageUrl = res['url']
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 PNG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    ...mapActions('tagsView', { delCurrentView: 'delCurrentView' })
  },
  computed: {
    ...mapGetters('enumItem', [
      'enumFormat'
    ]),
    ...mapState('enumItem', {
      sexEnum: state => state.user.sexEnum,
      roleEnum: state => state.user.roleEnum,
      statusEnum: state => state.user.statusEnum,
      levelEnum: state => state.user.levelEnum
    })
  }
}
</script>
