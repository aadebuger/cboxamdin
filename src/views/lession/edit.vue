<template>
  <div class="app-container">
    <h1 class="text-center">新增课程</h1>
    <el-form ref="form" v-loading="formLoading" :model="form" label-width="100px" :rules="rules">
      <el-row type="flex" class="row-bg" justify="space-between">
        <el-col :span="18" :offset="2" style="padding-top: 20px;">
          <el-form-item label="课程名" prop="name" required>
            <el-input v-model="form.name" placeholder="请输入课程名称" />
          </el-form-item>
          <el-row type="flex" class="row-bg" justify="space-between">
            <el-form-item label="上课日期" prop="lm">
              <el-date-picker v-model="form.date" type="date" placeholder="选择日期" />
            </el-form-item>
            <el-form-item label="时间">
              <el-time-picker
                v-model="form.time"
                is-range
                range-separator="至"
                start-placeholder="开始时间"
                end-placeholder="结束时间"
                placeholder="选择时间范围"
              />
            </el-form-item>
          </el-row>
          <el-form-item label="班级：" prop="userLevel" required>
            <el-select v-model="form.userLevel" placeholder="班级">
              <el-option v-for="item in levelEnum" :key="item.key" :value="item.key" :label="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-row type="flex" class="row-bg" justify="end">
              <el-button @click="resetForm">重置</el-button>
              <el-button type="primary" @click="submitForm">提交</el-button>
            </el-row>
          </el-form-item>
        </el-col>
      </el-row>
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

var editstudent
export default {
  data() {
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
        deviceid: '',
        date: new Date(),
        time: [new Date(2000, 1, 1, 9, 0, 0), new Date(2000, 1, 1, 11, 59, 59)]
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
  },
  created() {
    const id = this.$route.query.id
    const _this = this
    if (id && parseInt(id) !== 0) {
      _this.formLoading = true

      const query = new AV.Query('Lesson')
      var p = this
      query.get(id).then((todo) => {
        // todo 就是 objectId 为 582570f38ac247004f39c24b 的 Todo 实例
        console.log('Student', todo)
        console.log('department', todo.get('department'))
        p.form.name = todo.get('name')
        p.form.lm = todo.get('lm')
        p.form.startTime = todo.get('startTime')
        p.form.endTime = todo.get('endTime')
        p.form.userLevel = todo.get('userlevel')
        _this.formLoading = false
        editstudent = todo
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
    submitForm() {
      const _this = this
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log('this.$route.query.id', this.$route.query.id)
          const id = this.$route.query.id
          const Todo = AV.Object.extend('Lesson')

          var todo = new Todo()

          if (id && parseInt(id) !== 0) {
            todo = editstudent
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
              _this.$router.push('/lession/list')
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
    submitForm1() {
      const _this = this
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
    resetForm() {
      const lastId = this.form.id
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
  }
}
</script>
